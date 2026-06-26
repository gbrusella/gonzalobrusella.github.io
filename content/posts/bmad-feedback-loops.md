---
title: "Multi-agent feedback loops with BMad"
date: 2026-06-26
draft: false
weight: 10
description: "Augmenting BMad's native adversarial review with external Antigravity and Codex challenges for cross-provider diversity."
tags: ["spec-driven-development", "bmad", "multi-agent", "claude-code", "antigravity", "codex", "python"]
series: "Spec-Driven Development"
---

BMad already has multi-agent feedback built in — the adversarial review agent, the QA agent, Party Mode. So using external agents for feedback isn't a workaround; it's an extension of what BMad already does natively.

The interesting question is where external agents add value that BMad's internal agents don't. The answer: cross-provider diversity. BMad's agents all run on the same underlying model. An external Antigravity or Codex challenge brings a genuinely different reasoning baseline — different training data, different strengths, different failure modes.

Running a design through BMad's adversarial agent and then Antigravity is like having two different senior architects review it. They'll surface different concerns.

---

## Role assignments

BMad's agent sequence maps to providers based on their strengths at each phase:

| Role | Agent | Why |
|---|---|---|
| PM / Author | Claude Code (Anthropic) | BMad's recommended runner; best at structured elicitation and following BMad's agent protocols |
| Architect | Claude Code (Anthropic) | Strongest on system design reasoning and architecture tradeoffs |
| Adversarial reviewer | Antigravity (Google) | External perspective; different reasoning baseline from the architect |
| Implementation validator | Codex (OpenAI) | Strong at verifying implementation feasibility and task coverage |

The loop runs at two points in the BMad sequence: after the architecture document (Antigravity challenges), and after the task breakdown (Codex validates). BMad's internal adversarial agent still runs — the external loop adds a second, provider-diverse pass.

---

## How the loop works

```
BMad PM agent → product brief
    ↓
BMad Architect agent → architecture document
    ↓
[External loop 1]
  Antigravity challenges architecture
    ↓ (gaps, alternatives, failure modes)
  Claude Code (Architect) revises
    ↓ (repeat until APPROVED)
    ↓
BMad Developer agent → task breakdown
    ↓
[External loop 2]
  Codex validates task coverage and feasibility
    ↓ (missing tasks, incorrect ordering, implementation concerns)
  Claude Code (Developer) revises
    ↓ (repeat until APPROVED)
    ↓
BMad Developer agent implements
BMad QA agent verifies
```

---

## The orchestrator

```python
import anthropic
import openai
from google import genai
import json
import re
from pathlib import Path

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
    try:
        json_match = re.search(r'\{.*?\}', response, re.DOTALL)
        if json_match:
            data = json.loads(json_match.group())
            return data.get("status", "NEEDS_REVISION"), data.get("feedback", response)
    except json.JSONDecodeError:
        pass
    status = "APPROVED" if "APPROVED" in response.upper() else "NEEDS_REVISION"
    return status, response

def run_architecture_loop(architecture_path: Path, brief_path: Path) -> str:
    """Loop: Claude Code (Architect) + Antigravity adversarial review."""
    brief = brief_path.read_text()
    architecture = architecture_path.read_text()

    architect_system = f"""You are a senior software architect working within the BMad Method framework.
You have received a product brief and are responsible for the architecture document.
The architecture must cover: system components, data models, API contracts, infrastructure, integration points, and key technical decisions with rationale.
Product brief context:
{brief}

When revising, address all adversarial feedback directly and document why each decision stands or how it changed.
Return only the updated architecture document."""

    adversary_system = """You are an adversarial architecture reviewer. Your goal is to find problems.
Challenge: scalability ceiling, single points of failure, implicit assumptions, missing failure handling, over-complexity, security gaps, performance bottlenecks, and integration risks.
You are from a different team than the architect. You have no stake in defending these decisions.
Return JSON:
{"status": "APPROVED" or "NEEDS_REVISION", "feedback": "specific challenges with suggested alternatives where applicable"}
APPROVED only when you genuinely cannot find material issues."""

    current_architecture = architecture
    architect_messages = [
        {"role": "user", "content": f"Review and improve this architecture document:\n\n{architecture}"}
    ]
    adversary_messages = [
        {"role": "user", "content": f"Product brief:\n\n{brief}\n\nChallenge this architecture:\n\n{architecture}"}
    ]

    for iteration in range(1, MAX_ITERATIONS + 1):
        print(f"\n[Architecture loop] Iteration {iteration}/{MAX_ITERATIONS}")

        challenge = call_antigravity(adversary_system, adversary_messages)
        status, feedback = parse_review(challenge)
        print(f"[Antigravity] Status: {status}")

        if status == "APPROVED":
            print("[Architecture loop] Architecture approved.")
            break

        print(f"[Antigravity] Feedback: {feedback[:200]}...")

        architect_messages.append({"role": "assistant", "content": current_architecture})
        architect_messages.append({
            "role": "user",
            "content": f"Adversarial feedback:\n\n{feedback}\n\nRevise the architecture to address these concerns. Document your responses to each challenge."
        })

        current_architecture = call_claude(architect_system, architect_messages)
        architect_messages.append({"role": "assistant", "content": current_architecture})
        architecture_path.write_text(current_architecture)

        adversary_messages.append({"role": "assistant", "content": challenge})
        adversary_messages.append({"role": "user", "content": f"Revised architecture:\n\n{current_architecture}"})

    return current_architecture

def run_tasks_loop(tasks_path: Path, architecture_path: Path, brief_path: Path) -> str:
    """Loop: Claude Code (Developer) + Codex implementation validation."""
    brief = brief_path.read_text()
    architecture = architecture_path.read_text()
    tasks = tasks_path.read_text() if tasks_path.exists() else ""

    developer_system = f"""You are a senior developer working within the BMad Method framework.
You produce implementation task breakdowns from architecture documents.
Tasks must be: ordered by dependency, include test tasks, reference specific architecture components, and be granular enough to estimate.
Product brief:
{brief}
Architecture:
{architecture}

Return only the updated tasks document as an ordered markdown checklist."""

    validator_system = f"""You are a senior developer validating a BMad task breakdown.
Check against the architecture and brief for: missing feature coverage, incorrect dependency ordering, missing test tasks, infeasible tasks given the architecture, and tasks that are too large to be atomic.
Architecture context:
{architecture}
Brief acceptance criteria:
{brief}

Return JSON:
{{"status": "APPROVED" or "NEEDS_REVISION", "feedback": "specific issues with task references"}}"""

    current_tasks = tasks
    developer_messages = [
        {"role": "user", "content": f"Produce an implementation task breakdown from this architecture:\n\n{architecture}\n\nCurrent tasks (refine if provided):\n\n{tasks}"}
    ]
    validator_messages = [
        {"role": "user", "content": f"Validate this task breakdown:\n\n{tasks}"}
    ]

    for iteration in range(1, MAX_ITERATIONS + 1):
        print(f"\n[Tasks loop] Iteration {iteration}/{MAX_ITERATIONS}")

        validation = call_codex(validator_system, validator_messages)
        status, feedback = parse_review(validation)
        print(f"[Codex] Status: {status}")

        if status == "APPROVED":
            print("[Tasks loop] Tasks approved.")
            break

        print(f"[Codex] Feedback: {feedback[:200]}...")

        developer_messages.append({"role": "assistant", "content": current_tasks})
        developer_messages.append({
            "role": "user",
            "content": f"Validator feedback:\n\n{feedback}\n\nRevise the task breakdown to address all issues."
        })

        current_tasks = call_claude(developer_system, developer_messages)
        developer_messages.append({"role": "assistant", "content": current_tasks})
        tasks_path.write_text(current_tasks)

        validator_messages.append({"role": "assistant", "content": validation})
        validator_messages.append({"role": "user", "content": f"Revised tasks:\n\n{current_tasks}"})

    return current_tasks

def run_bmad_feedback_loop(brief_path: str, architecture_path: str, tasks_path: str):
    """External feedback loops augmenting BMad's native agent sequence."""
    brief = Path(brief_path)
    architecture = Path(architecture_path)
    tasks = Path(tasks_path)

    if not brief.exists():
        raise FileNotFoundError(f"Brief not found: {brief_path}. Run the BMad PM agent first.")
    if not architecture.exists():
        raise FileNotFoundError(f"Architecture not found: {architecture_path}. Run the BMad Architect agent first.")

    print("=== BMad External Feedback Loop ===")
    print("Augmenting BMad's internal adversarial review with cross-provider challenges.")

    print("\n--- Phase 1: Architecture challenge (Claude Code + Antigravity) ---")
    run_architecture_loop(architecture, brief)

    print("\n--- Phase 2: Task validation (Claude Code + Codex) ---")
    run_tasks_loop(tasks, architecture, brief)

    print("\n=== External loop complete. ===")
    print("Continue with BMad Developer agent using the validated artifacts.")

if __name__ == "__main__":
    import sys
    if len(sys.argv) != 4:
        print("Usage: python orchestrator.py <brief_path> <architecture_path> <tasks_path>")
        print("Example: python orchestrator.py docs/brief.md docs/architecture.md docs/tasks.md")
        sys.exit(1)

    run_bmad_feedback_loop(sys.argv[1], sys.argv[2], sys.argv[3])
```

---

## Running the loop

```bash
# After BMad PM agent produces brief.md and Architect agent produces architecture.md

export ANTHROPIC_API_KEY=your_key
export OPENAI_API_KEY=your_key
export GOOGLE_API_KEY=your_key

# Run external feedback loops
python code/bmad/orchestrator.py \
  docs/brief.md \
  docs/architecture.md \
  docs/tasks.md

# Then continue with BMad Developer agent
# using the validated architecture and task list
```

---

## Native vs. external adversarial review

BMad's internal adversarial agent and the external Antigravity challenge serve different purposes and are worth running both.

BMad's internal adversary knows the BMad context — it understands the product brief format, the agent handoff protocol, and the framework's conventions. It's excellent at catching issues within the BMad workflow.

Antigravity doesn't know BMad. It reads the architecture document as a standalone artifact and challenges it from first principles. That's exactly what you want from a second reviewer: someone who isn't primed by the same context and can surface assumptions the first reviewer shared.

The disagreements between the two are usually the most valuable output. When BMad's adversary approves something that Antigravity challenges, that tension is worth investigating before the Developer agent starts writing code.
