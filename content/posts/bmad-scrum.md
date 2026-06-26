---
title: "BMad in a Scrum team"
date: 2026-06-24
draft: false
weight: 9
description: "Mapping BMad's PM, Architect, Developer, and QA agents onto Scrum ceremonies — and where the overhead pays off."
tags: ["spec-driven-development", "bmad", "scrum", "agile", "multi-agent"]
series: "Spec-Driven Development"
---

BMad's multi-agent model maps to Scrum at the role level, not the artifact level. The PM agent does what a product owner should do but often doesn't — rigorous requirement elicitation before sprint commitment. The Architect agent does what a tech lead does when there's time, which in most Scrum teams isn't often enough.

The result is that BMad doesn't just fit into Scrum — it fills the gaps Scrum has always had.

---

## Backlog refinement

Refinement is where BMad's value is most immediate.

A story comes in as a rough description. Before the team discusses it, the PM agent runs:

```
> Use the PM agent. Refine this story into a product brief:
> "As a user, I want to reset my password"
```

The PM agent produces a structured brief: user scenarios, success criteria, explicit out-of-scope items, open questions. It asks the things that refinement sessions skip because they're uncomfortable — "what happens if the user's email account is compromised?" "do we support SSO users?" "what's the fallback if the email bounces?"

The brief goes back to the product owner for review. In practice, product owners often find the PM agent surfaces requirements they hadn't articulated yet. That's not a criticism — it's the value of having something ask systematically.

After the brief is approved, the Architect agent produces a technical design:

```
> Use the Architect agent. Design the implementation for this brief: [paste brief]
```

Refinement ends with both a product brief and a technical design reviewed and committed. A story isn't sprint-ready without both. This is stricter than most Scrum teams' Definition of Ready, and that's the point.

---

## Sprint planning

Sprint planning with BMad is the fastest of the three frameworks because the most expensive conversations — what are we building and how — already happened in refinement.

Planning focuses on the Developer agent's task breakdown:

```
> Use the Developer agent. Break down the implementation tasks for this design: [paste architecture sections]
```

The output is an explicit, ordered task list. The team estimates against concrete tasks rather than abstract stories. The conversation during planning is "which task carries the most risk" rather than "what does 'done' mean for this story."

The adversarial review agent adds a useful ritual at the end of planning: challenge the sprint commitment before it's locked.

```
> Use the adversarial review agent. What could go wrong with this sprint plan? [paste task list and design]
```

It sounds pessimistic. It catches the dependency that would have blocked the sprint on day four.

---

## Sprint execution

During the sprint, the Developer agent works against the task list from planning. Because the design was produced in refinement and validated before the sprint started, the agent has real context — not just a story description but an architecture document and an explicit task list.

Mid-sprint discoveries still happen. When a developer finds that the design doesn't account for something real, the correct response in BMad is to go back up the chain: Architect agent revises the design, adversarial review challenges the revision, Developer agent continues with the updated plan. This feels slower in the moment but prevents the more common failure mode: a developer implementing a workaround that makes sense locally but creates problems system-wide.

The QA agent runs during the sprint, not just at the end:

```
> Use the QA agent. Review this implementation against the brief acceptance criteria: [paste implementation + brief]
```

Running QA incrementally rather than at sprint close surfaces gaps while the implementation is still fresh, not during sprint review when "we need to fix this" means carrying work into the next sprint.

---

## Definition of done

A story is done when:

1. All Developer agent tasks are marked complete
2. QA agent review passes against the brief acceptance criteria
3. The product brief and architecture document are committed to version control
4. Adversarial review found no unresolved issues

The documentation requirement isn't bureaucracy. The product brief and architecture document are the context that the next BMad session on this feature area will use. If they're not committed, the next developer — or the next agent — starts without the decisions that were made.

---

## Ceremony overhead

BMad adds time to refinement and sprint planning. That's the honest tradeoff.

For a mature Scrum team doing sprint planning in 2 hours for a 2-week sprint, adding PM agent + Architect agent + adversarial review to refinement is probably another 30–45 minutes per story, depending on complexity. That's real overhead.

What teams that have adopted BMad consistently report is that sprint execution becomes faster and sprint reviews become cleaner. Stories that used to generate 3–4 mid-sprint scope questions come in with those questions already answered. Bugs that used to appear in sprint review were caught by QA agent mid-sprint.

The overhead moves from execution to planning. Whether that's a good trade depends on your team. If mid-sprint blockers and sprint-review defects are pain points, it usually is.

---

## BMad for Scrum of Scrums

One area where BMad genuinely has no equivalent in the other frameworks: multi-team coordination.

Party Mode — where all agents engage simultaneously — can simulate a cross-team architectural review. Run the Architect agents for each team's domain against a shared integration design. The disagreements between agents often map to the disagreements between teams.

It's not a substitute for actual cross-team collaboration. But as preparation for a cross-team architecture review, running BMad Party Mode first produces a cleaner starting point — the easy conflicts get resolved before the humans get in the room.
