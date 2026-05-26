---
title: "x86-64 Assembly: Function Parameters and Calling Conventions"
date: 2026-05-26
author: "grit8086"
tags: ["reverse-engineering", "assembly", "x86-64", "research"]
draft: false
---

*Originally published by [grit8086](https://grit8086.github.io)*

A deep walkthrough of how x86-64 functions pass arguments, manage the stack, and follow calling conventions — covering both Microsoft x64 ABI and System V AMD64 ABI, shadow space, LEA instruction internals, and frame pointer mechanics.

Topics covered:
- Single and multiple parameter passing
- Stack layout diagrams for 32-bit and 64-bit
- Caller-save vs callee-save registers
- Shadow space explained
- The LEA instruction and RMX addressing forms
- Compiler optimizations using LEA

[Read the full report →](https://grit8086.github.io/posts/function-parameters/)
