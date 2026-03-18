# Design Rationale: Socratic Workflow Layer

## Context

The superpowers workflow has the agent acting as an advisor — recommending approaches, leading with preferred options, expressing opinions via language like "my instinct" and "(recommended)." A previous attempt to fix this by editing the brainstorming skill directly was insufficient: the agent still snuck in recommendations despite explicit instructions not to. The goal is to make the agent a Socratic facilitator that helps the user reason through decisions, never deciding for them.

## Options Considered

### Approach A: Cross-cutting Socratic skill + modify existing skills to reference it

Create a single `socratic-facilitation` skill defining behavioral rules. Existing skills reference it at decision points.

- **Tradeoffs:** Single source of truth, easy to iterate. But adds a dependency between skills, and rules in a separate file might get ignored by the LLM.

### Approach B: Inline Socratic rules in each skill + separate design-rationale skill

Embed Socratic behavior directly into each skill. Create a separate skill for generating the rationale document.

- **Tradeoffs:** Each skill is self-contained when loaded. But duplicates rules across skills, harder to maintain if the Socratic philosophy evolves.

### Approach C: Modify brainstorming only, rationale as a section in the spec

Focus all changes on the brainstorming skill. Add Design Rationale as a section in the existing spec document.

- **Tradeoffs:** Smallest change, fastest to validate. But doesn't address execution-phase decisions, and this approach was already tried and proved insufficient.

## Decision

**Approach A (cross-cutting skill) + the design-rationale skill from Approach B.**

## Justification

The user chose the cross-cutting approach because it optimizes for **iteration speed**. This is explicitly an experiment — the Socratic rules may not be strong enough to override Claude's default helpful-advisor behavior on the first try. Having the rules encapsulated in a single skill means changes propagate everywhere when the rules need to get stronger. The user acknowledged they don't have confidence it will work perfectly, but chose to optimize for the ability to iterate quickly.

The design-rationale skill was added as a separate skill (from Approach B) because it follows the same modularity principle: one skill, one responsibility. The Socratic skill owns conversation behavior; the rationale skill owns document generation. This matches the existing superpowers pattern where each skill has a clear, single purpose.

## Tradeoffs Acknowledged

- **Cross-cutting reference might be ignored:** The LLM might not follow rules from a referenced skill as strongly as inline rules. The user accepts this risk and will monitor behavior, iterating on the skill's language if needed.
- **Maintenance of data capture categories:** The socratic-facilitation skill owns the list of what data to capture. The design-rationale skill works with whatever it receives. This avoids maintaining two lists that could drift apart, but means the rationale document structure adapts to input rather than enforcing specific expectations.

## Rejected Alternatives

- **Inline rules per skill (Approach B pure):** Rejected because duplicating Socratic rules across multiple skills makes iteration harder. When the rules need to change (and they will — this is an experiment), changing one file is better than changing four.
- **Brainstorming-only changes (Approach C):** Rejected because it was already tried and failed. The agent still recommended despite explicit instructions. Also doesn't cover execution-phase decisions.
- **Design Rationale as a section in the spec document:** Rejected in favor of a separate document because it makes the rationale more focused and maintainable. However, the user didn't have a strong opinion here — the deciding factor was modularity and the ability to invoke the rationale skill independently.
- **Agent answers when asked directly ("what would you do?"):** Rejected entirely. The agent must never give its opinion, even when the user explicitly asks. Always redirect to help the user think.
- **Socratic activation on all execution decisions:** Rejected because by execution time, major decisions are done. Triggering Socratic exploration on small implementation decisions would slow things down with no growth value. Only activates for decisions that diverge from the plan or involve significant design choices.

## Key Insights

- The previous attempt failed not because the rules were wrong, but because they were mixed into a skill whose primary instructions still said "recommend." Conflicting instructions within the same skill let the LLM rationalize following the original behavior. Separating concerns (brainstorming owns workflow, Socratic owns behavior) should reduce this conflict.
- The user explicitly rejected the agent ever answering "what would you do?" — this is a hard line, not a preference. The growth value comes from the user reasoning through decisions, and any agent opinion undermines that.
- Probing depth should be adaptive per-question, not per-person. The same engineer might demonstrate deep expertise in one area and need scaffolding in another.
