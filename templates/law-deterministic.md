---
id: <generate-uuid>
name: <Human-Readable Name>
evaluation: deterministic
applies-to: [all]    # or a list of specific formats: [proposal, memo, report]
---

## What it checks

<!-- One sentence. What property must the artefact have? -->

## Why it matters

<!-- Why does failing this check indicate a real problem — not just a style preference?
     This is what makes the law tunable: when the "why" stops being true, retire the law. -->

## What passing looks like

<!-- The positive case. Be specific enough that a script can verify it. -->

## What failing looks like

<!-- Concrete examples of artefacts that fail this check. -->

## Check configuration

```yaml
# Machine-readable config consumed by the check script.
# The script reads this section — no hardcoded values elsewhere.
# Examples: blocked word lists, character limits, regex patterns, required fields.
```
