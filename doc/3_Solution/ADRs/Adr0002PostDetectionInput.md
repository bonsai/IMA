# Adr0002PostDetectionInput

## Status
Accepted

## Context
Traditional IMEs require pre-switching modes (JP/EN), which breaks focus and cognitive flow.

## Decision
Adopt the Raw Romaji Principle:
- Accept all inputs as raw romaji.
- Classify intent (CLI, JP, EN, MIXED) after the Commit (Enter or Focus Out).
- Perform transformations post-classification.

## Rationale
- Mimics ASR (Speech) flows.
- Eliminates "What mode am I in?" cognitive load.
- Treats input as a "Probability Flow" rather than a "State Machine."

## Consequences
- Post-commit transformation latency must be <10ms.
- Input field state must handle raw romaji until committed.
