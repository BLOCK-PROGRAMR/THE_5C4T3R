---
title: "Voting Application"
description: "Decentralized, tamper-resistant voting"
date: 2023-09-25
tech: ["Solidity", "Hardhat", "Ethers.js", "React.js", "MetaMask"]
layout: default
---

# Voting Application — On-Chain Secure Election

## Overview
The **Voting Application** is a blockchain-based election system that records every vote directly on Ethereum.  
It supports two fixed candidates — **YCP** and **TDP** — and ensures results cannot be altered after deployment.  
Users connect with **MetaMask** to vote, and results are viewable in real time.

---

## Key Features
- **Immutable Candidates** — Candidate list is set at deployment and cannot be changed.
- **One Vote per Address** — Smart contract blocks double voting using a voter tracking mapping.
- **Transparent Results** — Anyone can view vote counts on-chain without a middleman.
- **No Admin Control** — No reset, override, or privileged vote functions exist.
- **MetaMask + React Frontend** — Simple UI for voting and viewing live tallies.

---

## Security Highlights
- **Vote Integrity** — All votes are permanent and public on-chain records.
- **Double-Vote Prevention** — Once an address votes, further attempts revert.
- **Admin Abuse Prevention** — Contract ownership is renounced after deployment.
- **DoS Resistance** — No heavy loops in voting or tallying; all reads are O(1).

---

## Limitations
- **Sybil Attacks** — One address = one vote, but multiple addresses can be created without identity checks.
- **Public Votes** — No privacy; votes are linked to Ethereum addresses.

---

## Tech Stack
- Solidity — Voting logic.
- Hardhat — Development & testing.
- Ethers.js — Blockchain interaction.
- React.js + Tailwind CSS — Frontend display.
- MetaMask — Authentication & signing.

---

🔗 [GitHub Repository](https://github.com/BLOCK-PROGRAMR/Block-vote)
