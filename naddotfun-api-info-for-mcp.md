# Nad.fun API Documentation for MCP

## Overview

This document provides comprehensive information about the Nad.fun API, which can be used to retrieve token information, market data, account positions, and more from the Nad.fun platform. The API is designed to be used with MCP (Monad Compute Provider) implementations.

The base API endpoint is: `https://testnet-bot-api-server.nad.fun/`

## Table of Contents

- [Account Endpoints](#account-endpoints)
- [Order Endpoints](#order-endpoints)
- [Token Endpoints](#token-endpoints)
- [API Examples](#api-examples)

## Account Endpoints

### Get Account Positions

Retrieves token positions held by an account.

**Endpoint:** `GET /account/position/{account_address}`

**Path Parameters:**

- `account_address` (string): The EOA address

**Query Parameters:**

- `position_type` (enum: all, open, close): Filter by position type
- `page` (integer, optional, default=1): Page number
- `limit` (integer, optional, default=10): Number of items per page

**Example Request:**

```
GET /account/position/0x9876543210abcdef9876543210abcdef98765432?position_type=open&page=1&limit=10
```

**Example Response:**

```json
{
  "account_address": "0x9876543210abcdef9876543210abcdef98765432",
  "positions": [
    {
      "token": {
        "token_address": "0x1234567890abcdef1234567890abcdef12345678",
        "name": "My Token",
        "symbol": "MTK",
        "image_uri": "https://storage.example.com/images/mytoken.png",
        "creator": "0xabcdef1234567890abcdef1234567890abcdef12",
        "total_supply": "100000000",
        "created_at": 1713894357
      },
      "position": {
        "total_bought_native": "10000",
        "total_bought_token": "5000000",
        "current_token_amount": "4850000",
        "realized_pnl": "5.00",
        "unrealized_pnl": "2.50",
        "total_pnl": "7.50",
        "created_at": 1713894357,
        "last_traded_at": 1713897957
      },
      "market": {
        "market_address": "0xabcdef1234567890abcdef1234567890abcdef12",
        "market_type": "DEX",
        "price": "0.28"
      }
    }
  ],
  "total_count": 5
}
```

### Get Tokens Created by Account

Retrieves tokens created by a specific account.

**Endpoint:** `GET /account/create_token/{account_address}`

**Path Parameters:**

- `account_address` (string): The EOA address

**Query Parameters:**

- `page` (integer, optional, default=1): Page number
- `limit` (integer, optional, default=10): Number of items per page

**Example Request:**

```
GET /account/create_token/0x9876543210abcdef9876543210abcdef98765432?page=1&limit=10
```

**Example Response:**

```json
{
  "tokens": [
    {
      "token": {
        "token_address": "0x1234567890abcdef1234567890abcdef12345678",
        "name": "My Created Token",
        "symbol": "MCT",
        "image_uri": "https://storage.example.com/images/mycreatedtoken.png",
        "creator": "0x9876543210abcdef9876543210abcdef98765432",
        "total_supply": "50000000",
        "created_at": 1713894357
      },
      "is_listing": true,
      "market_cap": "50000000",
      "price": "0.10",
      "current_amount": "10000000",
      "description": "A token created by me for demonstration purposes"
    }
  ],
  "total_count": 3
}
```

## Order Endpoints

### Get Tokens by Creation Time

Retrieves tokens ordered by their creation time (newest first).

**Endpoint:** `GET /order/creation_time`

**Query Parameters:**

- `page` (integer, optional, default=1): Page number
- `limit` (integer, optional, default=10): Number of items per page

**Example Request:**

```
GET /order/creation_time?page=1&limit=10
```

**Example Response:**

```json
{
  "order_type": "CreationTime",
  "order_token": [
    {
      "token_info": {
        "token_address": "0x1234567890abcdef1234567890abcdef12345678",
        "name": "Newest Token",
        "symbol": "NEW",
        "image_uri": "https://storage.example.com/images/newest.png",
        "creator": "0xabcdef1234567890abcdef1234567890abcdef12",
        "total_supply": "100000000",
        "created_at": 1713894357
      },
      "market_info": {
        "market_address": "0xabcdef1234567890abcdef1234567890abcdef12",
        "market_type": "CURVE",
        "price": "0.28"
      }
    }
  ],
  "total_count": 150
}
```

### Get Tokens by Market Cap

Retrieves tokens ordered by their market capitalization (highest first).

**Endpoint:** `GET /order/market_cap`

**Query Parameters:**

- `page` (integer, optional, default=1): Page number
- `limit` (integer, optional, default=10): Number of items per page

**Example Request:**

```
GET /order/market_cap?page=1&limit=10
```

**Example Response:**

```json
{
  "order_type": "MarketCap",
  "order_token": [
    {
      "token_info": {
        "token_address": "0x1234567890abcdef1234567890abcdef12345678",
        "name": "Big Market Cap Token",
        "symbol": "BMCT",
        "image_uri": "https://storage.example.com/images/big-market-cap.png",
        "creator": "0xabcdef1234567890abcdef1234567890abcdef12",
        "total_supply": "50000000",
        "created_at": 1713784357
      },
      "market_info": {
        "market_address": "0xabcdef1234567890abcdef1234567890abcdef12",
        "market_type": "CURVE",
        "price": "1.20"
      }
    }
  ],
  "total_count": 150
}
```

### Get Tokens by Latest Trade

Retrieves tokens ordered by their most recent trade.

**Endpoint:** `GET /order/latest_trade`

**Query Parameters:**

- `page` (integer, optional, default=1): Page number
- `limit` (integer, optional, default=10): Number of items per page

**Example Request:**

```
GET /order/latest_trade?page=1&limit=10
```

**Example Response:**

```json
{
  "order_type": "LatestTrade",
  "order_token": [
    {
      "token_info": {
        "token_address": "0x1234567890abcdef1234567890abcdef12345678",
        "name": "Recently Traded Token",
        "symbol": "RTT",
        "image_uri": "https://storage.example.com/images/recently-traded.png",
        "creator": "0xabcdef1234567890abcdef1234567890abcdef12",
        "total_supply": "80000000",
        "created_at": 1713694357
      },
      "market_info": {
        "market_address": "0xabcdef1234567890abcdef1234567890abcdef12",
        "market_type": "CURVE",
        "price": "0.75"
      }
    }
  ],
  "total_count": 150
}
```

## Token Endpoints

### Get Token Metadata

Retrieves detailed metadata for a specific token.

**Endpoint:** `GET /token/{token}`

**Path Parameters:**

- `token` (string): Token contract address

**Example Request:**

```
GET /token/0x1234567890abcdef1234567890abcdef12345678
```

**Example Response:**

```json
{
  "token_address": "0x1234567890abcdef1234567890abcdef12345678",
  "name": "My Token",
  "symbol": "MTK",
  "image_uri": "https://storage.example.com/images/mytoken.png",
  "creator_address": "0xabcdef1234567890abcdef1234567890abcdef12",
  "description": "A sample token for demonstration purposes",
  "twitter": "https://twitter.com/mytoken",
  "telegram": "https://t.me/mytoken",
  "website": "https://mytoken.example.com",
  "is_listing": true,
  "total_supply": "100000000",
  "created_at": 1713894357,
  "create_transaction_hash": "0x9876543210fedcba9876543210fedcba98765432"
}
```

### Get Token Chart Data

Retrieves price chart data for a token.

**Endpoint:** `GET /token/chart/{token}`

**Path Parameters:**

- `token` (string): Token contract address

**Query Parameters:**

- `interval` (string: 1m, 5m, 15m, 30m, 1h, 4h, 1d, 1w): Chart interval
- `base_timestamp` (integer): Base timestamp

**Example Request:**

```
GET /token/chart/0x1234567890abcdef1234567890abcdef12345678?interval=1h&base_timestamp=1713897600
```

**Example Response:**

```json
{
  "data": [
    {
      "time_stamp": 1713894000,
      "open_price": "0.25",
      "close_price": "0.27",
      "high_price": "0.28",
      "low_price": "0.24",
      "volume": "15000000"
    },
    {
      "time_stamp": 1713897600,
      "open_price": "0.27",
      "close_price": "0.29",
      "high_price": "0.30",
      "low_price": "0.26",
      "volume": "1850000"
    }
  ],
  "token_id": "0x1234567890abcdef1234567890abcdef12345678",
  "interval": "1h",
  "base_timestamp": 1713897600,
  "total_count": 24
}
```

### Get Token Swap History

Retrieves swap history for a specific token.

**Endpoint:** `GET /token/swap/{token}`

**Path Parameters:**

- `token` (string): Token contract address

**Query Parameters:**

- `page` (integer, optional, default=1): Page number
- `limit` (integer, optional, default=10): Number of items per page

**Example Request:**

```
GET /token/swap/0x1234567890abcdef1234567890abcdef12345678?page=1&limit=10
```

**Example Response:**

```json
{
  "swaps": [
    {
      "swap_id": 12345,
      "account_address": "0x9876543210abcdef9876543210abcdef98765432",
      "token_address": "0x1234567890abcdef1234567890abcdef12345678",
      "is_buy": true,
      "mon_amount": "500",
      "token_amount": "10000",
      "created_at": 1713894357,
      "transaction_hash": "0xabcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890"
    },
    {
      "swap_id": 12346,
      "account_address": "0xfedcba9876543210fedcba9876543210fedcba98",
      "token_address": "0x1234567890abcdef1234567890abcdef12345678",
      "is_buy": false,
      "mon_amount": "300",
      "token_amount": "5000",
      "created_at": 1713897957,
      "transaction_hash": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"
    }
  ],
  "total_count": 42
}
```

### Get Token Market Information

Retrieves market data for a specific token.

**Endpoint:** `GET /token/market/{token}`

**Path Parameters:**

- `token` (string): Token contract address

**Example Request:**

```
GET /token/market/0x1234567890abcdef1234567890abcdef12345678
```

**Example Response:**

```json
{
  "market_id": "0xabcdef1234567890abcdef1234567890abcdef12",
  "market_type": "CURVE",
  "token_address": "0x1234567890abcdef1234567890abcdef12345678",
  "virtual_native": "28000000",
  "virtual_token": "100000000",
  "reserve_token": "5000000",
  "reserve_native": "1400000",
  "price": "0.28",
  "latest_trade_at": 1713894357,
  "created_at": 1713790000
}
```

### Get Token Holders

Retrieves the list of holders for a specific token.

**Endpoint:** `GET /token/holder/{token}`

**Path Parameters:**

- `token` (string): Token contract address

**Query Parameters:**

- `page` (integer, optional, default=1): Page number
- `limit` (integer, optional, default=10): Number of items per page

**Example Request:**

```
GET /token/holder/0x1234567890abcdef1234567890abcdef12345678?page=1&limit=10
```

**Example Response:**

```json
{
  "holders": [
    {
      "current_amount": "500000",
      "account_address": "0x9876543210abcdef9876543210abcdef98765432",
      "is_dev": true
    },
    {
      "current_amount": "350000",
      "account_address": "0xfedcba9876543210fedcba9876543210fedcba98",
      "is_dev": false
    }
  ],
  "total_count": 128
}
```

## API Examples

Here are some code examples to demonstrate how to use the Nad.fun API in JavaScript/TypeScript:

### Fetching Token Lists

```typescript
/**
 * Get tokens ordered by creation time
 * @returns List of tokens ordered by creation time
 */
async function getLatestTokens() {
  const response = await fetch(
    "https://testnet-bot-api-server.nad.fun/order/creation_time?page=1&limit=10",
  );
  const data = await response.json();
  return data.order_token;
}

/**
 * Get tokens ordered by market cap
 * @returns List of tokens ordered by market cap
 */
async function getTopMarketCapTokens() {
  const response = await fetch(
    "https://testnet-bot-api-server.nad.fun/order/market_cap?page=1&limit=10",
  );
  const data = await response.json();
  return data.order_token;
}

/**
 * Get tokens ordered by latest trade
 * @returns List of tokens ordered by latest trade
 */
async function getLatestTradedTokens() {
  const response = await fetch(
    "https://testnet-bot-api-server.nad.fun/order/latest_trade?page=1&limit=10",
  );
  const data = await response.json();
  return data.order_token;
}
```

### Fetching Token Details

```typescript
/**
 * Get token metadata
 * @param tokenAddress - The token contract address
 * @returns Token metadata
 */
async function getTokenInfo(tokenAddress) {
  const response = await fetch(
    `https://testnet-bot-api-server.nad.fun/token/${tokenAddress}`,
  );
  const data = await response.json();
  return data;
}

/**
 * Get token chart data
 * @param tokenAddress - The token contract address
 * @param interval - Chart interval (1m, 5m, 15m, 30m, 1h, 4h, 1d, 1w)
 * @returns Chart data for the token
 */
async function getTokenChart(tokenAddress, interval = "1h") {
  const baseTimestamp = Math.floor(Date.now() / 1000);
  const response = await fetch(
    `https://testnet-bot-api-server.nad.fun/token/chart/${tokenAddress}?interval=${interval}&base_timestamp=${baseTimestamp}`,
  );
  const data = await response.json();
  return data;
}

/**
 * Get token market information
 * @param tokenAddress - The token contract address
 * @returns Market data for the token
 */
async function getTokenMarket(tokenAddress) {
  const response = await fetch(
    `https://testnet-bot-api-server.nad.fun/token/market/${tokenAddress}`,
  );
  const data = await response.json();
  return data;
}

/**
 * Get token swap history
 * @param tokenAddress - The token contract address
 * @returns Swap history for the token
 */
async function getTokenSwapHistory(tokenAddress) {
  const response = await fetch(
    `https://testnet-bot-api-server.nad.fun/token/swap/${tokenAddress}?page=1&limit=10`,
  );
  const data = await response.json();
  return data.swaps;
}

/**
 * Get token holders
 * @param tokenAddress - The token contract address
 * @returns List of token holders
 */
async function getTokenHolders(tokenAddress) {
  const response = await fetch(
    `https://testnet-bot-api-server.nad.fun/token/holder/${tokenAddress}?page=1&limit=10`,
  );
  const data = await response.json();
  return data.holders;
}
```

### Fetching Account Information

```typescript
/**
 * Get account positions
 * @param accountAddress - The account EOA address
 * @param positionType - Position type filter (all, open, close)
 * @returns List of token positions for the account
 */
async function getAccountPositions(accountAddress, positionType = "open") {
  const response = await fetch(
    `https://testnet-bot-api-server.nad.fun/account/position/${accountAddress}?position_type=${positionType}&page=1&limit=10`,
  );
  const data = await response.json();
  return data.positions;
}

/**
 * Get tokens created by an account
 * @param accountAddress - The account EOA address
 * @returns List of tokens created by the account
 */
async function getAccountCreatedTokens(accountAddress) {
  const response = await fetch(
    `https://testnet-bot-api-server.nad.fun/account/create_token/${accountAddress}?page=1&limit=10`,
  );
  const data = await response.json();
  return data.tokens;
}
```

## Additional Resources

- Swagger UI: `https://testnet-bot-api-server.nad.fun/swagger-ui`
- OpenAPI JSON: `https://testnet-bot-api-server.nad.fun/api-docs/openapi.json`
