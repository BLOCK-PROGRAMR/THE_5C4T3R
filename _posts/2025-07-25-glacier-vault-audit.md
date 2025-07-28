---
layout: post
title: "GlacierVault CTF Challenge Audit"
date: 2025-07-25
categories: audits
---

This audit covers the GlacierVault CTF challenge.

The key vulnerability lies in the way collateralized flashloans were validated. There was an incorrect assumption about the token supply and untrusted execution in a delegate call.

### Vulnerabilities:
- Flashloan logic abused by custom ERC20 tokens.
- Delegated external call without sanity checks.
