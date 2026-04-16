---
id: c3d4e5f6-a7b8-9012-cdef-123456789012
name: No Banned Jargon
evaluation: deterministic
applies-to: [all]
---

## What it checks

The deliverable does not contain consulting jargon that obscures meaning.

## Why it matters

Jargon substitutes for precision. "Leverage synergies" means nothing specific. "Combine the data engineering teams to eliminate the duplicated Kafka maintenance" means something a client can act on. If something is important, the writing describes what it changes structurally — the reader decides the magnitude. Jargon signals that the recommendation isn't clear enough to express in plain language.

## What passing looks like

Zero matches (case-insensitive) for any term in the blocked list.

## What failing looks like

"We recommend leveraging synergies between the platform and data teams to unlock value through a best-in-class transformation programme." Every italicised word is a failure.

## Check configuration

```yaml
# Case-insensitive substring matching.
# Extend this list as jargon is encountered in drafts.
blocked-terms:
  - synergy
  - synergies
  - leverage synergies
  - unlock value
  - best-in-class
  - world-class
  - paradigm shift
  - transformative
  - game-changing
  - game changing
  - cutting-edge
  - cutting edge
  - next-generation
  - move the needle
  - low-hanging fruit
  - boil the ocean
  - circle back
  - deep dive          # acceptable as a section title, not in body text
  - thought leader
  - thought leadership
  - north star
  - blue sky
```
