---
id: a1b2c3d4-e5f6-7890-abcd-ef1234567890
name: No Placeholder Text
evaluation: deterministic
applies-to: [all]
---

## What it checks

The deliverable contains no placeholder markers that indicate incomplete content.

## Why it matters

A placeholder in a client deliverable is not a minor oversight — it signals that the deliverable was assembled from a template and not fully completed. It erodes client trust instantly. One `[TBD]` in a proposal can undo weeks of relationship-building.

## What passing looks like

Zero matches for any marker in the blocked list, anywhere in the document body (including headers, footers, tables, and appendices).

## What failing looks like

Any occurrence of a placeholder marker. Examples: "As shown in [INSERT TABLE], the ROI is significant" or "Timeline: [TBD]".

## Check configuration

```yaml
# Case-insensitive substring matching.
blocked-markers:
  - "[TODO]"
  - "[TBD]"
  - "[INSERT"
  - "[PLACEHOLDER"
  - "[DRAFT]"
  - "[XX"
  - "[CLIENT NAME"
  - "lorem ipsum"
```
