# Socratic Workflow Layer

Add a Socratic facilitation layer to the superpowers workflow that ensures users make well-reasoned decisions. The agent never recommends, suggests, or takes sides — it facilitates the user's thinking through adaptive questioning.

## Motivation

The current workflow has the agent acting as an advisor: recommending approaches, leading with preferred options, expressing opinions. This means users can accept decisions without fully reasoning through them. The Socratic layer flips this — the agent presents options neutrally, probes the user's justification, and holds firm until decisions are well-reasoned. The Design Rationale document is the artifact that proves this process worked.

## Architecture

Two new skills and modifications to four existing skills:

```
New:
  skills/socratic-facilitation/SKILL.md    — behavioral rules (cross-cutting)
  skills/design-rationale/SKILL.md         — ADR document generator

Modified:
  skills/brainstorming/SKILL.md            — references socratic-facilitation, invokes design-rationale
  skills/executing-plans/SKILL.md          — references socratic-facilitation (conditional)
  skills/subagent-driven-development/SKILL.md — references socratic-facilitation (conditional)
  skills/systematic-debugging/SKILL.md     — references socratic-facilitation (conditional)
```

The workflow chain (brainstorming → writing-plans → execution → finishing) stays the same. The Socratic layer wraps around decision points within each skill without changing the flow.

### Activation rules

- **Brainstorming:** socratic-facilitation is always active. Every decision point goes through Socratic probing.
- **Execution (executing-plans / subagent-driven-development):** socratic-facilitation activates only when a decision diverges from the original plan or involves a significant design/architecture choice not covered in brainstorming. Small implementation decisions are handled normally by the agent.
- **Debugging (systematic-debugging):** socratic-facilitation activates only when debugging reveals something that challenges original design assumptions. Normal hypothesis selection proceeds without it.

## Skill 1: `socratic-facilitation`

### Purpose

Define how the agent behaves at any decision point. A behavioral overlay, not a workflow. Can also be invoked standalone for Socratic exploration on any topic.

### Core behavioral rules

- Never recommend, suggest, or express preference — even when asked directly
- Present all options with equal weight, equal detail, equal framing
- When user asks "what would you do?" — redirect to help them think, never answer
- No proceeding without justified decisions (hold firm)

### Adaptive probing

Assessed per-question, not per-person. The same user might demonstrate deep understanding of API design and be lost on caching strategy.

- **Shallow justification** ("B sounds right"): Probe deeply. 3-5 questions about tradeoffs, failure modes, what they're optimizing for. Decompose into smaller questions if the user struggles.
- **Moderate justification** (mentions some tradeoffs but misses key ones): 2-3 targeted questions on the gaps. Don't re-ask what they've already addressed.
- **Strong justification** (articulates tradeoffs, failure modes, rejected alternatives): Acknowledge and move on. Ask about the biggest remaining blind spot, if any.

### When the user is stuck

1. First: decompose the question into smaller pieces they CAN reason about
2. If still stuck after 2-3 narrower questions: surface options without recommending ("Some teams handle this with X, others with Y — what resonates with your situation?")
3. Never say "I'd recommend X" or "The best approach is Y"

### Anti-pattern table

| Never say | Instead say |
|-----------|------------|
| "I'd recommend option A" | "What's drawing you toward one of these over the others?" |
| "My instinct is..." | "What does your instinct tell you, and what's behind it?" |
| "The best approach is..." | "What would 'best' mean for your situation?" |
| "Option A (recommended)" | "Option A" (no annotation) |
| "Great idea!" / "That's a solid approach!" | "What made you land on that?" |
| "I agree" | "Walk me through your reasoning" |
| "You should..." | "What happens if you...?" |

### Decision data capture

Track during conversation for downstream use (design-rationale, spec writing, etc.):
- Decisions made
- Alternatives considered
- Tradeoffs acknowledged
- User's justification (in their own reasoning, not the agent's interpretation)
- What was rejected and why
- Extra meaningful insights that emerged from the exploration

The socratic-facilitation skill owns the categories of data to capture. Downstream skills (like design-rationale) work with whatever data was captured — they do not define their own data requirements. This ensures a single source of truth for what gets tracked.

**Handoff format:** Decision data is passed as structured context when invoking downstream skills. The exact format (markdown summary, structured notes, etc.) is an implementation detail for the plan to determine.

### Trigger

- Referenced by other skills at their decision points
- Also invocable standalone for Socratic exploration on any topic

## Skill 2: `design-rationale`

### Purpose

Generate an ADR-style document from decision data captured during a Socratic exploration.

### Input

Decision data captured by the socratic-facilitation skill. The design-rationale skill does not define its own data requirements — it works with whatever was passed to it.

### Output

A document saved to `docs/superpowers/decisions/YYYY-MM-DD-<topic>-rationale.md`.

### Document format

The skill owns the document structure. It organizes the received data into a readable ADR-style narrative. Required structural elements:
- Context — what problem or decision was being explored
- Options Considered — each approach with its tradeoffs
- Decision — what was chosen
- Justification — the user's reasoning chain
- Tradeoffs Acknowledged — what the user recognized they're accepting
- Rejected Alternatives — what was decided against, and the user's reasoning

Optional sections when meaningful data emerged:
- Key insights from the exploration
- Assumptions the decision rests on
- Conditions that would trigger revisiting this decision

### Key principle

The document narrates the user's thought process. Not "the team decided" or "we recommend" — it reflects what the user reasoned through.

### Trigger

- Invoked by brainstorming after design approval
- Also invocable standalone to generate a rationale document from any Socratic exploration

## Modifications to existing skills

### Brainstorming

- Do NOT merge the `feat/brainstorm-engineer-growth` branch — its Socratic additions are superseded by the new `socratic-facilitation` skill
- On `main`'s version of the brainstorming skill, remove recommendation language:
  - Checklist item 4: "Propose 2-3 approaches — with trade-offs and your recommendation" → "Present 2-3 approaches — with trade-offs for each"
  - "Exploring approaches" section: remove "Present options conversationally with your recommendation and reasoning" and "Lead with your recommended option and explain why"
  - Replace with neutral language: "Present approaches with tradeoffs"
- Add reference: "At all decision points, follow the `socratic-facilitation` skill"
- Add reference: "After design approval, invoke `design-rationale` skill to generate the rationale document"
- Audit for any remaining language that conflicts with socratic-facilitation rules (any phrasing that implies the agent should recommend, suggest, prefer, or lead)

### Executing-plans

- Keep existing "Ask for clarification rather than guessing" at blockers
- Add: "When a blocker requires a direction decision from the user that diverges from the original plan or involves a significant design/architecture choice, follow the `socratic-facilitation` skill to guide the decision"

### Subagent-driven-development

- When implementer status is BLOCKED or NEEDS_CONTEXT and the resolution requires a user decision that diverges from the plan or is a significant design choice, reference socratic-facilitation
- Small implementation decisions proceed normally

### Systematic-debugging

- At Phase 3 (Hypothesis selection), when debugging reveals something that challenges original design assumptions, reference socratic-facilitation
- Normal hypothesis selection and investigation proceeds without it

## What this design does NOT change

- The workflow chain (brainstorming → writing-plans → execution → finishing)
- How specs are written (format, review loop, user review gate)
- How plans are decomposed
- How subagents are dispatched, reviewed, or re-dispatched
- TDD discipline
- Verification-before-completion
- Git worktree management
