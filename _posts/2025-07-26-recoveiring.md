---
layout: post
title: "Exploiting ecrecover to Return Zero Address"
date: 2025-07-25
categories: analysis
---

In this writeup, we explore how to exploit `ecrecover` to return the zero address by using a signature with `(v, r, s) = (0, 0, 0)`.

This bug often occurs when signature verification fails but fallback logic accepts the recovered address — in this case, it's the zero address.

### Key Learnings:
- `ecrecover` can return `0x000...0000` if the input signature is invalid.
- Always validate the returned address.
