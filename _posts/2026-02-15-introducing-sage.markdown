---
layout: post
title: "Introducing SAGE: A Language for the Age of AI-Powered Development"
date: 2026-02-15
description: "SAGE (Semi-formal Agent Guidance & Engineering) is a new specification language designed specifically for the era of AI-assisted software development."
tags: [sage, formal-methods, ai, llm, specification]
---

**TL;DR**: SAGE (Semi-formal Agent Guidance & Engineering) is a new specification language designed specifically for the era of AI-assisted software development. It seamlessly blends natural language with formal notation, offers optional refinement for design tracking, and is optimized for both human readability and LLM token efficiency.

-----

## The Problem: Specifications in the Age of GenAI

We're living through a fundamental shift in how software gets written. With tools like Claude, GPT-4, and specialized code generation models, the bottleneck in software development is no longer *writing* code—it's *specifying* what to build.

Think about it: When you ask an AI to "build a user authentication system," what do you get? Something that works, probably. But does it:

- Hash passwords with bcrypt?
- Implement rate limiting?
- Use proper session management?
- Handle edge cases correctly?

The quality of AI-generated code depends entirely on the quality of your specification. **Traditional programming languages were designed for writing implementations, not specifications**. Meanwhile, formal specification languages like TLA+ and Alloy are powerful but have steep learning curves and aren't optimized for LLM consumption.

We needed something in between—accessible, flexible, and LLM-optimized. That's SAGE.

-----

## What is SAGE?

SAGE is a specification language with a radical idea: **mix natural language and formal notation in the same file, and make formality optional**.

```
@fn authenticate(email: Str, password: Str) -> Result<Session>

"First, find the user by email"
let user = db.find_user(email)
if user.is_none() =>
  "User doesn't exist - return generic error for security"
  ret Err("Invalid credentials")

"Verify password against stored hash"
!!let is_valid = crypto.bcrypt_verify(password, user.password_hash)
if !is_valid => ret Err("Invalid credentials")

"Create and return new session"
ret Ok(create_session(user))
```

Notice what's happening:
- Natural language explains *what* and *why*
- Type annotations provide safety
- The `!!` marker highlights critical operations
- Code and commentary flow together naturally

-----

## The Three Levels of SAGE

SAGE works at **three levels of formality**. You choose the level based on your needs:

### Level 0: Pure Natural Language

Perfect for initial design:

```
"Build a user authentication system"
"Users can register with email and password"
"Passwords must be hashed with bcrypt"
"Sessions expire after 24 hours"

!! "Use PostgreSQL for storage"
!! "Use Redis for rate limiting"
```

The `!!` markers tell the AI which decisions are non-negotiable.

### Level 1: Structured Specifications

This is where you'll spend 90% of your time:

```
@mod auth_service

@type User = { id: Int, email: Str, password_hash: Str }

@fn register(email: Str, password: Str) -> Result<User>
@req email.contains("@")
@req password.len() >= 8

"Check if user already exists"
let existing = db.find_by_email(email)
if existing.is_some() => ret Err("Email already in use")

"Hash password with bcrypt"
!!let hash = crypto.bcrypt_hash(password, cost: 12)

ret Ok({id: db.next_id(), email: email, password_hash: hash})
```

You get types, contracts (`@req`/`@ens`), natural language documentation, and explicit design decisions.

### Level 2: Formal Refinement

For mission-critical systems:

```
@spec UserAuth
@state
  users: Set<User>
  sessions: Set<Session>

@invariant "No duplicate emails"
  ∀ u1,u2 ∈ users: u1.email ≠ u2.email

@refine UserAuth as SecureUserAuth
@decisions
  !! "Use bcrypt with cost factor 12"
  "Store password hashes, never plaintext"

@preserves
  ✓ "No duplicate emails invariant still holds"
```

Mathematical invariants, tracked design decisions, and refinement proofs.

-----

## Why Refinement Matters in the AI Era

Refinement lets you start with *what* the system should do, then incrementally refine it to *how*, proving you've preserved the original properties.

In the age of AI-generated code, this becomes essential:

1. **Design decisions are prompts**: "Use bcrypt with cost 12" is an instruction
2. **Multiple implementations exist**: AI generates alternatives; refinement tracks which to choose
3. **Verification is critical**: How do you know AI-generated code is correct?
4. **Evolution is constant**: Requirements change; refinement tracks the evolution

**In SAGE, refinement is optional.** Start with Level 1. Add formal specs when needed.

-----

## Token Efficiency

SAGE is **38% more compact** than equivalent natural language:

**Java + Javadoc (~850 tokens):**
```java
/**
 * Authenticates a user with email and password.
 * @param email The user's email address
 * @param password The user's password
 * @return Result containing session or error
 */
public Result<Session> authenticate(String email, String password) {
    // First, look up the user by email
    Optional<User> userOpt = database.findUserByEmail(email);
    ...
}
```

**SAGE (~380 tokens):**
```
@fn authenticate(email: Str, password: Str) -> Result<Session>
"Look up user by email"
let user = db.find_user(email)
if user.is_none() => ret Err("Invalid credentials")
...
```

Less boilerplate, more signal. LLMs generate better code from SAGE specs.

-----

## The Philosophy: Formality is a Dial

SAGE treats formality as a **spectrum**:

```
Natural Language ←────────────────→ Full Formal Proof
     Level 0           Level 1           Level 2
   (Prototype)      (Production)   (Mission-Critical)
```

You can:
- Start informal and add rigor incrementally
- Use different levels for different parts of the system
- Keep it simple for straightforward code

This is how engineers actually think.

-----

## Getting Started

### Install

```bash
# Clone and build
git clone https://github.com/lememta/sage-lang
cd sage-lang
./build.sh

# Try the compiler
.lake/build/bin/sage examples/level1-structured.sage

# VS Code extension
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

## Join Us

SAGE is our bet on the future:

- **Specifications are first-class** — Not afterthoughts
- **Natural language is code** — Documentation and implementation unified
- **Formality is accessible** — You don't need a PhD to write correct software
- **AI is a collaborator** — Better specs → better generated code

The compiler is open source. We need engineers to try it, researchers to formalize it, and early adopters to push its limits.

**Try SAGE today**: [github.com/lememta/sage-lang](https://github.com/lememta/sage-lang)

-----

*SAGE is open source (MIT License). The future of software development is collaborative—between humans and AI, between formal and informal, between specification and implementation.*
