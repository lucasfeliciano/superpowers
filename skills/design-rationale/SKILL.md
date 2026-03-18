---
name: design-rationale
description: "Use when generating a design rationale document after a Socratic exploration or brainstorming session. Also invocable standalone to document decisions from any exploration."
---

# Design Rationale

Generate an ADR-style document from decision data captured during a Socratic exploration.

## Purpose

Create a document that narrates the **user's** thought process — not "the team decided" or "we recommend." The document reflects what the user reasoned through: what they considered, what they chose, why they chose it, and what they explicitly decided against.

## Input

Decision data captured by the `socratic-facilitation` skill. This skill does not define its own data requirements — it works with whatever decision data was passed to it.

The data categories are defined by the socratic-facilitation skill and typically include:
- Decisions made
- Alternatives considered
- Tradeoffs acknowledged
- User's justification
- What was rejected and why
- Extra meaningful insights from the exploration

## Output

Save the document to: `docs/superpowers/decisions/YYYY-MM-DD-<topic>-rationale.md`

(User preferences for document location override this default)

## Document Structure

Organize the received data into these sections:

### Required sections (always include):

- **Context** — what problem or decision was being explored. Set the scene for why this decision needed to be made.
- **Options Considered** — each approach that was presented, with its tradeoffs as discussed during the exploration. Present neutrally — this is a record, not a recommendation.
- **Decision** — what was chosen.
- **Justification** — the user's reasoning chain. Write this from the user's perspective, capturing their actual reasoning, not the agent's interpretation or summary.
- **Tradeoffs Acknowledged** — what the user explicitly recognized they're accepting by making this choice.
- **Rejected Alternatives** — what was decided against, and the user's reasoning for ruling each out.

### Optional sections (include when meaningful data emerged):

- **Key Insights** — non-obvious things that emerged from the exploration that add context to the decisions.
- **Assumptions** — what the decision rests on. If these assumptions change, the decision may need revisiting.
- **Revisit Conditions** — specific conditions or events that would trigger reconsidering this decision.

## Writing Style

- Use the user's own words and reasoning where possible
- Do not editorialize or add the agent's opinion
- Do not use "we recommend" or "the best approach" — this document records what the user decided, not what the agent thinks
- Be specific and concrete — "user chose X because of latency constraints" not "user preferred X"
- Keep it concise but complete — every decision should have a clear justification trail

## Process

1. Gather all decision data from the Socratic exploration
2. Organize into the document structure above
3. Write the document
4. Save to the decisions directory
5. Commit to git

## Trigger

- Invoked by brainstorming after design approval (before writing the spec)
- Also invocable standalone to generate a rationale document from any Socratic exploration
