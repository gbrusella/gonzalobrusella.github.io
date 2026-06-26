---
title: "Multi-agent feedback loops with OpenSpec"
date: 2026-06-20
draft: false
weight: 7
description: "Using Codex, Claude Code, and Antigravity to pressure-test OpenSpec proposals and design documents before implementation."
tags: ["spec-driven-development", "openspec", "multi-agent", "claude-code", "antigravity", "codex", "python"]
series: "Spec-Driven Development"
---

OpenSpec's core artifact is the `proposal.md` — a document that captures what will be built, why, and how before any code gets written. That artifact is small enough to be consumed and critiqued by an AI in a single API call, and structured enough that feedback can be precise.

That's the right target for a feedback loop. Author a proposal, challenge it from a different perspective, revise, repeat until it's solid.

---

## Role assignments

OpenSpec's brownfield focus and spec delta model shape the role assignments:

| Role | Agent | Why |
|---|---|---|
| Author | Codex (OpenAI) | Strong at producing structured technical proposals and design decisions |
| Spec reviewer | Claude Code (Anthropic) | Best at verifying requirements completeness and catching missing scenarios |
| Design challenger | Antigravity (Google) | Effective at stress-testing technical design decisions and surfacing alternatives |

The loop runs in two phases: requirements review (Codex + Claude Code on `proposal.md` and `spec.md`), then design challenge (Codex + Antigravity on `design.md`). Separating concerns keeps feedback focused — requirements questions don't bleed into architecture debates.

---

## How the loop works

```
/openspec:proposal generates initial artifacts
    ↓
Phase 1: Requirements review
  Claude Code reviews proposal.md + spec.md
    ↓ (missing scenarios, ambiguous requirements)
  Codex revises proposal.md + spec.md
    ↓ (repeat until APPROVED)

Phase 2: Design challenge
  Antigravity stress-tests design.md
    ↓ (alternative approaches, scalability concerns, hidden dependencies)
  Codex revises design.md
    ↓ (repeat until APPROVED)

Approved artifacts committed → implementation
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

def run_requirements_loop(proposal_path: Path, spec_path: Path, existing_specs_dir: Path) -> tuple[str, str]:
    """Phase 1: Codex authors, Claude Code reviews requirements."""
    proposal = proposal_path.read_text() if proposal_path.exists() else ""
    spec = spec_path.read_text() if spec_path.exists() else ""

    # Load existing specs for context
    existing_specs = ""
    if existing_specs_dir.exists():
        for spec_file in existing_specs_dir.rglob("spec.md"):
            existing_specs += f"\n\n--- {spec_file} ---\n{spec_file.read_text()}"

    author_system = """You are a senior developer using OpenSpec's proposal-driven workflow.
Your task is to write and refine two artifacts:
1. proposal.md — what will be built, user scenarios, rationale, and acceptance criteria
2. spec.md — functional requirements using SHALL statements and Given/When/Then scenarios

Proposals must be brownfield-aware: consider how changes affect existing behavior.
Return your response as two sections:
## PROPOSAL
[proposal content]
## SPEC
[spec content]"""

    reviewer_system = f"""You are a requirements analyst reviewing an OpenSpec proposal.md and spec.md.
Your focus is requirements quality: completeness, consistency with existing specs, missing scenarios, and ambiguous SHALL statements.

Existing system specs for context:
{existing_specs if existing_specs else "No existing specs — this is a greenfield feature."}

Return JSON:
{{"status": "APPROVED" or "NEEDS_REVISION", "feedback": "specific, section-referenced feedback"}}"""

    current_proposal = proposal
    current_spec = spec

    author_messages = [
        {"role": "user", "content": f"Current proposal.md:\n\n{proposal}\n\nCurrent spec.md:\n\n{spec}\n\nRefine both artifacts."}
    ]
    reviewer_messages = [
        {"role": "user", "content": f"Review this proposal.md:\n\n{proposal}\n\nAnd spec.md:\n\n{spec}"}
    ]

    for iteration in range(1, MAX_ITERATIONS + 1):
        print(f"\n[Requirements loop] Iteration {iteration}/{MAX_ITERATIONS}")

        review = call_claude(reviewer_system, reviewer_messages)
        status, feedback = parse_review(review)
        print(f"[Claude Code] Status: {status}")

        if status == "APPROVED":
            print("[Requirements loop] Approved.")
            break

        print(f"[Claude Code] Feedback: {feedback[:200]}...")

        author_messages.append({"role": "assistant", "content": f"## PROPOSAL\n{current_proposal}\n## SPEC\n{current_spec}"})
        author_messages.append({"role": "user", "content": f"Reviewer feedback:\n\n{feedback}\n\nRevise both artifacts."})

        revision = call_codex(author_system, author_messages)
        author_messages.append({"role": "assistant", "content": revision})

        proposal_match = re.search(r'## PROPOSAL\n(.*?)(?=## SPEC|\Z)', revision, re.DOTALL)
        spec_match = re.search(r'## SPEC\n(.*)', revision, re.DOTALL)

        current_proposal = proposal_match.group(1).strip() if proposal_match else current_proposal
        current_spec = spec_match.group(1).strip() if spec_match else current_spec

        proposal_path.write_text(current_proposal)
        spec_path.write_text(current_spec)

        reviewer_messages.append({"role": "assistant", "content": review})
        reviewer_messages.append({"role": "user", "content": f"Revised proposal.md:\n\n{current_proposal}\n\nRevised spec.md:\n\n{current_spec}"})

    return current_proposal, current_spec

def run_design_loop(design_path: Path, proposal_path: Path) -> str:
    """Phase 2: Codex authors, Antigravity stress-tests design decisions."""
    design = design_path.read_text() if design_path.exists() else ""
    proposal = proposal_path.read_text()

    author_system = """You are a senior software architect writing an OpenSpec design.md artifact.
design.md captures technical decisions: component design, data models, API contracts, infrastructure choices, and dependency analysis.
Each decision should include the rationale and alternatives considered.
Return only the updated design.md content."""

    challenger_system = """You are a senior architect stress-testing a technical design document.
Challenge: scalability assumptions, hidden dependencies, over-engineering, missing failure modes, better alternatives, and security implications.
Be direct. If a decision is wrong, say so and explain why.
Return JSON:
{"status": "APPROVED" or "NEEDS_REVISION", "feedback": "specific challenge with suggested alternatives"}"""

    current_design = design
    author_messages = [
        {"role": "user", "content": f"Accepted proposal.md:\n\n{proposal}\n\nCurrent design.md:\n\n{design}\n\nRefine the design document."}
    ]
    challenger_messages = [
        {"role": "user", "content": f"Proposal context:\n\n{proposal}\n\nChallenge this design.md:\n\n{design}"}
    ]

    for iteration in range(1, MAX_ITERATIONS + 1):
        print(f"\n[Design loop] Iteration {iteration}/{MAX_ITERATIONS}")

        challenge = call_antigravity(challenger_system, challenger_messages)
        status, feedback = parse_review(challenge)
        print(f"[Antigravity] Status: {status}")

        if status == "APPROVED":
            print("[Design loop] Approved.")
            break

        print(f"[Antigravity] Feedback: {feedback[:200]}...")

        author_messages.append({"role": "assistant", "content": current_design})
        author_messages.append({"role": "user", "content": f"Design challenge:\n\n{feedback}\n\nRevise the design document to address these points."})

        current_design = call_codex(author_system, author_messages)
        author_messages.append({"role": "assistant", "content": current_design})
        design_path.write_text(current_design)

        challenger_messages.append({"role": "assistant", "content": challenge})
        challenger_messages.append({"role": "user", "content": f"Revised design.md:\n\n{current_design}"})

    return current_design

def run_openspec_feedback_loop(change_dir: str, specs_dir: str):
    """Full OpenSpec multi-agent feedback loop."""
    change = Path(change_dir)
    specs = Path(specs_dir)

    proposal_path = change / "proposal.md"
    design_path = change / "design.md"

    # The spec.md lives in openspec/specs/<feature>/spec.md
    # Infer from the change directory name
    feature_name = change.name
    spec_path = specs / feature_name / "spec.md"
    spec_path.parent.mkdir(parents=True, exist_ok=True)

    if not proposal_path.exists():
        raise FileNotFoundError(f"proposal.md not found in {change_dir}. Run /openspec:proposal first.")

    print("=== OpenSpec Multi-Agent Feedback Loop ===")
    print(f"Change: {change_dir}")

    print("\n--- Phase 1: Requirements review (Codex + Claude Code) ---")
    run_requirements_loop(proposal_path, spec_path, specs)

    print("\n--- Phase 2: Design challenge (Codex + Antigravity) ---")
    run_design_loop(design_path, proposal_path)

    print("\n=== Loop complete. Artifacts ready for implementation. ===")

if __name__ == "__main__":
    import sys
    if len(sys.argv) != 3:
        print("Usage: python orchestrator.py <change_dir> <specs_dir>")
        print("Example: python orchestrator.py openspec/changes/add-remember-me openspec/specs")
        sys.exit(1)

    run_openspec_feedback_loop(sys.argv[1], sys.argv[2])
```

---

## Running the loop

```bash
# Generate initial artifacts
# (run in your agent: /openspec:proposal Add remember me with 30-day sessions)

# Set API keys
export ANTHROPIC_API_KEY=your_key
export OPENAI_API_KEY=your_key
export GOOGLE_API_KEY=your_key

# Run the feedback loop
python code/openspec/orchestrator.py \
  openspec/changes/add-remember-me \
  openspec/specs

# Commit the approved artifacts
git add openspec/
git commit -m "feat: reviewed proposal and spec for remember-me feature"
```

---

## The spec delta after the loop

One of the benefits of running the loop before implementation: when the spec delta gets generated (showing how `auth-session/spec.md` changed), it reflects a requirements set that has already been challenged and defended. The delta in the PR isn't a first draft — it's a considered position.

That changes the tone of code review. Instead of the reviewer questioning requirements, they're verifying implementation against an agreed spec. Faster, more focused, fewer comment threads that spiral into product debates.
