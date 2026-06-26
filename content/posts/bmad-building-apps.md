---
title: "Building with BMad: directing a team of agents"
date: 2026-06-22
draft: false
weight: 8
description: "How BMad's multi-agent model handles product briefs, architecture, adversarial review, and implementation from scratch and in existing codebases."
tags: ["spec-driven-development", "bmad", "ai", "multi-agent", "claude-code"]
series: "Spec-Driven Development"
---

BMad treats software development as a team sport — even when you're the only human on the project. You're the product director. The agents are your specialists. Your job is to describe the outcome; their job is to figure out how to build it.

That's a genuinely different way to work with AI, and it shows most clearly when you're starting something new.

---

## Starting from zero

### Step 1: Install

BMad works with any AI coding assistant that supports custom system prompts. Claude Code is the recommended path:

```bash
# Install via Claude Code
claude
> /bmad-install
```

This scaffolds the BMad agent configurations into your project. Each agent — PM, Architect, Developer, QA — gets a system prompt that defines its role, responsibilities, and handoff protocol.

### Step 2: Pressure-test the idea

Before any specification work starts, BMad has an optional but valuable step: running the PM agent to refine and challenge the product concept.

```
> Use the PM agent. I want to build a team productivity platform with Kanban boards, task assignment, and time tracking.
```

The PM agent doesn't take your description at face value. It asks the questions a product manager would ask: Who are the primary users? What's the core workflow? What does success look like at 6 months? What are you explicitly not building?

The output is a structured product brief — not a technical document, a product document. What problem it solves, who it's for, what it must do, and what's out of scope. This brief drives everything that follows.

BMad calls this the Analysis Phase, and it's genuinely the part most development processes skip. Scoping ambiguity that gets resolved in the brief doesn't surface as a mid-sprint disagreement.

### Step 3: Architecture

The Architect agent picks up the brief:

```
> Use the Architect agent. Here is the product brief: [paste brief]
```

The Architect designs the system: component breakdown, data model, API contracts, infrastructure choices, integration points. The output is a structured architecture document with decisions and rationale — not pseudocode, not implementation, just design.

BMad's adversarial review can run here: a separate agent session specifically tasked with finding flaws in the architecture before anyone starts coding.

```
> Use the adversarial review agent. Challenge this architecture: [paste architecture doc]
```

The adversarial agent looks for scalability issues, missing failure modes, over-engineering, incorrect assumptions. The Architect responds. The exchange continues until the architecture is solid.

### Step 4: Development

The Developer agent picks up the architecture document and implements it. Because the architecture was explicitly designed and reviewed, the Developer has real context — not just a feature description but a complete technical blueprint.

```
> Use the Developer agent. Implement the authentication module per the architecture document. [paste relevant sections]
```

For complex implementations, BMad's phased approach applies: implement core structure first, validate it, then add behavior incrementally.

### Step 5: Verification

QA agent reviews the implementation against the brief and architecture:

```
> Use the QA agent. Verify the authentication module against these acceptance criteria: [paste from brief]
```

The QA agent produces a test plan and flags gaps between the spec and the implementation. Gaps go back to the Developer agent, not forward to production.

---

## Adding a feature to an existing codebase

BMad's established projects workflow adds a context phase before the normal agent sequence.

### Forensic investigation

If you're onboarding to an existing codebase or the feature touches a system that wasn't built with BMad, start with forensic investigation:

```
> Use the forensic investigation agent. Analyze the authentication module and produce a system understanding document.
```

The forensic agent examines code, tests, comments, and whatever documentation exists to reconstruct the intent behind the system. The output is a "system understanding document" that gives the Architect and Developer accurate context about what already exists.

This step is expensive — it takes time and tokens — but it's significantly cheaper than the Architect designing a feature that conflicts with the existing system's assumptions.

### Feature addition sequence

With the existing system documented:

1. **PM agent** — scopes the new feature relative to what already exists. Defines how it interacts with current behavior.
2. **Architect agent** — designs the addition, explicitly referencing the existing architecture. Documents integration points and migration requirements.
3. **Adversarial review** — challenges the addition specifically for conflicts with existing behavior.
4. **Developer agent** — implements against the reviewed design.
5. **QA agent** — verifies the new feature without breaking existing behavior.

The extra steps feel heavyweight until you compare them to the alternative: a developer implementing a feature that conflicts with the existing system's assumptions and discovering the conflict in QA.

---

## Party mode

BMad has a feature called Party Mode for complex decisions: all agents engage simultaneously with a problem, producing independent analyses. You read the divergences and make a call.

It's most useful when the product brief has a genuinely hard tradeoff — build now vs. build right, microservices vs. monolith, third-party vs. internal. Instead of one agent's perspective, you get three or four, and the disagreements surface the actual considerations.

Not every decision needs Party Mode. But for the ones that do, it's more useful than asking one agent the same question multiple ways.

---

## The honest overhead

BMad is the heaviest of the three frameworks. The agent handoffs require intentional orchestration — you're directing the team, not running a command. The payoff is proportional to the complexity of what you're building.

For a simple CRUD feature in a well-understood system, BMad is overkill. For a new product, a significant platform change, or a feature that touches multiple existing systems, the structured agent sequence catches things that a lighter process misses.

The question isn't whether BMad is good. It is. The question is whether your current feature justifies the overhead. Most teams that adopt BMad end up using it selectively — full sequence for significant features, lighter tools for routine work.
