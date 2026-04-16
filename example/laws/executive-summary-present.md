---
id: b2c3d4e5-f6a7-8901-bcde-f12345678901
name: Executive Summary Present
evaluation: deterministic
applies-to: [proposal, report]
---

## What it checks

The deliverable begins with an executive summary that states the core recommendation.

## Why it matters

Senior stakeholders read the executive summary and nothing else. If the recommendation isn't in the first section, it doesn't exist for the decision-maker. An analysis without a leading recommendation forces the reader to do synthesis work that you should have done for them.

## What passing looks like

The first H2 section (or the content before the first H2 if using a title + lead structure) contains:
1. The word "summary" or "recommendation" in the heading (case-insensitive)
2. At least one declarative sentence that states what the client should do

## What failing looks like

- A deliverable that opens with "Background" or "Context" before stating the recommendation
- An executive summary that describes the problem without stating a position
- No identifiable summary section at all

## Check configuration

```yaml
# Look for a heading containing any of these terms in the first 500 characters
heading-terms:
  - summary
  - recommendation
  - overview

# The section under that heading must contain at least one sentence with an action verb
action-indicators:
  - "we recommend"
  - "our recommendation"
  - "the recommended"
  - "we propose"
  - "should"
  - "we advise"
```
