# Sequences + Laws Starter Kit

A pattern for building governed evaluation pipelines that run inside Claude Cowork. This kit gives you the structural primitives, design rationale, and copy-paste-able templates to build your own.

## What's in here

| File | What it is |
|------|-----------|
| `guide.md` | The full pattern: what sequences and laws are, why each piece exists, design decisions, smells to watch |
| `templates/law-deterministic.md` | Template for a deterministic law (script-checkable, no judgment) |
| `templates/law-interpretive.md` | Template for an interpretive law (agent-evaluated, requires judgment) |
| `templates/evaluator-skill.md` | Template SKILL.md for an interpretive evaluation stage |
| `templates/subscribed-laws.yaml` | Template for mapping laws to stages |
| `example/` | A worked example: "client deliverable gate" — a simple 2-stage sequence that isn't the publication pipeline |

## How to use this

1. Read `guide.md` first. It's ~15 minutes. It explains the pattern and the design decisions behind each piece.
2. Decide your stages (what questions your pipeline answers, in what order).
3. Write your laws using the templates.
4. Write your evaluator skills using the template.
5. Wire them together with `subscribed-laws.yaml`.
6. The runner script is the last thing you build — get the laws and skills right first.

## What this is NOT

This is not a framework to adopt wholesale. It's a structural pattern. Take the pieces that solve your problem, leave the rest. If you only need laws without a runner, that's fine. If you need a single-stage gate, that's fine. The pattern scales down as well as up.
