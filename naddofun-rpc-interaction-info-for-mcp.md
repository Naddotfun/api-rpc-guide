# Nad.fun RPC Interaction Information for MCP

## Overview

This document provides information on interacting with the Nad.fun platform through blockchain transactions. It explains how to perform token trades via the bonding curve mechanism and DEX, with code examples in TypeScript.

## Table of Contents

- [Understanding Nad.fun Trading Lifecycle](#understanding-nadfun-trading-lifecycle)
- [Contract ABI](#contract-abi)
- [Contract Addresses](#contract-addresses)
- [Bonding Curve Operations](#bonding-curve-operations)
- [DEX Operations](#dex-operations)
- [Helper Functions](#helper-functions)
- [Implementation Examples](#implementation-examples)

## Understanding Nad.fun Trading Lifecycle

Tokens on Nad.fun go through distinct phases:

1. **Bonding Curve Phase**: Initially, tokens are sold through a bonding curve mechanism. During this phase:

   - Tokens can be purchased, but **cannot be transferred**
   - Price follows a mathematical bonding curve formula
   - Available token amount is calculated as `reserveToken - targetToken`

2. **DEX Phase**: After all bonding curve tokens are sold, tokens are automatically listed on DEX:
   - Purchasing the last available tokens via `exactOutBuy` triggers DEX listing
   - Once listed, tokens can be transferred like regular ERC-20 tokens
   - Trading occurs through standard DEX swap functions

## Contract ABI

To interact with Nad.fun contracts, you'll need to use the appropriate ABI (Application Binary Interface) files. These are available in the official Nad.fun contract-abi repository.

### Installing the ABI Package

Add the following to your project's `package.json`:

```json
"dependencies": {
  "contract-abi": "https://github.com/Naddotfun/contract-abi.git#v0.3.0",
  // other dependencies
}
```

Then install using your preferred package manager:

```bash
npm install
# or
yarn install
# or
pnpm install
```

### Available ABI Files

The repository includes the following ABI files:

- `ICore.json` - Core contract for bonding curve interactions
- `IToken.json` - ERC-20 token interface
- `IBondingCurve.json` - Bonding curve contract interface
- `IBondingCurveFactory.json` - Factory for creating bonding curves
- `IUniswapV2Router.json` - DEX router interface
- `IUniswapV2Factory.json` - DEX factory interface
- `IUniswapV2Pair.json` - DEX pair interface

### Using the ABI Files

Import the ABI files in your TypeScript code:

```typescript
import * as NadFunAbi from "contract-abi";

// Example usage
const coreAbi = NadFunAbi.ICore;
const tokenAbi = NadFunAbi.IToken;
const routerAbi = NadFunAbi.IUniswapV2Router;
```

### Contract Addresses

The following contract addresses are used for Monad Testnet:

```typescript
// Contract addresses on Monad Testnet
export const CONTRACT_ADDRESSES = {
  CORE:
    process.env.CORE_CONTRACT_ADDRESS ||
    "0x822EB1ADD41cf87C3F178100596cf24c9a6442f6",
  BONDING_CURVE_FACTORY:
    process.env.BONDING_CURVE_FACTORY_ADDRESS ||
    "0x60216FB3285595F4643f9f7cddAB842E799BD642",
  INTERNAL_UNISWAP_V2_ROUTER:
    process.env.INTERNAL_UNISWAP_V2_ROUTER_ADDRESS ||
    "0x619d07287e87C9c643C60882cA80d23C8ed44652",
  INTERNAL_UNISWAP_V2_FACTORY:
    process.env.INTERNAL_UNISWAP_V2_FACTORY_ADDRESS ||
    "0x13eD0D5e1567684D964469cCbA8A977CDA580827",
  WRAPPED_MON:
    process.env.WRAPPED_MON_ADDRESS ||
    "0x3bb9AFB94c82752E47706A10779EA525Cf95dc27",
};
```

## Bonding Curve Operations

### Buy Tokens from Bonding Curve

Basic purchase of tokens from the bonding curve:

```typescript
/**
 * Buys tokens from a bonding curve via Core
 * @param privateKey - Private key of the sender
 * @param tokenAddress - Token address to buy
 * @param amount - Amount of MON to spend
 * @returns Transaction hash
 */
export async function buyFromCore(
  privateKey: string,
  tokenAddress: string,
  amount: string,
): Promise<string> {
  const walletClient = createWalletClientFromPrivateKey(privateKey);

  const coreAbi = NadFunAbi.ICore;

  const amountIn = parseEther(amount);
  // 1% fee
  const fee = (amountIn * 10n) / 1000n;
  const totalValue = amountIn + fee;

  // 20 minutes deadline
  const deadline = BigInt(Math.floor(Date.now() / 1000) + 20 * 60);
  const to = walletClient.account?.address;

  const txHash = await walletClient.writeContract({
    address: CONTRACT_ADDRESSES.CORE as `0x${string}`,
    abi: coreAbi,
    functionName: "buy",
    args: [amountIn, fee, tokenAddress, to, deadline],
    value: totalValue,
    chain: walletClient.chain,
    account: walletClient.account!,
  });

  return txHash;
}
```

### Buy Exact Token Amount (exactOutBuy)

This function allows you to purchase an exact amount of tokens. This is particularly important for purchasing the remaining tokens to trigger DEX listing:

```typescript
/**
 * Buys exact output amount of tokens from bonding curve
 * The last token purchase will trigger DEX listing
 * @param privateKey - Private key of the sender
 * @param tokenAddress - Token address to buy
 * @param tokensOut - Exact amount of tokens to receive
 * @returns Transaction hash
 */
export async function exactOutBuyFromCore(
  privateKey: string,
  tokenAddress: string,
  tokensOut: bigint,
): Promise<string> {
  const walletClient = createWalletClientFromPrivateKey(privateKey);
  const address = walletClient.account?.address;

  // Get token market information
  const publicClient = createPublicRpcClient();
  const tokenMarketInfo = await fetch(
    `https://testnet-bot-api-server.nad.fun/token/market/${tokenAddress}`,
  );
  const marketData = await tokenMarketInfo.json();

  // Extract values from market data
  const virtualNative = BigInt(marketData.virtual_native);
  const virtualToken = BigInt(marketData.virtual_token);
  const reserveToken = marketData.reserve_token
    ? BigInt(marketData.reserve_token)
    : 0n;
  const targetToken = marketData.target_token
    ? BigInt(marketData.target_token)
    : 0n;

  // Calculate available tokens
  const availableTokens = reserveToken ? reserveToken - targetToken : 0n;

  // Check if requested amount exceeds available tokens
  if (tokensOut > availableTokens) {
    throw new Error(
      `Requested token amount (${tokensOut}) exceeds available supply (${availableTokens})`,
    );
  }

  // Calculate required input amount
  const k = virtualNative * virtualToken;
  const effectiveAmount = calculateRequiredAmountIn(
    tokensOut,
    k,
    virtualNative,
    virtualToken,
  );

  // Calculate fee (1%)
  const fee = (effectiveAmount * 10n) / 1000n;

  // Set deadline (short deadline for simplicity)
  const deadline = BigInt(Date.now() + 10000);

  // Execute transaction
  const txHash = await walletClient.writeContract({
    address: CONTRACT_ADDRESSES.CORE as `0x${string}`,
    abi: NadFunAbi.ICore,
    functionName: "exactOutBuy",
    args: [
      effectiveAmount + fee,
      tokensOut,
      tokenAddress as `0x${string}`,
      address as `0x${string}`,
      deadline,
    ],
    value: effectiveAmount + fee,
    chain: walletClient.chain,
    account: walletClient.account!,
  });

  return txHash;
}
```

### Sell Tokens to Bonding Curve

Selling tokens back to the bonding curve:

```typescript
/**
 * Sells tokens to a bonding curve via Core
 * @param privateKey - Private key of the sender
 * @param tokenAddress - Token address to sell
 * @param amount - Amount of tokens to sell
 * @returns Transaction hash
 */
export async function sellToCore(
  privateKey: string,
  tokenAddress: string,
  amount: string,
): Promise<string> {
  const walletClient = createWalletClientFromPrivateKey(privateKey);
  const publicClient = createPublicRpcClient();

  const coreAbi = NadFunAbi.ICore;
  const erc20Abi = NadFunAbi.IToken;
  const coreAddress = CONTRACT_ADDRESSES.CORE as `0x${string}`;

  const amountIn = parseEther(amount);
  const deadline = BigInt(Math.floor(Date.now() / 1000) + 20 * 60);
  const to = walletClient.account?.address;

  // Check current token allowance
  const currentAllowance = (await publicClient.readContract({
    address: tokenAddress as `0x${string}`,
    abi: erc20Abi,
    functionName: "allowance",
    args: [walletClient.account?.address, coreAddress],
  })) as bigint;

  // Approve tokens if needed
  if (currentAllowance < amountIn) {
    const maxApproveAmount = parseEther(
      "115792089237316195423570985008687907853269984665640564039457584007913129639935", // Max uint256
    );

    const approveTxHash = await walletClient.writeContract({
      address: tokenAddress as `0x${string}`,
      abi: erc20Abi,
      functionName: "approve",
      args: [coreAddress, maxApproveAmount],
      chain: walletClient.chain,
      account: walletClient.account!,
    });

    await publicClient.waitForTransactionReceipt({ hash: approveTxHash });
  }

  // Execute sell transaction
  const txHash = await walletClient.writeContract({
    address: coreAddress,
    abi: coreAbi,
    functionName: "sell",
    args: [amountIn, tokenAddress as `0x${string}`, to, deadline],
    chain: walletClient.chain,
    account: walletClient.account!,
  });

  return txHash;
}
```

## DEX Operations

### Buy Tokens from DEX

Purchase tokens from the DEX after they are listed:

```typescript
/**
 * Buys tokens from Uniswap/DEX
 * Only available after the token is listed
 * @param privateKey - Private key of the sender
 * @param tokenAddress - Token address to buy
 * @param amount - Amount of MON to spend
 * @param slippage - Slippage percentage (default 0.5%)
 * @returns Transaction hash
 */
export async function buyFromDex(
  privateKey: string,
  tokenAddress: string,
  amount: string,
  slippage: number = 0.5,
): Promise<string> {
  // Create wallet client
  const walletClient = createWalletClientFromPrivateKey(privateKey);
  const publicClient = createPublicRpcClient();

  // Check if token is listed
  const tokenInfo = await fetch(
    `https://testnet-bot-api-server.nad.fun/token/${tokenAddress}`,
  );
  const tokenData = await tokenInfo.json();

  if (!tokenData.is_listing) {
    throw new Error(
      "Token is not yet listed on DEX. Use bonding curve functions to purchase.",
    );
  }

  // Get Router ABI
  const routerAbi = NadFunAbi.IUniswapV2Router;

  // Parse amount to wei
  const valueInWei = parseEther(amount);

  // Calculate deadline (20 minutes from now)
  const deadline = BigInt(Math.floor(Date.now() / 1000) + 20 * 60);

  // Get expected output amount
  const path = [
    CONTRACT_ADDRESSES.WRAPPED_MON as `0x${string}`,
    tokenAddress as `0x${string}`,
  ];
  const amounts = await publicClient.readContract({
    address: CONTRACT_ADDRESSES.UNISWAP_V2_ROUTER as `0x${string}`,
    abi: routerAbi,
    functionName: "getAmountsOut",
    args: [valueInWei, path],
  });

  // Calculate minimum amount with slippage
  const expectedTokenAmount = (amounts as bigint[])[1];
  const slippageFactor = 1000n - BigInt(Math.floor(slippage * 10));
  const minTokens = (expectedTokenAmount * slippageFactor) / 1000n;

  // Submit transaction
  const txHash = await walletClient.writeContract({
    address: CONTRACT_ADDRESSES.UNISWAP_V2_ROUTER as `0x${string}`,
    abi: routerAbi,
    functionName: "swapExactNativeForTokens",
    args: [minTokens, path, walletClient.account?.address, deadline],
    value: valueInWei,
    chain: walletClient.chain,
    account: walletClient.account!,
  });

  return txHash;
}
```

### Sell Tokens to DEX

Sell tokens to the DEX after they are listed:

```typescript
/**
 * Sells tokens to Uniswap/DEX
 * Only available after the token is listed
 * @param privateKey - Private key of the sender
 * @param tokenAddress - Token address to sell
 * @param amount - Amount of tokens to sell
 * @param slippage - Slippage percentage (default 0.5%)
 * @returns Transaction hash
 */
export async function sellToDex(
  privateKey: string,
  tokenAddress: string,
  amount: string,
  slippage: number = 0.5,
): Promise<string> {
  // Create wallet client
  const walletClient = createWalletClientFromPrivateKey(privateKey);
  const publicClient = createPublicRpcClient();

  // Check if token is listed
  const tokenInfo = await fetch(
    `https://testnet-bot-api-server.nad.fun/token/${tokenAddress}`,
  );
  const tokenData = await tokenInfo.json();

  if (!tokenData.is_listing) {
    throw new Error(
      "Token is not yet listed on DEX. Use bonding curve functions to sell.",
    );
  }

  // Get Router and Token ABIs
  const routerAbi = NadFunAbi.IUniswapV2Router;
  const tokenAbi = NadFunAbi.IToken;

  // Parse amount to wei
  const tokenAmountInWei = parseEther(amount);

  // Calculate deadline (20 minutes from now)
  const deadline = BigInt(Math.floor(Date.now() / 1000) + 20 * 60);

  // First approve tokens to Router
  const approveTxHash = await walletClient.writeContract({
    address: tokenAddress as `0x${string}`,
    abi: tokenAbi,
    functionName: "approve",
    args: [
      CONTRACT_ADDRESSES.UNISWAP_V2_ROUTER as `0x${string}`,
      tokenAmountInWei,
    ],
    chain: walletClient.chain,
    account: walletClient.account!,
  });

  // Wait for approval confirmation
  await publicClient.waitForTransactionReceipt({ hash: approveTxHash });

  // Get expected output amount
  const path = [
    tokenAddress as `0x${string}`,
    CONTRACT_ADDRESSES.WRAPPED_MON as `0x${string}`,
  ];
  const amounts = await publicClient.readContract({
    address: CONTRACT_ADDRESSES.UNISWAP_V2_ROUTER as `0x${string}`,
    abi: routerAbi,
    functionName: "getAmountsOut",
    args: [tokenAmountInWei, path],
  });

  // Calculate minimum amount with slippage
  const expectedAmount = (amounts as bigint[])[1];
  const slippageFactor = 1000n - BigInt(Math.floor(slippage * 10));
  const minAmount = (expectedAmount * slippageFactor) / 1000n;

  // Submit transaction
  const txHash = await walletClient.writeContract({
    address: CONTRACT_ADDRESSES.UNISWAP_V2_ROUTER as `0x${string}`,
    abi: routerAbi,
    functionName: "swapExactTokensForNative",
    args: [
      tokenAmountInWei,
      minAmount,
      path,
      walletClient.account?.address,
      deadline,
    ],
    chain: walletClient.chain,
    account: walletClient.account!,
  });

  return txHash;
}
```

## Helper Functions

### Create Clients

```typescript
/**
 * Creates a wallet client from a private key
 * @param privateKey - Private key to use
 * @returns Viem wallet client
 */
export function createWalletClientFromPrivateKey(
  privateKey: string,
): WalletClient {
  const account = privateKeyToAccount(privateKey as `0x${string}`);

  return createWalletClient({
    account,
    chain: BLOCKCHAIN_CONFIG.chain,
    transport: http(BLOCKCHAIN_CONFIG.rpcUrl),
  });
}

/**
 * Creates a public client for read-only operations
 * @returns Viem public client
 */
export function createPublicRpcClient(): PublicClient {
  return createPublicClient({
    chain: BLOCKCHAIN_CONFIG.chain,
    transport: http(BLOCKCHAIN_CONFIG.rpcUrl),
  });
}
```

### Calculate Required Input for Exact Output

```typescript
/**
 * Calculate required input amount for bonding curve's exactOutBuy
 * @param tokensOut - Desired token output amount
 * @param k - Constant product (virtualNative * virtualToken)
 * @param virtualNative - Virtual native token balance
 * @param virtualToken - Virtual token balance
 * @returns Required input amount
 */
function calculateRequiredAmountIn(
  tokensOut: bigint,
  k: bigint,
  virtualNative: bigint,
  virtualToken: bigint,
): bigint {
  // Formula: (k / (virtualToken - tokensOut)) - virtualNative
  return k / (virtualToken - tokensOut) - virtualNative;
}
```

## Implementation Examples

### Complete Usage Flow

```typescript
async function tradingExample() {
  const privateKey = "0xyourprivatekey";
  const tokenAddress = "0x1234567890abcdef1234567890abcdef12345678";

  try {
    // First, get token market information
    const tokenMarketInfo = await fetch(
      `https://testnet-bot-api-server.nad.fun/token/market/${tokenAddress}`,
    );
    const marketData = await tokenMarketInfo.json();

    // Check if the token is in bonding curve phase or DEX phase
    if (marketData.market_type === "CURVE") {
      console.log("Token is in bonding curve phase");

      // Check available tokens
      const reserveToken = marketData.reserve_token
        ? BigInt(marketData.reserve_token)
        : 0n;
      const targetToken = marketData.target_token
        ? BigInt(marketData.target_token)
        : 0n;
      const availableTokens = reserveToken - targetToken;

      console.log(`Available tokens: ${availableTokens}`);

      if (availableTokens > 0n) {
        // Option 1: Buy with MON amount
        const buyTxHash = await buyFromCore(privateKey, tokenAddress, "1.0");
        console.log(`Buy transaction hash: ${buyTxHash}`);

        // Option 2: Buy exact tokens (useful for buying remaining tokens)
        if (availableTokens <= parseEther("1000")) {
          // If only a small amount remains, buy all to trigger DEX listing
          const exactBuyTxHash = await exactOutBuyFromCore(
            privateKey,
            tokenAddress,
            availableTokens,
          );
          console.log(`Exact buy transaction hash: ${exactBuyTxHash}`);
          console.log("Token should now be listed on DEX!");
        }
      } else {
        console.log(
          "No tokens available in bonding curve. Token might be listed on DEX.",
        );
      }
    } else if (marketData.market_type === "DEX") {
      console.log("Token is listed on DEX");

      // Buy tokens from DEX
      const buyTxHash = await buyFromDex(privateKey, tokenAddress, "1.0", 0.5);
      console.log(`DEX buy transaction hash: ${buyTxHash}`);

      // Sell tokens to DEX
      const sellTxHash = await sellToDex(
        privateKey,
        tokenAddress,
        "100.0",
        0.5,
      );
      console.log(`DEX sell transaction hash: ${sellTxHash}`);
    }
  } catch (error) {
    console.error("Error:", error.message);
  }
}
```

### Important Notes

1. **Token Trading Lifecycle**:

   - During bonding phase: Tokens can be purchased but cannot be transferred
   - After the last token is purchased through `exactOutBuy`, the token is automatically listed on DEX
   - Once listed on DEX, tokens can be transferred like regular ERC-20 tokens

2. **Calculating Available Tokens**:

   ```typescript
   // Critical calculation for determining remaining tokens in bonding curve
   const availableTokens = reserveToken - targetToken;

   // Check before attempting exactOutBuy
   if (tokensOut > availableTokens) {
     throw new Error(
       `Requested token amount exceeds available supply. Maximum available: ${availableTokens}`,
     );
   }
   ```

3. **Phase Transition**:

   - The transition from bonding curve to DEX is automatic once all tokens are purchased
   - Use `exactOutBuy` to purchase the exact remaining amount

4. **Fee Handling**:

   - Bonding curve: 1% fee is added to the transaction value
   - DEX: Standard DEX fees apply (typically 0.3%)

5. **Slippage Protection**:
   - For DEX trades, always use appropriate slippage settings (default 0.5%)
   - For bonding curve, price is deterministic based on the formula
