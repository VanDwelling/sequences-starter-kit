# Worked Example: Client Deliverable Gate

A 2-stage sequence that evaluates client-facing deliverables (proposals, memos, reports) before they leave the building. Deliberately simple — the point is to see the pattern, not to build a production pipeline.

## The domain

You're a consultancy. Deliverables go to clients. Some quality problems are mechanical (wrong client name, missing sections, banned jargon). Some require judgment (is the recommendation actually supported by the analysis?). You want Claude to catch both before the deliverable reaches a human reviewer.

## The stages

| Stage | Name | Type | Question | Decision values |
|-------|------|------|----------|-----------------|
| 1 | `mechanical-checks` | deterministic | Does this deliverable pass basic formatting and content rules? | PASS / FAIL |
| 2 | `argument-quality` | interpretive | Is the core recommendation supported and actionable? | CLEAR / REVISE / WEAK |

Stage 2 requires stage 1 to PASS.

## The laws

### Stage 1: Mechanical checks (3 laws)

- **No placeholder text** — the deliverable contains no `[TODO]`, `[TBD]`, `[INSERT]`, or `PLACEHOLDER` markers
- **Executive summary present** — the first section is an executive summary that states the recommendation
- **No banned jargon** — no consulting-speak that obscures meaning (`synergies`, `leverage`, `best-in-class`, etc.)

### Stage 2: Argument quality (2 laws)

- **Recommendation grounded in analysis** — the recommendation follows from the analysis presented, not from unstated assumptions
- **Actionable next steps** — the deliverable ends with concrete, time-bound next steps that the client can act on without asking "but what do I actually do?"

## File structure

```
example/
  laws/
    no-placeholder-text.md          # deterministic
    executive-summary-present.md    # deterministic
    no-banned-jargon.md             # deterministic
    recommendation-grounded.md      # interpretive
    actionable-next-steps.md        # interpretive
  skills/
    argument-quality-evaluator.md   # SKILL.md for stage 2
  subscribed-laws.yaml              # maps laws → stages
```

No runner, no log, no orchestrator. That's deliberate. You add those when you need audit trails and independence enforcement. For now, the agent reads the skill, reads the laws, evaluates.

## How to invoke (in Claude Cowork)

```
Evaluate this deliverable against the client deliverable gate:
- File: <path to deliverable>
- Read the evaluator skill at example/skills/argument-quality-evaluator.md
- Run the mechanical checks first (laws in example/laws/ with evaluation: deterministic)
- Then run the argument quality evaluation
```

Or simpler, if you've set up your CLAUDE.md to recognise the sequence:

```
Run the deliverable gate on <file>.
```
