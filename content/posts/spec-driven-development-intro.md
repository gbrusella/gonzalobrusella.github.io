---
title: "Three ways to stop winging it with AI"
date: 2026-06-08
draft: false
weight: 1
description: "Spec-Kit, OpenSpec, and BMad compared — an introduction to spec-driven development with AI coding agents."
tags: ["spec-driven-development", "ai", "spec-kit", "openspec", "bmad"]
series: "Spec-Driven Development"
---

Everyone I talk to has the same story. They gave an instruction to Copilot or Claude, got code back, ran it, it half-worked, they asked again, it drifted in a different direction, and thirty minutes later they had a pile of files that almost did the thing they wanted.

That's not an AI problem. That's a process problem.

Spec-Driven Development — SDD — is the answer the industry landed on: write down *what* you want before the AI writes *how* to build it. Simple idea. The interesting part is the three frameworks that have emerged to do it, each with a different take on what "writing it down" should look like.

---

## The common ground

All three frameworks share one core belief: the specification is the source of truth, not the code. Code is just the current expression of the spec. When something breaks, you fix the spec. When requirements change, you update the spec. The AI handles the translation.

That's a real shift in thinking — especially for developers who've spent years treating docs as an afterthought.

---

## Spec-Kit

GitHub built Spec-Kit and released it open source in late 2025. It's the most structured of the three — and the most popular, with over 100,000 GitHub stars in its first months.

The workflow is a four-step pipeline: **Specify → Plan → Tasks → Implement**. Each step produces a Markdown file that feeds the next. You write what you want to build (the spec). The AI turns that into a technical blueprint (the plan). The plan breaks down into a checklist (tasks). Then the AI codes against the checklist.

Before any of that starts, you write a `constitution.md` — a one-time document that captures the ground rules for the project: coding standards, testing requirements, architectural constraints. It's the thing that keeps every future feature consistent with the decisions you made on day one.

Spec-Kit works with 30+ AI coding agents, integrates through slash commands (`/speckit.specify`, `/speckit.plan`, etc.), and has a growing ecosystem of community extensions — Jira integration, architecture review gates, regulatory traceability templates.

**Best for:** Teams that want a complete, opinionated workflow they can customize incrementally. Strong on both new projects and legacy modernization.

---

## OpenSpec

OpenSpec, from Fission AI, is lighter on its feet. Its tagline calls it "brownfield-first" — which is a technical way of saying it was designed for codebases that already exist, not blank slates.

The key concept is the **spec delta**. When you plan a change, OpenSpec generates a diff of the requirements themselves — what the system *was* supposed to do vs. what it will do after your change. That diff becomes the thing you review, not just the code diff. It's a small idea with big implications for code reviews: reviewers can understand intent without reading implementation.

The workflow kicks off with a proposal command: describe the change, and OpenSpec generates a `proposal.md`, `design.md`, `tasks.md`, and the spec deltas all at once. You review the plan before any code gets written.

No API keys, no MCP server, no external dependencies. Specs live in your repo alongside your code, organized by capability folder. OpenSpec installs via npm and exposes slash commands in Claude Code, Cursor, Copilot, and most other agents.

**Best for:** Developers and teams working in established codebases who want minimal process overhead and want spec changes to be reviewable through their normal git workflow.

---

## BMad

BMad — Build More Architect Dreams — is the most ambitious of the three. Where Spec-Kit and OpenSpec focus on the *writing-a-spec* part of the process, BMad covers the whole product lifecycle, from idea to shipped feature.

The framework deploys a team of specialized AI agents: a Product Manager agent that refines your idea, an Architect agent that designs the system, a Developer agent that writes the code. You direct the team; they execute. BMad calls this "named agents," and it's genuinely a different way to work — closer to managing a sprint than typing into a chat window.

BMad also ships with things Spec-Kit and OpenSpec don't: adversarial review (an agent that tries to break your plan), brainstorming workflows, forensic investigation tools for debugging at the requirements level, and a module marketplace for vertical extensions (game dev, enterprise testing, creative writing).

Installation is through any AI coding assistant that supports custom system prompts — Claude Code is the recommended path. v6, released in 2025, introduced a skills architecture that makes the whole system more composable.

**Best for:** Teams building complex products from scratch who want AI involved at every stage, not just implementation.

---

## How they compare

| | Spec-Kit | OpenSpec | BMad |
|---|---|---|---|
| Origin | GitHub | Fission AI | Community (Brian Adkins) |
| Weight | Medium | Light | Heavy |
| Brownfield support | Good | First-class | Good |
| New project support | First-class | Good | First-class |
| Agent model | Single | Single | Multi-agent team |
| Ecosystem | Extensions, presets, bundles | Growing | Modules, marketplace |
| External dependencies | None | None | AI coding assistant |

---

## Which one?

Depends on where you are.

If you're starting a new project and want structure without a steep learning curve, Spec-Kit is the obvious pick. The four-phase pipeline is easy to explain to a team, and the GitHub ecosystem backing means integrations will keep appearing.

If you're working on an existing system and care about making spec changes visible in code reviews, OpenSpec's delta approach is worth the small setup cost.

If you're building something complex from scratch and want AI involved in planning, not just coding — BMad. The multi-agent model takes getting used to, but the output quality on complex decisions is noticeably better.

The good news is they're all lightweight enough to try. None of them require you to throw away your current tools. Pick one, run a single feature through it, and see if the spec-before-code discipline changes the output you get from your AI.

For most teams, it does.
