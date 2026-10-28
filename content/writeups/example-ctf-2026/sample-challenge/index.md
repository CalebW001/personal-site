---
title: "Sample Challenge"
date: 2026-10-28
draft: true
ctfs: ["Example CTF 2026"]
categories: ["pwn"]
tags: ["buffer-overflow", "ret2win"]
difficulty: "easy"
summary: "One-line summary of the challenge and the technique used."
---

## TL;DR

[One or two sentences: what the bug was and how you exploited it.]

## Challenge

> [Paste the challenge description here.]

Files: `chall`, `chall.c`

## Recon

[What you checked first and what you noticed.]

```bash
$ checksec ./chall
```

## Dead ends

[What you tried that didn't work, and why.]

## Solution

[Walk through the exploit and explain why each step works.]

```python
from pwn import *

p = process("./chall")
# ...
p.interactive()
```

Images go in this folder next to `index.md`, referenced like `![ghidra view](ghidra.png)`.

## Takeaways

[What you'd do faster next time and what you learned.]
