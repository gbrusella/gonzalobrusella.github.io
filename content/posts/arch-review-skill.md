---
title: "arch-review: an Agent Skill that reviews your application's architecture"
date: 2026-11-27
draft: false
description: "A read-only Agent Skill that reviews a medium-small application from its code and docs, and reports what's risky, what's unverified, and what to fix first."
tags: ["architecture", "agent-skills", "claude-code", "code-review", "adr"]
---

I published [arch-review](https://github.com/gbrusella/arch-review-skill), an Agent Skill that reviews the architecture of a medium-small application. It reads the source, the docs, and the infrastructure and pipeline files that ship with it, then tells you what's risky, what it couldn't verify, and what to fix first.

It reviews one application, not a landscape of systems. That's a deliberate limit. It is not an enterprise architecture tool.

## Read-only, and honest about gaps

The review never changes your project and never prints secret values. It also never treats missing information as a pass. If a requirement is documented or expected and the reviewer can't check it, the finding is marked `unverifiable`. Silence in the repo isn't evidence that something is fine.

Every finding carries an evidence status (`confirmed`, `risk`, `unverifiable` or `assumption`), plus severity, likelihood, confidence and a category. Severity and confidence are separate on purpose: a serious issue the reviewer isn't sure about should look different from a minor one it's certain about.

## What it checks

- **Design decisions.** ADRs against the MADR format, and the code against what the ADRs and design docs claim. When code contradicts a documented decision, you get one finding with both citations.
- **Quality attributes.** Performance and scalability, resilience, maintainability, observability, data architecture, cost, testing, security, supply chain and auditability.
- **Deployment.** What the docs say against what the pipeline actually does: environments, release strategy, rollback, migrations, infrastructure as code.
- **Cloud alignment.** It detects the provider from the repo (Azure, AWS, Google Cloud or Oracle Cloud) and rates the project against that provider's well-architected pillars. This is an evidence-based check, not a formal Well-Architected Review.
- **Scenarios.** It walks a few concrete failures through the design — a dependency down, a bad deploy, lost backups, 10x load — and records what happens.

## What you get back

A report with a verdict (`Not ready`, `Ready with conditions` or `Ready`), the top three actions, a quality-attribute heatmap, strengths, a prioritised findings table, and a remediation plan. Every item in the plan has an owner role, an effort estimate and a verification step. A plan without a way to check it was done isn't much of a plan.

There are four depths: compact for small targets (a single ADR, a feature, around 30 files), full, quick triage, and delta, which checks a pull request against the existing ADRs.

## When to use it

Two situations fit best.

**A brownfield project that needs remediation.** You inherit a codebase, or you own one that has drifted, and you need to know where to start. This is where the report earns its keep. The prioritised findings and the remediation plan (owner role, effort, verification step) turn "this feels fragile" into an ordered list. The code-versus-ADR check catches decisions that were documented once and quietly abandoned. Once the first pass is done, delta mode keeps new pull requests from reopening what you just fixed.

**The foundation of a greenfield project.** Before the codebase grows, review what exists: the ADRs, the design docs, the first scaffolding, the pipeline and infrastructure as code. Fixing a weak decision now costs a conversation, not a migration. At this stage the compact depth on a single ADR or one feature is often enough. It will also tell you what the documents don't answer yet, which is useful on day one: those `unverifiable` findings are your open questions.

Either way, it works best on one medium-small application with some written decisions to check against. With no ADRs or docs, it still reviews the code, but it has less to compare it to.

## What it does not do

It is not the right tool for:

- A landscape of systems or an enterprise architecture. It reviews one application.
- Compliance or certification. It makes no compliance determination and isn't a threat assessment.
- Measuring anything. It reads files and history, and runs no code, scanners or network audits. Vulnerability status and real performance numbers stay with you.
- Checking live state. It can't see your cloud account or repository settings such as branch protection, and it says so in the report instead of guessing.

## Install and run

It's both a plugin and a marketplace, and the skill follows the open Agent Skills format, so it isn't tied to one tool. In Claude Code:

```
/plugin marketplace add gbrusella/arch-review-skill
/plugin install arch-review@arch-review-skill
```

There are manifests for Copilot CLI, Gemini CLI and Cursor too, and any tool that reads Agent Skills can use a copy of `skills/arch-review/`. Then ask in plain language:

> Review the architecture of this project. Is it ready for production?

> Review this pull request against our ADRs.

## How far to trust it

It has been tried on a handful of small synthetic projects (Node with Terraform, Bicep and CloudFormation) and several real .NET, Java and Node repositories. Expect rough edges on stacks it hasn't seen. If you hit one, [open an issue](https://github.com/gbrusella/arch-review-skill/issues).
