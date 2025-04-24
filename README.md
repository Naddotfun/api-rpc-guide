# Nad.fun API & RPC Interaction Guide

This repository contains a comprehensive guide for interacting with the Nad.fun platform through blockchain transactions on the Monad chain. It provides detailed information and code examples for token trading via bonding curve mechanisms and DEX operations.

## Quick Links

- [Full Documentation](./naddofun-rpc-interaction-info-for-mcp.md)
- [Understanding Nad.fun Trading Lifecycle](./naddofun-rpc-interaction-info-for-mcp.md#understanding-nadfun-trading-lifecycle)
- [Contract ABI](./naddofun-rpc-interaction-info-for-mcp.md#contract-abi)
- [Bonding Curve Operations](./naddofun-rpc-interaction-info-for-mcp.md#bonding-curve-operations)
- [DEX Operations](./naddofun-rpc-interaction-info-for-mcp.md#dex-operations)
- [Helper Functions](./naddofun-rpc-interaction-info-for-mcp.md#helper-functions)
- [Implementation Examples](./naddofun-rpc-interaction-info-for-mcp.md#implementation-examples)

## Overview

The Nad.fun platform utilizes a unique token trading mechanism with two distinct phases:

1. **Bonding Curve Phase**: Initial token offerings through a mathematical bonding curve
2. **DEX Phase**: After all bonding curve tokens are sold, automatic listing on the DEX

This guide provides TypeScript implementation examples for interacting with both phases, including:

- Installing and using the contract ABIs
- Buying tokens from a bonding curve
- Purchasing exact token amounts to trigger DEX listing
- Selling tokens back to a bonding curve
- Trading tokens on the DEX after listing
- Helper functions for wallet creation and calculations

## Getting Started

To get started, navigate to the [full documentation](./naddofun-rpc-interaction-info-for-mcp.md) and follow the installation instructions for adding the contract ABIs to your project.

## For the Monad Devs MCP Challenge

This guide is specifically created for the Monad Devs MCP challenge participants to facilitate integration with the Nad.fun platform.
