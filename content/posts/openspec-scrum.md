---
title: "OpenSpec in a Scrum team"
date: 2026-06-18
draft: false
weight: 6
description: "Layering OpenSpec's lightweight proposal workflow into backlog refinement, sprint planning, and code review."
tags: ["spec-driven-development", "openspec", "scrum", "agile"]
series: "Spec-Driven Development"
---

OpenSpec's value in a Scrum context is precision and reviewability. It doesn't try to replace Scrum ceremonies — it makes the artifacts those ceremonies produce better, and makes changes to those artifacts visible in ways that normal development doesn't.

---

## Backlog refinement

A story enters refinement as a description from the product owner. The team runs a proposal:

```
/openspec:proposal As a user I want to reset my password via an email link
```

Within seconds the team has:

- `proposal.md` — what will be built, the user scenarios, the rationale
- `design.md` — technical decisions (token storage, expiry, email delivery)
- `tasks.md` — implementation breakdown
- `specs/password-reset/spec.md` — functional requirements in a reviewable format

The proposal replaces the informal technical discussion that usually fills the back half of refinement. Instead of the team verbally designing a feature in a meeting room, the AI produces a design draft that the team reacts to. The conversation becomes "this design decision is wrong, here's why" rather than "how should we approach this."

Refinement ends with a reviewed `proposal.md` committed to the branch. That's the Definition of Ready artifact. The product owner can read it. QA derives acceptance tests from `specs/password-reset/spec.md`. The developer has a task list ready for sprint planning.

**What to watch:** OpenSpec is lightweight, which means it won't enforce completeness the way Spec-Kit's checklist command does. Designate someone in refinement — usually the tech lead or senior developer — to read the proposal critically before accepting it. What edge cases did the AI miss? What constraint from the product owner wasn't captured?

---

## Sprint planning

Sprint planning in an OpenSpec workflow is faster than traditional because the technical design is already done.

The team reviews `design.md` to verify the architectural approach. If something's wrong, they refine:

```
/openspec:proposal Revise password reset to use one-time tokens stored in Redis with 15-minute TTL
```

The proposal updates. The team estimates against `tasks.md` — a concrete list rather than an abstract story.

Because specs live in the repository, the agent knows what already exists. When a new feature touches existing specs, the proposal shows the delta automatically. Sprint planning becomes the moment where the team reviews what will change — in requirements — before committing to implementing it.

---

## Sprint execution

OpenSpec doesn't have an implement command. During the sprint, developers use their agent of choice with the `proposal.md` and `tasks.md` as context. Claude Code, Copilot, Cursor — any of them can pick up the task list and work against it.

The spec file lives at `openspec/specs/password-reset/spec.md` and gets updated when requirements change. If a developer discovers mid-sprint that the proposed approach won't work, they update the proposal first, get it reviewed, then update the implementation. The spec stays aligned with reality.

For the PR, the developer includes:
- The code diff
- The spec delta (if existing behavior changed)
- A link to `proposal.md`

Reviewers see intent and implementation together. That's a meaningful improvement over the normal code review where the reviewer has to infer why from what.

---

## Definition of done

A story is done when:

1. Tasks in `tasks.md` are complete
2. Code passes review
3. `openspec/specs/` reflects the current state of the feature
4. Any spec deltas are reviewed and approved alongside the code

The fourth point is the one teams tend to skip at first. If the spec delta isn't reviewed, you get drift — the specs stop describing what the system actually does and become historical artifacts. The delta review is what keeps them alive.

---

## Collaboration model

OpenSpec's team model is git-first. Specs are files. They go through PRs like everything else. No separate tool for the product owner, no integration to maintain.

For a Scrum team already working in git, this is low friction. The spec library is just another directory in the repository. Anyone with repository access can read the specs. Anyone with write access can update them.

For larger teams or multi-repo systems, OpenSpec Workspaces (currently in development) will add deeper collaboration features. Until then, the git model handles most team scenarios well.

---

## The honest tradeoff

OpenSpec is the easiest of the three frameworks to layer into an existing Scrum team because it requires the least process change. A team can adopt it one feature at a time without disrupting the rest of their workflow.

The cost is that it provides less structure than Spec-Kit and less orchestration than BMad. There's no built-in completeness check, no constitutional constraints, no multi-phase quality gates. What you get is a fast, git-native way to produce and track requirements alongside code.

For teams that want to start somewhere without overhauling their process, OpenSpec is the right entry point.
