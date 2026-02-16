---
layout: post
title: "Introducing SAGE: A Language for the Age of AI-Powered Development"
date: 2026-02-15
description: "SAGE is a specification language that blends natural language with formal notation, designed for AI-assisted development."
tags: [sage, formal-methods, ai, llm, specification, lean]
---

**TL;DR**: SAGE (Semi-formal Agent Guidance & Engineering) blends natural language with formal notation, scales from quick prototypes to verified systems, and uses **38% fewer tokens** than natural language prompts. Now implemented in Lean 4.

-----

## The Problem

When you ask an AI to "build a user authentication system," what do you get? Maybe it works. But does it hash passwords with bcrypt? Implement rate limiting? Handle edge cases?

**The quality of AI-generated code depends on your specification.**

Traditional programming languages are for *implementations*, not *specifications*. Formal languages like TLA+ are powerful but steep. We needed something in between—accessible, flexible, and LLM-optimized.

That's SAGE.

-----

## Three Levels of Formality

SAGE lets you choose your level:

### Level 0: Natural Language

```
"Build a user authentication system"
"Passwords must be hashed with bcrypt"
!! "Use PostgreSQL for storage"
```

The `!!` marks non-negotiable decisions. Hand this to an LLM and get working code.

### Level 1: Structured Specs

```
@type User = { id: Int, email: Str, password_hash: Str }

@fn register(email: Str, password: Str) -> Result<User>
@req email.contains("@") && password.len() >= 8
@ens "Password is hashed, never stored in plaintext"

!!let hash = crypto.bcrypt_hash(password, cost: 12)
```

Types, contracts (`@req`/`@ens`), and explicit decisions. This is production-grade.

### Level 2: Formal Refinement

```
@spec UserAuth
@invariant ∀ u1,u2 ∈ users: u1.email ≠ u2.email

@refine UserAuth as SecureUserAuth
@decisions
  !! "Use bcrypt with cost 12"
@preserves ✓ "No duplicate emails"
```

Mathematical invariants, design tracking, refinement proofs. For mission-critical systems.

**Start at Level 0. Add rigor when you need it.**

-----

## Token Efficiency

SAGE is 38% more compact than natural language:

| Approach | Tokens |
|----------|--------|
| Java + Javadoc | ~850 |
| **SAGE** | **~380** |

Less boilerplate, more signal. LLMs generate better code from SAGE specs.

-----

## Why Lean 4?

SAGE is implemented in **Lean 4**—a theorem prover and programming language. Why?

- **Real performance** — Compiled code, not just proofs
- **Future verification** — We can eventually *prove* generated code satisfies specs
- **It fits** — Dependent types model exactly what SAGE expresses

My background is formal verification (NASA, CMU CyLab, AWS Automated Reasoning, SeaHorn, JayHorn). Lean lets us bridge the gap between practical tools and rigorous methods.

**Architecture:**
```
Source → Lexer → Parser → AST → Type Checker → LSP Server
```

All Lean 4, with comprehensive tests and VS Code integration.

-----

## Getting Started

### Install

```bash
# 1. Install Lean 4
curl https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh -sSf | sh

# 2. Clone and build
git clone https://github.com/lememta/sage-lang
cd sage-lang
./build.sh

# 3. Try it
.lake/build/bin/sage examples/level1-structured.sage

# 4. VS Code extension
./scripts/setup-vscode-lsp.sh
```

### Your First Spec

```
@mod auth

@fn login(email: Str, password: Str) -> Result<Session>
@req email.is_valid()
@ens "Session expires in 24 hours"

!! "Use bcrypt"
!! "Rate limit: 5 attempts/minute"
```

### Learn More

- **GitHub**: [github.com/lememta/sage-lang](https://github.com/lememta/sage-lang)
- **Docs**: `QUICKSTART.md`, `USAGE.md`, `docs/tutorial.md`

-----

## The Vision

SAGE is a bet on the future:

- **Specs are first-class** — Not afterthoughts
- **Formality scales** — From napkin to proof
- **AI collaboration** — Better specs → better code

The compiler is open source. Contributions welcome.

**Try SAGE today**: [github.com/lememta/sage-lang](https://github.com/lememta/sage-lang)
