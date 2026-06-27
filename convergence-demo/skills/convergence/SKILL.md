---
name: convergence-sequence
description: |
  **Convergence Sequence**: Runs a five-persona dialectic loop to sharpen any input — a claim, statement, draft, design question — until it is structurally defensible, strategically deployable, implementationally grounded, and regulatorily viable. Max 3 rounds; unresolved tensions are named, not smoothed. Trigger on: "converge on this", "run the convergence", "sharpen this", "five-persona review", "stress-test with the team", or any request to run a position through multi-perspective dialectic refinement.
---

# Convergence Sequence Orchestrator

You orchestrate a five-persona dialectic loop that takes any input and sharpens it through structured multi-perspective challenge until convergence or explicit disagreement.

## What this sequence does

It is a **quality gate and refinement loop** — not a content generator. It takes a statement, claim, draft, or design question and runs it through five structurally distinct evaluator stances until:

- **Convergence**: all five agents release (no remaining structural objections), OR
- **Named disagreement**: after 3 rounds, unresolved tensions are mapped explicitly. The disagreement IS the output — it tells you the idea has a genuine design conflict.

## The five agents

All five are **equal dialectic participants**. No hierarchy, no tiers, no "primary" vs "secondary." Each brings a structurally distinct lens:

| Agent | Skill path | Optimises for | Key question |
|---|---|---|---|
| **SF** (Satya Fowler) | `skills/persona-sf/SKILL.md` | Structural coherence + leverage | "What framing collapses the most complexity?" |
| **SC** (Structural Challenger) | `skills/persona-sc/SKILL.md` | Structural defensibility | "Where are the holes?" |
| **SI** (Strategic Interlocutor) | `skills/persona-si/SKILL.md` | Strategic deployability | "Does this land with consequential force?" |
| **Practitioner** | `skills/persona-practitioner/SKILL.md` | Operational truth | "What breaks first in a real organisation?" |
| **RR** (Regulatory Realist) | `skills/persona-rr/SKILL.md` | Regulatory feasibility | "Does regulation already mandate this?" |

## Invocation

The user provides:

- **Input**: The thing to sharpen (a claim, statement, draft, design question)
- **Context** (optional): Where this came from
- **Audience** (optional): Who will ultimately receive this (shapes SI's evaluation)

Example:

```
Run the convergence sequence:
- Input: "The missing production gate for AI in regulated enterprises is a Failure Mode Register — a documented, owned, and reviewable account of what the system gets wrong, when, and who carries it."
- Context: Output from a design riffing session
- Audience: CTO / Chief Risk Officer at a Dutch bank
```

## Execution protocol

### Step 0 — Read persona specs

Before spawning any agent, read ALL five persona skill files to understand each agent's stance, output format, and verdict protocol. The skill paths are listed in the table above — all relative to this `convergence-demo/` directory.

### Round 1 — Opening

**SF opens.** Spawn a sub-agent (Task tool, subagent_type: "general-purpose") with the following prompt structure:

```
Read the persona skill at skills/persona-sf/SKILL.md.
Adopt the SF (Satya Fowler) persona in loop-participant mode.

Your input:
- round: 1
- input_type: raw
- payload: [the user's input]

Produce your opening position following the output format in the skill spec.
```

Collect SF's opening position (claims, framing, leverage).

**Four challengers respond.** Spawn SC, SI, Practitioner, and RR as **parallel** sub-agents (all four in a single message with multiple Task tool calls). Each receives:

```
Read the persona skill at [skill path].
Adopt the [persona name] persona in loop-participant mode.

Your input:
- round: 1
- position: [SF's opening position — full text]

Produce your assessment following the output format in the skill spec.
```

Collect all four responses. Parse verdicts (HOLD/RELEASE).

### Round 2 — Absorption (if needed)

If ANY agent is HOLD:

**SF absorbs.** Spawn SF with all HOLD challenges as input.

**HOLD agents re-evaluate.** Spawn ONLY the agents that were HOLD in Round 1, in parallel. Each receives SF's revised position and their prior challenges.

Collect responses. Parse verdicts.

### Round 3 — Final (if needed)

If ANY agent is still HOLD after Round 2, run one final round with the same protocol.

**After Round 3, stop.** If agents are still HOLD, the disagreement is the output.

### Completion — Synthesis

After convergence (all RELEASE) or round cap (3 rounds):

Produce the **convergence report**:

```markdown
# Convergence Report

## Input
[Original input]

## Outcome
CONVERGED (round N) | DISAGREEMENT (after 3 rounds)

## Final Position
[SF's final position — claims, framing, leverage]

## Challenge Trajectory
[How each agent's challenges shaped the final position]
- SC: [key challenge → how absorbed]
- SI: [key challenge → how absorbed]
- Practitioner: [key challenge → how absorbed]
- RR: [key challenge → how absorbed]

## Compression
[SI's board sentence — the one-sentence version]

## First Move
[SI's recommended concrete action]

## Unresolved Tensions (if DISAGREEMENT)
[Named tensions with which agents disagree and why]

## Residual Risks (always present)
[Practitioner's remaining implementation risk]
[RR's jurisdiction scope and caveats]
```

## Structural properties

### Why max 3 rounds
Five opinionated agents can theoretically ping-pong indefinitely. Three rounds is the empirical sweet spot: Round 1 surfaces the big issues, Round 2 resolves most of them, Round 3 catches what was missed. Beyond that, diminishing returns.

### Why parallel challengers
SC, SI, Practitioner, and RR challenge independently — they don't need to see each other's challenges. Parallel execution saves time and prevents groupthink.

### Why SF absorbs (not an external synthesiser)
SF is the architect. Its job is to integrate structural challenges into a coherent position. An external synthesiser would either duplicate SF's role or produce a mechanical merge without architectural judgment.

### Convergence vs agreement
Convergence is NOT agreement. It means no agent has a remaining *structural* objection. Agents may still prefer different framings. RELEASE means "I can't break this," not "I endorse this."

## What this sequence does NOT do

- Generate content from scratch — it refines existing input
- Replace human judgment — the convergence report is advisory
- Guarantee quality — convergence means "survived five structural tests," not "is necessarily good"
