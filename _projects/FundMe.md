---
title: "Fund Me"
description: "ETH funding smart contract with real-time ETH/USD conversion"
date: 2024-03-15
tech: ["Solidity", "Foundry", "Chainlink"]
layout: default
---

# Fund Me — ETH Crowdfunding with Price Feeds

## Overview
**Fund Me** is a Solidity-based smart contract that accepts ETH contributions and calculates their USD value in real time using a **Chainlink ETH/USD price feed**.  
It is built and deployed using **Foundry** — a fast, modular Ethereum development toolkit written in Rust.

---

## Key Features
- **Fund Contract** — Users send ETH via the `fund()` function.
- **USD Conversion** — Real-time price data from Chainlink oracles.
- **Owner Withdraw** — Only the contract owner can withdraw collected funds.
- **Minimum Funding Requirement** — Enforces a minimum USD amount before accepting contributions.

---

## Tech Stack
- **Solidity** — Smart contract logic.
- **Foundry** — Development, testing, and deployment.
- **Chainlink Price Feeds** — ETH/USD conversion.
- **Sepolia Testnet** — Deployment and testing network.

---

## Security Highlights
- **Access Control** — Only the owner can withdraw funds.
- **Price Feed Validation** — Reliable Chainlink oracle data.
- **Gas Efficiency** — Optimized contract structure.
- **Test Coverage** — Unit tests for funding, withdrawal, and price conversion logic.

---

## Typical Workflow
1. **Fund** — User calls `fund()` with ETH.
2. **Price Check** — Contract checks USD value using Chainlink.
3. **Store Contribution** — Records sender and amount.
4. **Withdraw** — Owner retrieves total funds.

---

📦 [Github](https://github.com/BLOCK-PROGRAMR/FundTest)
