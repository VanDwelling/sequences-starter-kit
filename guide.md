# Building Governed Evaluation Pipelines with Sequences and Laws

## The core idea

A **sequence** is a staged evaluation pipeline. It takes an artefact (a draft, a deliverable, a model diff, a piece of code) and runs it through a series of gates. Each gate asks a specific question. The artefact either passes or it doesn't.

A **law** is the unit of governance inside a gate. It defines what must be true for the artefact to pass that gate. Laws are not guidelines, suggestions, or best practices. They are pass/fail criteria with explicit definitions of what passing and failing look like.

The separation matters. The sequence defines the *order* and *prerequisites*. The laws define the *criteria*. The skills define *how an agent evaluates* against those criteria. Three independent concerns, three independent places to change things.

## Why this exists

The problem this solves: you want Claude (or any agent) to evaluate work against your standards, but you need that evaluation to be governed, auditable, and tunable without rewriting prompts every time your standards evolve.

Without this pattern, quality standards live inside prompt text — scattered, implicit, hard to version, impossible to tune independently. When you want to change what counts as "good enough," you're editing prose in a prompt and hoping the agent interprets it the same way.

With this pattern, each criterion is a standalone file. You change one law, the agent reads the updated law next time it evaluates. You add a new law, it's automatically picked up by the stage that subscribes to it. You remove a law, gone. No prompt surgery.

## The anatomy

### Stages

A sequence has stages. Each stage has:

- **A question it answers** — "Does this pass mechanical checks?" or "Is this grounded in the model?" or "Would someone publish this under their name?"
- **An evaluation type** — either `deterministic` (a script checks it, no judgment) or `interpretive` (an agent reads the laws and makes a judgment call)
- **Decision values** — what outcomes are possible (PASS/FAIL, GROUNDED/UNGROUNDED, ACCEPT/REVISE/REJECT)
- **Prerequisites** — which earlier stages must have passed before this one runs

Design rule: **deterministic stages always come first**. They're cheap, fast, and catch mechanical errors before you spend agent tokens on interpretive evaluation. If a deliverable has broken formatting or banned vocabulary, there's no point asking an agent whether the argument is sound.

Design rule: **separate your interpretive axes**. If you have two different interpretive questions (e.g., "Is this technically accurate?" and "Is this on-brand?"), give each its own stage. Combined evaluation loses diagnostic precision — when something fails, you won't know *which* axis failed. It also makes independence enforcement impossible (more on this below).

### Laws

Each law is a standalone markdown file with YAML frontmatter:

```yaml
---
id: <uuid>
name: Human-readable name
evaluation: deterministic | interpretive
applies-to: [<format-list> | all]
---
```

The body has four sections:

1. **What it checks** — scope, not broader
2. **Why it matters** — the design reason this law exists (this is what makes the law tunable — when the "why" changes, you know to update or retire the law)
3. **What passing looks like** — the positive case
4. **What failing looks like** — concrete examples of failure

For deterministic laws, add a `Check configuration` section with the machine-readable config (word lists, thresholds, regex patterns). The script reads this section directly from the law file — no hardcoded values in the script.

For interpretive laws, the "what passing/failing looks like" sections are what the evaluator agent reads. Write them as evaluation criteria, not as code.

**The key invariant**: if output quality is consistently low, the correct fix is to revise the laws — not to auto-revise output against the current laws. Auto-revision reinforces the current law configuration without questioning whether the laws themselves are wrong. Laws are the unit of governance. Changing governance is a deliberate act.

### Law subscriptions

Laws are standalone. Stages subscribe to them. The subscription mapping lives in a `subscribed-laws.yaml` file per sequence:

```yaml
subscriptions:
  - law: <uuid>
    stage: deterministic-checks
  - law: <uuid>
    stage: interpretive-review
```

This separation means:
- A law can exist without being subscribed to any stage (dormant — useful for laws you're developing)
- A law can be subscribed to different stages in different sequences
- You can see at a glance which laws apply to which stage

### Evaluator skills

Each interpretive stage has a corresponding SKILL.md file. This tells the agent how to perform the evaluation. A skill includes:

- What it evaluates (scope)
- What to read first (which laws, what reference material)
- The evaluation protocol (step-by-step)
- The decision model (what outcomes are possible and what each means)
- Output format (structured, not prose)
- What it does NOT do (explicit scope negation — this prevents the agent from expanding its evaluation and collapsing stage boundaries)

**The skill reads laws, not hardcoded criteria.** The skill says "read the laws subscribed to this stage and evaluate against each." If a criterion changes, the law file changes — the skill's protocol stays the same.

### The runner

The runner is the mechanical backbone. It's a Python script that:

1. Runs deterministic stages automatically (calling the check script)
2. Records decisions from interpretive evaluators
3. Enforces prerequisites (stage N requires stage N-1 to have passed)
4. Enforces evaluator independence between interpretive stages
5. Manages the sequence log (create, read, update, archive)
6. Promotes artefact metadata on final acceptance (e.g., `status: draft` → `status: accepted`)

The runner is stateless per invocation. It reads the artefact and the log, does its work, writes the log. Safe to call from any context.

For interpretive stages, the runner does NOT call the evaluator. It outputs structured instructions that tell an agent what to invoke. This keeps the runner mechanical and the evaluation interpretive.

You don't need a runner on day one. If you're starting with a simple sequence, you can have the agent read the laws and evaluate directly. Build the runner when you need audit trails, prerequisite enforcement, or independence guarantees.

### The sequence log

One log per artefact, keyed by UUID. Lives in `sequences/<domain>/logs/`. Records what happened: which stages ran, what decisions were made, by whom, when. Structured YAML — no prose, queryable by agents.

The log also maintains a revision trail: if a stage is re-evaluated, the previous result is archived (not deleted). If an upstream stage is re-run, all downstream results are invalidated and archived. This gives you a complete audit history.

### Independence enforcement

For interpretive stages, context matters. The same agent that drafted a piece may be biased when evaluating it. The runner tracks `evaluator-id` per interpretive stage and refuses to record a later stage if its evaluator-id matches an earlier one.

This is the minimum structural guarantee — it prevents the most dangerous collapse (same evaluation stance for all interpretive axes). For maximum rigour, run interpretive stages in separate agent contexts (fresh conversation, no shared memory of the draft).

The trade-off:
- **Separate agent contexts per stage**: full independence, manual handoffs, use for high-stakes artefacts
- **Single orchestrator with distinct evaluator-ids**: partial independence (different IDs but shared context), convenient one-step execution, use for routine work
- **Same evaluator-id for all stages**: structurally blocked by the runner — never possible

## Design decisions worth understanding

### Why standalone law files (not a laws table or config block)

Each law must be independently readable (by scripts, agents, and humans), tunable (single-file edit to change a threshold), and versionable (git history per law). A single config file for all laws creates merge conflicts, makes it hard to see when a specific criterion changed, and tempts you to couple laws that should be independent.

### Why the artefact carries its own metadata

The runner reads everything it needs from the artefact's frontmatter: UUID, format, references, sources. No external configuration per artefact. This makes artefacts self-describing and portable. If you move an artefact, its metadata moves with it.

### Why logs are centralised (not co-located with artefacts)

Logs and artefacts have different audiences (agents vs humans), different lifecycles, and different query patterns. Centralised logs let you ask "which artefacts are stuck at stage 2?" without scanning every artefact directory.

### Why "revise laws, not drafts"

If you build an auto-revision loop (fail → revise → re-submit), you're training the system to work around its current laws rather than questioning whether the laws are right. This is an automation illusion — it looks like quality control but is actually law-entrenchment. The human decides when laws need to change.

## Smells to watch when building your own

**Combined evaluation stages.** If a single stage evaluates more than one axis, you lose diagnostic precision and can't enforce independence between axes. Split it.

**Laws without enforcement.** A law that no evaluator reads is a wish, not a constraint. Every law must have exactly one consumer that enforces it.

**Hardcoded thresholds.** If the check script has a hardcoded word list or character count, it will drift from the law file. The law file is the single source of truth.

**Orchestrator as evaluator.** If the orchestrator both drafts and evaluates, author-intent leakage is guaranteed. Document the trade-off. Provide an escape hatch.

**Missing prerequisite enforcement.** Check that prerequisites are enforced on ALL code paths — not just the "output instructions" path but also the "record decision" path. If a downstream decision can be recorded when an upstream stage hasn't passed, the entire sequence property collapses.

**Logs without revision trail.** If you can overwrite a stage result without archiving the previous one, you lose audit history.

**No reconciliation path for interpretive disagreement.** If two evaluators (or two runs of the same evaluator) disagree on an interpretive law, the pattern doesn't resolve that automatically — and it shouldn't. Interpretive disagreement is a signal, not a bug. It means either the law is ambiguous (fix the law) or the artefact is genuinely on the boundary (human decides). Don't build an auto-reconciliation loop. Surface the disagreement and let a human choose.

**Over-engineering before you have laws.** The runner, the log, the independence enforcement — these are structural safeguards. They matter when you have a real pipeline. But the first thing to get right is the laws. If your laws are vague, no amount of pipeline machinery will produce good evaluations. Start with laws. Make them sharp. Add machinery when you need it.

## What about the check script?

Deterministic laws need a script to enforce them. The script reads the artefact, loads each deterministic law file, extracts the `Check configuration` YAML block, and runs the check. It outputs structured pass/fail results per law.

You don't need to build this on day one. For your first iteration, you can have Claude read the deterministic laws and check the artefact manually (it's perfectly capable of doing substring matching and section detection). Build the script when you want: automation (no agent tokens spent on mechanical checks), consistency (the script applies the law identically every time), or speed (the script runs in milliseconds).

When you do build it, the contract is simple:

```python
# Pseudocode — adapt to your domain
def check_law(artefact_text, law_config):
    """Returns {law_name, verdict: PASS|FAIL, details: [...]}"""

def run_checks(artefact_path, laws_directory):
    """
    1. Read the artefact
    2. For each .md file in laws_directory where evaluation == deterministic:
       a. Parse the Check configuration YAML block
       b. Run the check
       c. Collect the result
    3. Return {passed: N, failed: N, results: [...]}
    """
```

The law file is the single source of truth for thresholds, word lists, and patterns. The script reads them at runtime — no hardcoded values.

## How to start

1. **Pick your domain.** What artefact are you evaluating? (A deliverable, a proposal, a code change, a design document.)

2. **Name your stages.** What questions must be answered, in what order? Keep it to 2-3 stages to start.

3. **Write your laws.** Start with 3-5 laws per stage. Each one should be specific enough that two people reading it would agree on whether a given artefact passes or fails. If they wouldn't agree, the law is too vague.

4. **Write your evaluator skills.** One per interpretive stage. The skill tells the agent: read these laws, evaluate this artefact against each, produce a structured verdict.

5. **Test by hand.** Before building a runner, have Claude evaluate an artefact using your laws and skills. See what works and what doesn't. Revise the laws.

6. **Build the runner when you need it.** When you want audit trails, prerequisite enforcement, or independence guarantees, that's when the runner earns its complexity.

## Reference implementation

The publication sequence in the `ai-resilience` repo is a full reference implementation. It has three stages (deterministic checks → worldview grounding → voice quality), ~20 laws, a runner with independence enforcement and revision trails, and evaluator skills for each interpretive stage. Study it if you want to see the pattern at full scale, but don't feel obligated to match its complexity on day one.
