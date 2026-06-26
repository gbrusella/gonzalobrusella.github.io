---
title: "Building with Spec-Kit: from blank repo to shipping features"
date: 2026-06-10
draft: false
weight: 2
description: "A step-by-step walkthrough of building greenfield apps and adding features to existing codebases using Spec-Kit's four-phase pipeline."
tags: ["spec-driven-development", "spec-kit", "ai", "claude-code"]
series: "Spec-Driven Development"
---

Spec-Kit's core bet is that the gap between what you intend to build and what the AI actually builds is a specification problem. Fix the spec, fix the gap. The four-phase pipeline — Specify → Plan → Tasks → Implement — is how it closes that gap systematically.

Here's what that looks like in practice, both when you're starting fresh and when you're adding to something that already exists.

---

## Starting from zero

### Step 1: Install and initialize

```bash
pip install specify-cli
specify init .
```

The CLI prompts you to select your AI coding agent (Claude Code, Copilot, Cursor, Codex, and 30+ others). It scaffolds two directories:

- `.github/` — prompt files your agent picks up as slash commands
- `.specify/` — templates, scripts, and memory

### Step 2: Write the constitution

This is the one step that has no equivalent in traditional development. Before any feature gets specified, you define the ground rules for everything that follows.

```
/speckit.constitution
```

Tell it your tech stack, testing standards, architectural constraints, naming conventions — anything that should be true of every line of code the AI writes for this project. This goes into `.specify/memory/constitution.md` and gets loaded as context on every subsequent command.

Spend real time here. A weak constitution produces consistent but wrong output. A strong one means the AI makes the same architectural decisions you would, every time.

### Step 3: Specify the first feature

```
/speckit.specify Build a user authentication system with email/password login and JWT sessions
```

The agent generates a `spec.md` in a new feature directory (`001-user-auth/`). The spec covers what the feature does, the user scenarios, and the acceptance criteria — no implementation details yet. Tech-agnostic by design.

Run `/speckit.clarify` to refine, `/speckit.checklist` to validate completeness before moving on.

### Step 4: Plan

```
/speckit.plan
```

Now the tech stack enters the picture. The agent reads the spec and the constitution, maps requirements to your specific stack, and produces a `plan.md` with architecture decisions, component breakdown, and the rationale for each choice.

Review this carefully. The plan is where the AI makes decisions you'd otherwise make in a design meeting. If something looks wrong, fix it here — not after the tasks are generated.

### Step 5: Tasks and implementation

```
/speckit.tasks
/speckit.analyze
/speckit.implement
```

`tasks.md` is the implementation checklist. `/speckit.analyze` cross-checks consistency across spec, plan, and tasks before a single line of code gets written. Then `/speckit.implement` executes against the checklist.

The agent marks tasks complete as it goes. You can stop and resume — context lives in the files, not the chat window.

---

## Adding a feature to an existing codebase

This is where the spec history starts paying off.

Each previous feature left behind a `spec.md` and a `plan.md`. When you specify a new feature that touches the same area, the agent has documented context about what was already decided and why.

```
/speckit.specify Add "remember me" checkbox with 30-day session extension
```

The agent reads existing specs in related directories, factors in the constitution, and produces a spec that fits the existing system rather than treating it as a greenfield problem.

The `/speckit.analyze` step is especially important for feature work. It checks the new spec and plan against existing artifacts for conflicts before implementation. Catches "this contradicts how sessions work in 001-user-auth" at the requirements level, not during code review.

### What to watch for

The constitution doesn't automatically know about patterns that emerged organically during development — things you decided mid-sprint that never got documented. Before specifying a new feature, add anything material to the constitution. Keeps the AI's architectural decisions aligned with where the codebase actually ended up.

### Updating an existing spec

When a feature needs to change existing behavior:

```
/speckit.specify Modify session expiration to support configurable timeouts per user role
```

Spec-Kit generates a new spec in a new feature directory. The old spec stays unchanged — it documents what was originally built. The new spec documents the change and its rationale. You end up with a complete decision trail in version control.

---

## The rhythm

After the first few features, Spec-Kit becomes a habit more than a process. A feature starts with a spec command, runs through the pipeline, and lands as code with full documentation baked in.

The payoff compounds. Six months in, when someone asks why the session timeout is 24 hours by default, the answer is in `001-user-auth/plan.md`. When a new developer joins, the spec history is the codebase tour.

The work you do upfront to write a good spec isn't overhead. It's the thing that makes every AI interaction after it deterministic instead of a guess.
