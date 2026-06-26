---
title: "Building with OpenSpec: proposal-first development"
date: 2026-06-16
draft: false
weight: 5
description: "How OpenSpec's proposal workflow and spec delta model handle both greenfield features and changes to existing codebases."
tags: ["spec-driven-development", "openspec", "ai", "brownfield"]
series: "Spec-Driven Development"
---

OpenSpec's philosophy is that most spec-driven tools assume you're starting fresh. OpenSpec assumes you're not. It's built for the more common scenario: a codebase that already exists, features that need to fit into it, and a team that doesn't want a heavyweight process getting in the way.

The workflow is lighter than Spec-Kit — fewer phases, less scaffolding — but the key innovation is the spec delta: a requirements diff that shows exactly how a change modifies what the system is supposed to do.

---

## Starting from zero

### Step 1: Install

```bash
npm install -g @fission-ai/openspec@latest
```

OpenSpec exposes slash commands in Claude Code, Cursor, Copilot, Codex, and most major agents. No API keys, no external dependencies. Everything lives in your repository.

### Step 2: First proposal

There's no constitution step, no upfront scaffolding. You describe the first feature you want to build:

```
/openspec:proposal Build a user authentication system with email/password login and JWT sessions
```

OpenSpec searches your (currently empty) codebase, then generates four artifacts in `openspec/changes/user-auth/`:

```
openspec/changes/user-auth/
├── proposal.md      ← what will be built and why
├── design.md        ← technical decisions
├── tasks.md         ← implementation breakdown
└── specs/
    └── auth-login/
        └── spec.md  ← functional requirements
```

The spec gets committed to `openspec/specs/auth-login/spec.md` — its permanent home. Future features that touch authentication will find it there.

### Step 3: Review and refine

The proposal is the artifact you review before any code gets written. Read `proposal.md` and `design.md`. If something looks wrong, refine:

```
/openspec:proposal Revise to use refresh tokens alongside JWT access tokens, 15-minute access token expiry
```

OpenSpec regenerates the proposal with your constraint applied. Iterate until the plan reflects what you actually want to build.

### Step 4: Implement

Once the proposal is approved, hand it to your agent for implementation. The `tasks.md` is the checklist. OpenSpec doesn't have a dedicated implement command — it's designed to work with whatever agent workflow you already use.

Specs accumulate as you build. Each new feature adds to `openspec/specs/`, organized by capability:

```
openspec/specs/
├── auth-login/
│   └── spec.md
├── auth-session/
│   └── spec.md
└── checkout-cart/
    └── spec.md
```

This library becomes the living documentation for what the system is supposed to do.

---

## Adding a feature to an existing codebase

This is where OpenSpec pulls ahead.

### The spec delta

When you run a proposal against a codebase that already has specs, OpenSpec reads them before generating anything. The proposal shows you not just what the new feature will do — it shows you how it changes existing requirements.

```
/openspec:proposal Add "remember me" checkbox with 30-day session extension
```

OpenSpec reads `openspec/specs/auth-session/spec.md`, identifies the affected requirements, and generates a delta alongside the new proposal:

```diff
openspec/specs/auth-session/spec.md

### Requirement: Session expiration
- The system SHALL expire sessions after 24 hours of inactivity.
+ The system SHALL support configurable session expiration periods.

#### Scenario: Default session timeout
  GIVEN a user has authenticated
- WHEN 24 hours pass without activity
+ WHEN 24 hours pass without "Remember me"
  THEN invalidate the session token

+ #### Scenario: Extended session with remember me
+ GIVEN user checks "Remember me" at login
+ WHEN 30 days have passed
+ THEN invalidate the session token
+ AND clear the persistent cookie
```

That delta goes into the PR alongside the code diff. Reviewers can verify intent before reading a single line of implementation.

### Why this matters

In a traditional code review, a reviewer has to understand the intent of a change from the code itself — which means understanding the implementation to infer the requirement. The spec delta inverts that. The requirement change is explicit. The code change is just its expression.

It's a small shift in process with a meaningful effect on review quality. Intent mismatches — "I thought we were only extending sessions for premium users" — get caught before merge, not in a post-release incident.

### Starting specs for existing code

If you have a mature codebase with no existing specs, OpenSpec doesn't try to generate them retroactively. The recommended approach: create specs for features as you touch them. Work on the checkout flow? Create the checkout spec. Fix a bug in auth? Create the auth spec.

It's slower than having full coverage from day one, but it's more accurate. Generated specs for code you didn't write tend to describe what the code does rather than what it was supposed to do — which are often different things.

---

## The rhythm

OpenSpec's lightweight process means the proposal step adds maybe 10 minutes to the start of feature work. That time comes back during implementation (less rethinking mid-feature) and during review (faster, higher-quality review conversations).

The spec library grows with the codebase rather than being a separate artifact that drifts. When someone joins the team six months in, `openspec/specs/` tells them what the system is supposed to do. Not what it currently does — what it was built to do. That distinction matters more than it sounds.
