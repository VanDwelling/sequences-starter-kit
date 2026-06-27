# Convergence Sequence — Live Demo

A self-contained, copy-paste-able five-persona dialectic that runs in Claude Cowork. Drop it in, invoke it, watch it work.

## What it does

Takes any input — a claim, position, design decision, draft paragraph — and runs it through five structurally distinct evaluation perspectives in up to 3 rounds. The output is either a sharpened position (all five perspectives released their objections) or a named disagreement (the idea has a genuine design conflict that can't be resolved by iteration).

The five perspectives: architectural coherence (SF), structural holes (SC), audience deployability (SI), implementation reality (Practitioner), regulatory viability (RR).

## Setup (2 minutes)

### Step 1: Copy the skills

Copy the entire `skills/` folder into your repo's `.claude/skills/` directory:

```bash
cp -r skills/* /path/to/your-repo/.claude/skills/
```

You should end up with:
```
.claude/skills/
  convergence/SKILL.md
  persona-sf/SKILL.md
  persona-sc/SKILL.md
  persona-si/SKILL.md
  persona-practitioner/SKILL.md
  persona-rr/SKILL.md
```

### Step 2: Add to CLAUDE.md

Paste this block into your repo's `CLAUDE.md` file:

```markdown
## Convergence Sequence

Five-persona dialectic for sharpening claims, positions, and design decisions.

**Invoke with:**
- "Run the convergence sequence on: [your input]"
- "Converge on this: [your input]"
- "Sharpen this: [your input]"

**What happens:** SF opens with a structural position. SC, SI, Practitioner, and RR challenge in parallel. SF absorbs. HOLD agents re-evaluate. Max 3 rounds. Output is a convergence report.

**Personas available standalone:**
- "SF stance on [topic]" — architectural framing
- "SC stance on [position]" — structural holes (Ackoff, Beer, Feynman, Munger)
- "SI stance on [position]" — audience fit, decision-change test, compression
- "Practitioner stance on [position]" — first failure point, capability mapping
- "RR stance on [position]" — regulatory mapping (EU AI Act, DORA, EBA, DNB, FCA, ECB)
```

### Step 3: Try it

Open a Cowork session and say:

```
Run the convergence sequence on: "Every AI system in production should have a Failure Mode Register — a documented, owned, and reviewable account of what the system gets wrong, when, and who carries it."
```

Or start with a single persona:

```
SC stance on: "Rate limiting is sufficient protection against prompt injection in enterprise LLM deployments."
```

## What to watch for

When the sequence runs, notice:

1. **SF's opening** — how it reframes the raw input into explicit claims with invariants
2. **The four parallel challenges** — each perspective finds different problems (SC finds logical holes, Practitioner finds capability gaps, RR finds regulatory overlaps, SI tests whether anyone would act on it)
3. **SF's absorption** — how it rebuilds the position to address challenges without losing structural coherence
4. **The verdicts** — HOLD means "I still have structural objections." RELEASE means "I can't break this." Convergence is not agreement.
5. **The final report** — compression (one board sentence), first move (Monday-morning action), residual risks (what's still dangerous even after convergence)

## Adapting the personas

The personas are domain-specific to AI governance in regulated financial services. If your domain is different, the pattern still works — you just need different lenses:

- **SF** → replace with your domain's architectural thinker
- **SC** → the four analytical frames (Ackoff, Beer, Feynman, Munger) are domain-agnostic — keep them
- **SI** → replace the audience archetypes (CTO, CRO, etc.) with your audience
- **Practitioner** → the capability/maturity/partial-adoption questions are domain-agnostic — keep them
- **RR** → replace the regulatory frameworks with your domain's regulatory context (or drop this persona entirely if regulation isn't relevant)

The structural pattern — opener, parallel challenge, absorption, re-evaluation, synthesis — is the reusable primitive. The personas are the domain-specific configuration.
