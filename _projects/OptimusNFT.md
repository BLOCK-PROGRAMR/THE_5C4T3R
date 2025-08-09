---
title: "OptimusNFT"
description: "ERC721 Token - Optimus Testing NFT"
date: 2023-06-05
tech: ["Solidity", "Foundry", "Ethereum"]
layout: default
---

# OptimusNFT — Secure ERC-721 Deployment

## Overview
**OptimusNFT** is a custom **ERC-721** NFT contract built with **Foundry** and deployed on the Ethereum Goerli testnet.  
It integrates with **OpenSea** for marketplace compatibility and focuses on **security-first design** — preventing common NFT vulnerabilities through strict access controls, immutable metadata, and safe minting logic.

---

## Key Features
- **ERC-721 + ERC-2981** — NFT standard with royalty support.
- **Safe Public Minting** — Supply cap and price validation.
- **Marketplace Ready** — IPFS-hosted metadata for decentralization.
- **Immutable Metadata** — Prevents post-deployment tampering.

---

## Security Highlights
- **Reentrancy Prevention** — CEI pattern + `nonReentrant`.
- **Strict Access Control** — `Ownable` for admin functions.
- **No Unbounded Loops** — Avoids gas DoS risks.
- **Metadata Protection** — Immutable base URI after deployment.

---

## Tech Stack
- Solidity — Smart contract logic.
- Foundry — Development & testing.
- Ethereum (Goerli) — Deployment network.
- IPFS — Metadata storage.

---

🔗 [GitHub Repository](https://github.com/BLOCK-PROGRAMR/Optimus_NFT)
