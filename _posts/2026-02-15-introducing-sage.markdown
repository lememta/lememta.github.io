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
- Store data in the right database?

The quality of AI-generated code depends entirely on the quality of your specification. And here's the rub: **traditional programming languages were designed for writing implementations, not specifications**.

Meanwhile, formal specification languages like TLA+, Alloy, and Z are incredibly powerful for verification but:

- Have steep learning curves
- Don't mix well with natural language
- Generate verbose specifications
- Are disconnected from implementation
- Aren't optimized for LLM consumption

We needed something in between. Something that's:

- **Accessible** to software engineers (not just formal methods experts)
- **Flexible** enough to start simple and add rigor incrementally
- **Token-efficient** for LLM processing
- **Expressive** enough to capture both intent and implementation

That's why we built SAGE.

-----

## What is SAGE?

SAGE (Semi-formal Agent Guidance & Engineering) is a specification language with a radical idea at its core: **mix natural language and formal notation in the same file, and make formality optional**.

Here's a simple SAGE function:

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

Notice what's happening here:

- Natural language explains *what* and *why*
- Type annotations provide safety
- The `!!` marker highlights critical security operations
- Code and commentary flow together naturally

-----

## The Three Levels of SAGE

One of SAGE's core innovations is that it works at **three levels of formality**. You choose the level based on your needs:

### Level 0: Pure Natural Language

Perfect for initial design and requirements gathering:

```
"Build a user authentication system"
"Users can register with email and password"
"Passwords must be hashed with bcrypt"
"Sessions expire after 24 hours"
"Rate limit login attempts to 5 per minute"

!! "Use PostgreSQL for storage"
!! "Use Redis for rate limiting"
```

Hand this to an LLM, and it can generate a complete implementation. The `!!` markers tell the AI which decisions are non-negotiable.

### Level 1: Structured Specifications

This is where you'll spend 90% of your time. It's like modern TypeScript or Rust, but with native support for natural language:

```
@mod auth_service
@use std.crypto
@use db.postgres as pg

@type User = { id: Int, email: Str, password_hash: Str }

@fn register(email: Str, password: Str) -> Result<User>
@req email.contains("@")
@req password.len() >= 8

"Check if user already exists"
let existing = pg.query("SELECT * FROM users WHERE email = $1", [email])
if !existing.is_empty() => ret Err("Email already in use")

"Hash password with bcrypt"
!!let hash = crypto.bcrypt_hash(password, cost: 12)

let user = {id: pg.next_id(), email: email, password_hash: hash}
pg.execute("INSERT INTO users VALUES ($1, $2, $3)", 
           [user.id, user.email, user.password_hash])
ret Ok(user)
```

You get:

- Type safety
- Preconditions and postconditions
- Natural language documentation inline
- Clear error handling
- Explicit design decisions

### Level 2: Formal Refinement

For mission-critical systems where you need to track design evolution:

```
@spec UserAuth
"High-level specification of authentication"

@state
  users: Set<User>
  sessions: Set<Session>

@invariant "No duplicate emails"
  ∀ u1,u2 ∈ users: u1.email ≠ u2.email

@ops register(email: Str, password: Str) -> Result<User>
@req email ∉ {u.email | u ∈ users}
@ens ∃u ∈ users': u.email = email

@refine UserAuth as SecureUserAuth
"Add concrete password security"

@decisions
  !! "Use bcrypt with cost factor 12"
  "Store password hashes, never plaintext"
  "Sessions are 32-byte random tokens"

@state
  users: Set<{id: Int, email: Str, password_hash: Str}>

@maps "Concrete user maps to abstract user via email"
  AbstractUser(u) = {email: u.email}

@preserves
  ✓ "No duplicate emails invariant still holds"
  "Proof: email is still unique in concrete representation"
```

Now you have:

- Mathematical precision where it matters
- Tracked design decisions
- Refinement proofs
- Alternative design paths
- Full traceability

-----

## Why Refinement Matters (Even More in the AI Era)

Refinement is one of the most elegant ideas in computer science. You start with a high-level specification of *what* the system should do, then incrementally refine it to *how* it does it, proving at each step that you've preserved the original properties.

In traditional software development, refinement was often seen as academic—too heavyweight for practical use. But in the age of AI-generated code, refinement becomes **essential**:

1. **Design decisions are now prompts**: "Use bcrypt with cost 12" is a prompt instruction
2. **Multiple implementations exist**: AI can generate alternatives; refinement tracks which to choose
3. **Verification is critical**: When AI generates code, how do you know it's correct?
4. **Evolution is constant**: Requirements change; refinement tracks the evolution

Here's the beautiful part: **In SAGE, refinement is optional**. Start with Level 1 code. If your system becomes critical, add formal specs and refinements. If requirements change, create an alternative refinement path:

```
@refine UserAuth as JWTAuth [alternative]
"Alternative: JWT tokens instead of server sessions"

@decisions
  "Stateless JWT tokens"
  "No server-side session storage"

@compare_with SecureUserAuth
  advantages: "Easier horizontal scaling"
              "No database lookups for validation"
  disadvantages: "Cannot revoke before expiry"
                 "Need refresh token strategy"
```

Now you have a formal record of design alternatives with their tradeoffs. Give this to an LLM and say "implement the JWT version" or "compare these approaches."

-----

## Token Efficiency: Designed for LLMs

SAGE is deliberately compact. Here's a comparison:

**Traditional Java + Comments (850 tokens):**

```java
/**
 * Authenticates a user with email and password.
 * Returns a session token if successful.
 * @param email The user's email address
 * @param password The user's password
 * @return Result containing session or error
 */
public Result<Session> authenticate(String email, String password) {
    // First, look up the user by email
    Optional<User> userOpt = database.findUserByEmail(email);
    if (userOpt.isEmpty()) {
        return Result.error("Invalid credentials");
    }
    // ... rest of implementation
}
```

**SAGE (380 tokens - 55% reduction):**

```
@fn authenticate(email: Str, password: Str) -> Result<Session>

"Look up user by email"
let user = db.find_user(email)
if user.is_none() => ret Err("Invalid credentials")

# ... rest of implementation
```

This isn't just about saving tokens (though that's nice for API costs). It's about **information density**. Every token in SAGE carries meaning. There's less boilerplate, less ceremony, more signal.

LLMs trained on SAGE specifications can:

- Generate code more accurately
- Understand intent more clearly
- Maintain consistency better
- Produce less hallucinated code

-----

## Real-World Example: From Spec to Implementation

Let's walk through a complete example. Suppose you want to build a payment processing system.

**Week 1: Initial Design (Level 0)**

```
"Build a payment processing system"
"Support credit card and bank transfer payments"
"Store transaction history"
"Handle refunds and chargebacks"
"Comply with PCI DSS requirements"

!! "All payment data must be encrypted at rest"
!! "Use Stripe API for credit card processing"
!! "Transaction IDs must be globally unique"
```

You give this to an LLM. It generates a basic implementation. Good enough for a prototype.

**Week 4: Adding Structure (Level 1)**

Now you need to ship to production. You add types and contracts:

```
@mod payment_processor
@use stripe.api
@use db.postgres as pg

@type Payment = {
  id: Str,
  amount: Float,
  currency: Str,
  method: PaymentMethod,
  status: PaymentStatus
}

@type PaymentMethod = CreditCard | BankTransfer
@type PaymentStatus = Pending | Completed | Failed | Refunded

@fn process_payment(amount: Float, method: PaymentMethod) -> Result<Payment>
@req amount > 0.0
@req "Amount must not exceed daily limit" && amount <= 10000.0
@ens "Payment is recorded in database"

"Generate unique transaction ID"
!!let tx_id = crypto.uuid_v4()

"Process with appropriate method"
let result = match method {
  CreditCard => stripe.charge(amount, tx_id),
  BankTransfer => bank.transfer(amount, tx_id)
}

if result.is_err() =>
  "Log failure and return error"
  log.error("Payment failed", tx_id, result.error)
  ret Err(result.error)

"Record successful payment"
let payment = {
  id: tx_id,
  amount: amount,
  currency: "USD",
  method: method,
  status: Completed
}
pg.execute("INSERT INTO payments ...", [payment.id, payment.amount])
ret Ok(payment)
```

**Month 3: Going Enterprise (Level 2)**

Your startup is growing. You need to support multiple payment providers and ensure correctness:

```
@spec PaymentProcessor
"High-level payment processing specification"

@state
  payments: Set<Payment>
  accounts: Map<AccountId, Balance>

@invariant "Money is conserved"
  ∑ accounts.values() = TOTAL_MONEY

@invariant "No duplicate transaction IDs"
  ∀ p1,p2 ∈ payments: p1.id = p2.id ⟹ p1 = p2

@ops process_payment(amount: Float) -> Result<Payment>
@req amount > 0
@ens |payments'| = |payments| + 1

@refine PaymentProcessor as StripePayments
"Implementation using Stripe API"

@decisions
  !! "Use Stripe for all credit card processing"
  "Implement idempotency using transaction IDs"
  "Store payment records in PostgreSQL"
  "Use optimistic locking for account balances"

@preserves
  ✓ "Money conservation: tracked via Stripe webhooks"
  ✓ "No duplicates: enforced by unique transaction ID"

@refine PaymentProcessor as MultiProviderPayments [alternative]
"Alternative: Support multiple payment providers"

@decisions
  "Route to Stripe or PayPal based on method"
  "Implement provider failover"

@compare_with StripePayments
  advantages: "Redundancy if Stripe has issues"
              "Can negotiate better rates"
  disadvantages: "More complex integration"
                 "Need to handle different APIs"
```

Now you have:

- Formal correctness properties
- Multiple implementation strategies
- Clear rationale for each design decision
- Traceability from requirements to code

-----

## The Philosophy: Formality is a Dial, Not a Switch

Traditional approaches to formal methods treat formality as binary: either you're doing formal verification or you're not. SAGE treats formality as a **spectrum**:

```
Natural Language ←──────────────→ Full Formal Proof
     Level 0           Level 1           Level 2
   (Prototype)      (Production)   (Mission-Critical)
```

You can:

- Start informal and add rigor incrementally
- Use different levels for different parts of the system
- Dial up formality for critical components
- Keep it simple for straightforward code

This is how engineers actually think. We don't need formal proofs for a "hello world" function, but we do for payment processing.

-----

## Comparison with Other Approaches

### vs. Traditional Programming Languages (Java, Python, TypeScript)

**Traditional:**
- ✅ Well-established ecosystem
- ✅ Rich libraries and tooling
- ❌ Not designed for specification
- ❌ Comments separate from code
- ❌ Verbose syntax
- ❌ No refinement concept

**SAGE:**
- ✅ Specification-first design
- ✅ Natural language integrated
- ✅ Token-efficient
- ✅ Refinement tracking
- ⚠️ New ecosystem (less mature)

### vs. Formal Specification Languages (TLA+, Alloy, Z)

**Formal Languages:**
- ✅ Mathematical precision
- ✅ Powerful verification
- ✅ Proven correctness
- ❌ Steep learning curve
- ❌ Disconnected from implementation
- ❌ Not LLM-friendly

**SAGE:**
- ✅ Accessible to all engineers
- ✅ Connects spec to implementation
- ✅ Optional formality
- ✅ LLM-optimized
- ⚠️ Less powerful verification (trade-off for accessibility)

### vs. Documentation Formats (Markdown, RST)

**Documentation:**
- ✅ Natural language
- ✅ Easy to write
- ❌ Not executable
- ❌ No type checking
- ❌ Drift from code

**SAGE:**
- ✅ Natural language + types
- ✅ Executable specifications
- ✅ Type-checked
- ✅ Code is the documentation

-----

## Use Cases

### 1. AI-Assisted Development

**Before SAGE:**
- Write detailed prompt
- Review generated code
- Iterate on prompt
- Manually verify correctness

**With SAGE:**

```
@fn authenticate(email: Str, password: Str) -> Result<Session>
@req "Email format must be valid"
@req "Password must meet complexity requirements"
@ens "Session token is cryptographically secure"

!! "Use bcrypt with cost 12 for password hashing"
!! "Rate limit to 5 attempts per IP per minute"

# LLM generates implementation from this spec
```

The LLM has precise requirements, explicit constraints, and clear success criteria.

### 2. System Design Documentation

SAGE specs are executable documentation. Your design docs can be:

- Type-checked
- Compiled
- Tested
- Generated into implementations

No more drift between docs and code.

### 3. Legacy System Understanding

Reverse-engineer a system by writing SAGE specs:

```
@spec LegacyAuthSystem
"Observed behavior of current auth system"

@state
  users: Database<User>     # Actually a MySQL database
  sessions: Cache<Session>  # Redis cache

@invariant "Sessions expire after 30 minutes"
@invariant "Max 3 sessions per user"
```

Then create refinements representing different modernization strategies.

### 4. Compliance and Audit

For regulated industries:

```
@spec GDPR_Compliance
"GDPR requirements for user data"

@invariant "User can request data deletion"
@invariant "Data minimization: only collect necessary data"
@invariant "Consent is explicit and recorded"

@refine GDPR_Compliance as UserService
@decisions
  !! "Implement right to erasure via soft delete"
  !! "Log all consent events with timestamp"

@preserves
  ✓ "Data deletion: verified by test suite"
  ✓ "Minimization: code review confirms"
  ✓ "Consent logging: database constraints enforce"
```

Now your compliance is formalized, tracked, and verifiable.

-----

## Getting Started

Ready to try SAGE? Here's how:

### Install the Parser

```bash
git clone https://github.com/lememta/sage-lang
cd sage-lang
pnpm install
pnpm build
```

### Your First SAGE Program

Create `hello.sage`:

```
@mod hello

@fn greet(name: Str) -> Str
"Create a personalized greeting"
ret "Hello, " + name + "!"
```

### VS Code Extension

```bash
cd packages/vscode
npx @vscode/vsce package --no-dependencies
code --install-extension sage-vscode-0.1.0.vsix
```

### Learn More

- **GitHub**: [github.com/lememta/sage-lang](https://github.com/lememta/sage-lang)
- **Tutorial**: See `docs/tutorial.md` in the repo
- **Examples**: Check the `examples/` folder

-----

## Join the Movement

Software development is changing. AI is becoming our pair programmer, our code reviewer, our architect. But AI is only as good as our specifications.

SAGE is our bet on the future: a world where:

- **Specifications are first-class**: Not an afterthought, but the starting point
- **Natural language is code**: Documentation and implementation are unified
- **Formality is accessible**: You don't need a PhD to write correct software
- **Design decisions are tracked**: Refinement captures the *why*, not just the *what*
- **AI is our collaborator**: LLMs generate from specs, not vague prompts

We're building SAGE in the open. The compiler is open source. The specification is evolving. We need:

- **Engineers** to write SAGE code and give feedback
- **Researchers** to formalize the semantics and build verification tools
- **Tool builders** to create IDE plugins, linters, and generators
- **Early adopters** to use SAGE in production and report what works (and what doesn't)

The age of AI-powered development is here. Let's build the languages we need for it.

-----

**Try SAGE today**: [github.com/lememta/sage-lang](https://github.com/lememta/sage-lang)

-----

*SAGE is open source (MIT License). We believe the future of software development is collaborative—between humans and AI, between formal and informal, between specification and implementation.*
