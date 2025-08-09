---
title: "LotteryDapp"
description: "A decentralized, trustless Lottery Dapp"
date: 2023-07-21
tech: ["Solidity", "Hardhat", "ethers.js", "React.js", "Chainlink"]
layout: default
---

# LotteryDapp — Secure, Fair Blockchain Lottery

## Overview
**LotteryDapp** is a blockchain-based lottery where players join by paying an entry fee.  
At set intervals, a random winner is picked using **Chainlink VRF**, ensuring results are provably fair.  
No central authority controls the draw or payouts — all logic runs in a public smart contract.

---

## Key Features
- **Provably Fair Randomness** — Winner selected via Chainlink VRF.
- **Automated Draws** — Supports Chainlink Keepers or manual triggers.
- **Simple Entry** — Call `enter()` with the entry fee to join.
- **Full Pot Payout** — Winner receives the collected funds.
- **Event Tracking** — Emits events for entries, draws, and payouts.
- **Live Frontend** — React.js UI showing pot size, players, and past winners.

---

## Security Highlights
- **Randomness Integrity** — No block-based randomness; VRF prevents prediction or manipulation.
- **Safe Payouts** — Uses Checks-Effects-Interactions pattern with optional pull payments.
- **Access Control** — Only authorized admin can change key settings; ownership can be renounced.
- **Gas Safety** — No unbounded loops; player limit keeps transactions efficient.
- **Recovery Mode** — Handles oracle failure with refund or manual draw options.

---

## Limitations
- **Sybil Risk** — One address = one entry, but users can create multiple addresses.
- **Oracle Dependency** — Requires Chainlink VRF for randomness; fallback needed if unavailable.

---

## Tech Stack
- Solidity — Core lottery logic.
- Hardhat — Development & testing.
- Ethers.js — Blockchain interaction.
- React.js — Frontend UI.
- Chainlink VRF + Keepers — Randomness & automation.

---

🔗 [GitHub Repository](https://github.com/BLOCK-PROGRAMR/LOTTERYDAP)
