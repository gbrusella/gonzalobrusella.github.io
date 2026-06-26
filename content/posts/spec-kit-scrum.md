---
title: "Spec-Kit in a Scrum team"
date: 2026-06-12
draft: false
weight: 3
description: "How Spec-Kit's pipeline maps onto backlog refinement, sprint planning, execution, and definition of done."
tags: ["spec-driven-development", "spec-kit", "scrum", "agile"]
series: "Spec-Driven Development"
---

Scrum tells you *when* and *how* to deliver. It doesn't say much about how to translate a user story into something an AI agent can implement without guessing. That's the gap Spec-Kit fills.

The four-phase pipeline maps cleanly onto the Scrum lifecycle. Here's where each phase belongs.

---

## Backlog refinement

This is where Spec-Kit earns its place on a team.

A story comes in from the product owner as a rough description. During refinement, the team runs it through the spec cycle:

```
/speckit.specify As a user, I want to reset my password via email
/speckit.clarify The reset link should expire after 15 minutes. Only one active reset token per user at a time.
/speckit.checklist
```

The `checklist` command validates the spec for completeness and flags ambiguity. Think of it as a pre-flight check before the story gets accepted into a sprint. Ambiguity that slips through refinement becomes a mid-sprint question — and mid-sprint questions are expensive.

The output `spec.md` becomes the Definition of Ready artifact. A story isn't sprint-ready until it has a reviewed spec. The product owner can read it (it's plain language, no implementation details). QA can derive acceptance criteria from it. The developer has unambiguous intent to implement against.

**Tip for the PO:** The spec is the place to be specific about what success looks like. The AI will faithfully implement exactly what the spec says. Vague spec, vague output.

---

## Sprint planning

Planning opens with the plan phase:

```
/speckit.plan
```

The agent reads the accepted spec and the project constitution, maps requirements to the tech stack, and produces a `plan.md` with architecture decisions and component breakdown. This replaces the informal technical design discussion that usually happens in the first half of sprint planning.

The team reviews `plan.md` rather than designing from scratch in the room. Decisions get documented — which matters when someone asks three sprints later why a particular approach was taken.

```
/speckit.tasks
```

`tasks.md` is the implementation checklist. It becomes the basis for task breakdown and estimation. The conversation shifts from "how long will this take roughly" to "which of these tasks carries the most uncertainty" — a much more useful discussion.

Each feature gets its own git branch (`001-feature-name`). Switching features means switching branches. Context is always explicit.

---

## Sprint execution

```
/speckit.analyze
/speckit.implement
```

`/speckit.analyze` runs before implementation starts — a cross-consistency check across spec, plan, and tasks. Catches misalignment before the agent writes code. Worth the 30 seconds it takes.

During the sprint, developers work against `tasks.md`. The AI marks tasks complete as it goes. Because context lives in files rather than a chat session, the agent can be interrupted and resumed — no context loss between sessions.

For complex features, Spec-Kit recommends phased implementation: core functionality first, then validate, then add secondary behavior. Avoids the failure mode where the agent implements everything in one pass and the result is technically correct but structurally wrong.

The `/speckit.implement` command can be re-invoked on remaining tasks after review. The task list is the ground truth, not the chat history.

---

## Definition of done

A story is done when:

1. Tasks in `tasks.md` are marked complete
2. Implementation matches the `spec.md` acceptance criteria
3. The spec and plan files are committed to the branch

The third point matters. The spec and plan aren't just planning artifacts — they're documentation that lives alongside the code. Future developers (and future AI agents) can understand not just what the code does but what it was supposed to do and why it was built that way.

---

## Team configuration

Spec-Kit supports presets that adapt the workflow for specific methodologies. An agile preset restructures the spec template to use story format, adds mandatory acceptance criteria fields, and enforces test-first task ordering.

```bash
specify preset add agile
```

Worth setting up once at the team level. Keeps every developer's spec output consistent without requiring coordination.

Extensions add further integration points — Jira integration exists as a community extension, which means stories can link directly to their spec files. Not mandatory, but useful if the team already lives in Jira.

---

## Where Spec-Kit changes team dynamics

The biggest shift isn't technical — it's the conversation in refinement. When the spec process surfaces ambiguity, it forces the product owner and developer to resolve it before the sprint starts rather than discovering it on day three.

In practice, mid-sprint scope questions drop significantly. Not because the spec predicts everything, but because writing a spec forces the kind of specificity that exposes the questions that were always there — just unanswered.

That's the actual value of the process: it moves the expensive conversations earlier.
