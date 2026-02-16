---
layout: post
title: "Introducing SAGE: Semi-formal AI-Guided Engineering"
date: 2026-02-15
description: "A specification language that bridges natural language and formal methods for AI-assisted development."
tags: [sage, formal-methods, ai, llm, specification]
---

I'm excited to announce **SAGE** (Semi-formal AI-Guided Engineering), a new specification language designed for the age of AI-assisted development.

## The Problem

When working with LLMs to generate code, we face a fundamental tension:

- **Natural language** is easy to write but imprecise — LLMs hallucinate, miss requirements, and make assumptions
- **Formal specifications** are precise but slow to write — most engineers won't use them for everyday work

What if we could have both?

## SAGE: Choose Your Level

SAGE is a specification language that scales from napkin sketches to formal verification. You choose your level of formality:

### Level 0: Just Write What You Want

```sage
"Build a todo app"
"Users can create tasks"
"Tasks can be completed or deleted"
!! "Use TypeScript and localStorage"
```

That's valid SAGE. The `!!` marks implementation decisions.

### Level 1: Add Structure When Ready

```sage
@type Task = { id: Str, title: Str, completed: Bool }

@fn create_task(title: Str) -> Task
@req title.len() > 0
@ens "Task created with completed = false"
```

Now we have types and contracts. An LLM can generate validated code from this.

### Level 2: Go Formal When It Matters

```sage
@spec PaymentSystem
@state accounts: Map<AccountId, Balance>
@invariant ∀ a ∈ accounts: a.balance >= 0
@invariant ∑ accounts.values() = TOTAL_MONEY
```

For critical systems, we can specify invariants that must always hold.

## Why It Works

SAGE is **38% more token-efficient** than equivalent natural language prompts. But more importantly, it produces better code because:

1. **Types become interfaces** — no guessing what data looks like
2. **`@req` becomes validation** — preconditions are enforced
3. **`@ens` becomes guarantees** — postconditions are implemented
4. **`!!` decisions are preserved** — tech choices aren't ignored

## Try It

The project is open source:

- **GitHub**: [github.com/lememta/sage-lang](https://github.com/lememta/sage-lang)
- **Parser**: TypeScript lexer and recursive descent parser
- **VS Code**: Syntax highlighting extension included
- **Tutorial**: Learn SAGE in 15 minutes

I've also included `AGENT.md` — a guide that teaches LLMs how to interpret SAGE specs. Drop it in your system prompt and your AI assistant will understand SAGE natively.

## The Vision

SAGE sits at the intersection of two worlds I've spent my career in:

1. **Formal methods** — making software provably correct (my work at NASA, CMU, and AWS)
2. **AI/ML systems** — building intelligent systems at scale (Groq, Amazon)

I believe the future of software development is human-AI collaboration with appropriate levels of rigor. SAGE is my attempt to make that practical.

Start simple. Add formality when you need it. Let SAGE grow with your project.

🌿
