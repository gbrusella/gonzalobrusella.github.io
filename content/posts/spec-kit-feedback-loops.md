---
title: "Multi-agent feedback loops with Spec-Kit"
date: 2026-06-14
draft: false
weight: 4
description: "Automating spec and plan reviews using Claude Code, Antigravity, and Codex in a Python feedback loop."
tags: ["spec-driven-development", "spec-kit", "multi-agent", "claude-code", "antigravity", "codex", "python"]
series: "Spec-Driven Development"
---

Spec-Kit's pipeline produces structured artifacts at each phase — `spec.md`, `plan.md`, `tasks.md`. Those artifacts are plain Markdown files, which means they're trivially consumable by any AI via API.

That's the opening for a multi-agent feedback loop: one agent authors the artifact, a second challenges it, the first revises, and the loop runs until the artifact passes review. The result is a spec or plan that's been pressure-tested before a single line of code gets written.

---

## Role assignments

For Spec-Kit's artifact sequence, the agent roles are:

| Role | Agent | Why |
|---|---|---|
| Author | Claude Code (Anthropic) | Strongest at structured reasoning and following constitutional constraints |
| Challenger | Antigravity (Google) | Effective at finding gaps and inconsistencies in requirements |
| Task validator | Codex (OpenAI) | Good at verifying task coverage and implementation feasibility |

The loop runs twice: once on `spec.md` (Author + Challenger), once on `plan.md` and `tasks.md` (Author + Codex).

---

## How the loop works

```
spec.md draft
    ↓
Antigravity challenges (gaps, contradictions, missing scenarios)
    ↓
Claude Code revises spec.md
    ↓
Antigravity approves or challenges again
    ↓  (converged)
plan.md draft
    ↓
Codex validates task coverage and feasibility
    ↓
Claude Code revises plan.md / tasks.md
    ↓
Codex approves or challenges again
    ↓  (converged)
/speckit.implement
```

Convergence is detected when the reviewer returns a structured JSON response with `"status": "APPROVED"`. The loop caps at 5 iterations to prevent infinite cycling on genuinely ambiguous specs.

---

## The orchestrator

```python
import anthropic
import openai
from google import genai
import json
import re
from pathlib import Path

# Initialize clients
anthropic_client = anthropic.Anthropic()
openai_client = openai.OpenAI()
google_client = genai.Client()

MAX_ITERATIONS = 5

def call_claude(system_prompt: str, messages: list) -> str:
    response = anthropic_client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=4096,
        system=system_prompt,
        messages=messages
    )
    return response.content[0].text

def call_antigravity(system_prompt: str, messages: list) -> str:
    history = "\n\n".join(
        f"{'User' if m['role'] == 'user' else 'Assistant'}: {m['content']}"
        for m in messages
    )
    response = google_client.models.generate_content(
        model="gemini-2.0-flash",
        contents=f"{system_prompt}\n\n{history}"
    )
    return response.text

def call_codex(system_prompt: str, messages: list) -> str:
    all_messages = [{"role": "system", "content": system_prompt}] + messages
    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=all_messages
    )
    return response.choices[0].message.content

def parse_review(response: str) -> tuple[str, str]:
    """Extract status and feedback from structured reviewer response."""
    try:
        json_match = re.search(r'\{.*?\}', response, re.DOTALL)
        if json_match:
            data = json.loads(json_match.group())
            return data.get("status", "NEEDS_REVISION"), data.get("feedback", response)
    except json.JSONDecodeError:
        pass
    status = "APPROVED" if "APPROVED" in response.upper() else "NEEDS_REVISION"
    return status, response

def run_spec_loop(spec_path: Path, constitution_path: Path) -> str:
    """Loop Claude Code + Antigravity on spec.md until approved."""
    constitution = constitution_path.read_text()
    spec = spec_path.read_text()

    author_system = f"""You are a senior software architect using Spec-Kit's spec-driven development methodology.
Your task is to write and refine a spec.md artifact following Spec-Kit conventions.
The spec must be technology-agnostic and cover: purpose, user scenarios, acceptance criteria, and non-functional requirements.
Project constitution (non-negotiable constraints):
{constitution}

When revising, apply all feedback precisely. Return only the updated spec.md content."""

    challenger_system = """You are a requirements analyst challenging a Spec-Kit spec.md artifact.
Your job is to find: missing scenarios, contradictions, ambiguous acceptance criteria, unconsidered edge cases, and violations of good spec-driven development practice.
Return JSON in this exact format:
{"status": "APPROVED" or "NEEDS_REVISION", "feedback": "your detailed feedback here"}
Be specific. Reference the exact section and line you're challenging."""

    author_messages = [{"role": "user", "content": f"Here is the current spec.md to review and improve:\n\n{spec}"}]
    challenger_messages = [{"role": "user", "content": f"Review this spec.md:\n\n{spec}"}]

    current_spec = spec

    for iteration in range(1, MAX_ITERATIONS + 1):
        print(f"\n[Spec loop] Iteration {iteration}/{MAX_ITERATIONS}")

        # Antigravity challenges
        challenge_response = call_antigravity(challenger_system, challenger_messages)
        status, feedback = parse_review(challenge_response)
        print(f"[Antigravity] Status: {status}")

        if status == "APPROVED":
            print("[Spec loop] Spec approved.")
            break

        print(f"[Antigravity] Feedback: {feedback[:200]}...")

        # Claude Code revises
        author_messages.append({"role": "assistant", "content": current_spec})
        author_messages.append({"role": "user", "content": f"Challenger feedback:\n\n{feedback}\n\nRevise the spec to address all points."})

        current_spec = call_claude(author_system, author_messages)
        author_messages.append({"role": "assistant", "content": current_spec})

        # Update challenger context
        challenger_messages.append({"role": "assistant", "content": challenge_response})
        challenger_messages.append({"role": "user", "content": f"Revised spec.md:\n\n{current_spec}"})

        # Write revised spec to disk
        spec_path.write_text(current_spec)

    return current_spec

def run_plan_loop(plan_path: Path, tasks_path: Path, spec_path: Path) -> tuple[str, str]:
    """Loop Claude Code + Codex on plan.md and tasks.md until approved."""
    spec = spec_path.read_text()
    plan = plan_path.read_text() if plan_path.exists() else ""
    tasks = tasks_path.read_text() if tasks_path.exists() else ""

    author_system = """You are a senior software architect using Spec-Kit's spec-driven development methodology.
Your task is to write and refine plan.md and tasks.md artifacts.
plan.md must cover: architecture decisions, component breakdown, tech stack rationale, and dependencies.
tasks.md must be an ordered checklist of implementation steps derived from the plan.
Return your response as two sections marked exactly:
## PLAN
[plan content]
## TASKS
[tasks content]"""

    validator_system = """You are a senior developer validating a Spec-Kit plan.md and tasks.md against the accepted spec.md.
Check for: missing requirements coverage, infeasible tasks, incorrect ordering, missing test tasks, and architectural gaps.
Return JSON in this exact format:
{"status": "APPROVED" or "NEEDS_REVISION", "feedback": "your detailed feedback here"}"""

    current_plan = plan
    current_tasks = tasks

    author_messages = [
        {"role": "user", "content": f"Accepted spec.md:\n\n{spec}\n\nCurrent plan.md:\n\n{plan}\n\nCurrent tasks.md:\n\n{tasks}\n\nRefine both artifacts."}
    ]
    validator_messages = [
        {"role": "user", "content": f"Spec.md:\n\n{spec}\n\nPlan.md:\n\n{plan}\n\nTasks.md:\n\n{tasks}"}
    ]

    for iteration in range(1, MAX_ITERATIONS + 1):
        print(f"\n[Plan loop] Iteration {iteration}/{MAX_ITERATIONS}")

        # Codex validates
        validation_response = call_codex(validator_system, validator_messages)
        status, feedback = parse_review(validation_response)
        print(f"[Codex] Status: {status}")

        if status == "APPROVED":
            print("[Plan loop] Plan and tasks approved.")
            break

        print(f"[Codex] Feedback: {feedback[:200]}...")

        # Claude Code revises
        author_messages.append({"role": "assistant", "content": f"## PLAN\n{current_plan}\n## TASKS\n{current_tasks}"})
        author_messages.append({"role": "user", "content": f"Validator feedback:\n\n{feedback}\n\nRevise both plan.md and tasks.md."})

        revision = call_claude(author_system, author_messages)

        # Parse sections
        plan_match = re.search(r'## PLAN\n(.*?)(?=## TASKS|\Z)', revision, re.DOTALL)
        tasks_match = re.search(r'## TASKS\n(.*)', revision, re.DOTALL)

        current_plan = plan_match.group(1).strip() if plan_match else current_plan
        current_tasks = tasks_match.group(1).strip() if tasks_match else current_tasks

        plan_path.write_text(current_plan)
        tasks_path.write_text(current_tasks)

        author_messages.append({"role": "assistant", "content": revision})
        validator_messages.append({"role": "assistant", "content": validation_response})
        validator_messages.append({"role": "user", "content": f"Revised plan.md:\n\n{current_plan}\n\nRevised tasks.md:\n\n{current_tasks}"})

    return current_plan, current_tasks

def run_speckit_feedback_loop(feature_dir: str, constitution_path: str):
    """Full Spec-Kit multi-agent feedback loop."""
    feature = Path(feature_dir)
    constitution = Path(constitution_path)

    spec_path = feature / "spec.md"
    plan_path = feature / "plan.md"
    tasks_path = feature / "tasks.md"

    if not spec_path.exists():
        raise FileNotFoundError(f"spec.md not found in {feature_dir}. Run /speckit.specify first.")

    print("=== Spec-Kit Multi-Agent Feedback Loop ===")
    print(f"Feature: {feature_dir}")
    print(f"Constitution: {constitution_path}")

    print("\n--- Phase 1: Spec review (Claude Code + Antigravity) ---")
    run_spec_loop(spec_path, constitution)

    print("\n--- Phase 2: Plan + Tasks review (Claude Code + Codex) ---")
    run_plan_loop(plan_path, tasks_path, spec_path)

    print("\n=== Loop complete. Artifacts are ready for /speckit.implement ===")

if __name__ == "__main__":
    import sys
    if len(sys.argv) != 3:
        print("Usage: python orchestrator.py <feature_dir> <constitution_path>")
        print("Example: python orchestrator.py .specify/features/001-user-auth .specify/memory/constitution.md")
        sys.exit(1)

    run_speckit_feedback_loop(sys.argv[1], sys.argv[2])
```

---

## Running the loop

After `/speckit.specify` generates the initial `spec.md`:

```bash
# Set API keys
export ANTHROPIC_API_KEY=your_key
export OPENAI_API_KEY=your_key
export GOOGLE_API_KEY=your_key

# Run the feedback loop before /speckit.plan
python code/spec-kit/orchestrator.py \
  .specify/features/001-user-auth \
  .specify/memory/constitution.md

# Once loop completes, continue the Spec-Kit pipeline
/speckit.plan
/speckit.tasks
/speckit.implement
```

The orchestrator writes revisions back to the artifact files in place, so when `/speckit.plan` runs next it reads the loop-hardened spec.

---

## What to expect

A typical spec for a non-trivial feature converges in 2–3 iterations. Antigravity tends to surface missing edge cases and unconsidered error scenarios in the first pass. The second pass usually challenges acceptance criteria specificity. By the third, most specs pass.

Plans take 1–2 iterations. Codex is good at spotting missing test tasks and incorrect dependency ordering — the things that cause implementation to stall mid-sprint.

The 5-iteration cap exists for a reason. A spec that can't converge in 5 rounds usually has a genuine ambiguity that needs a human decision, not more AI debate. Surface it to the team rather than letting the loop cycle indefinitely.
