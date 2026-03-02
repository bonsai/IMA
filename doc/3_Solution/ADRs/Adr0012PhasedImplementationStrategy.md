# Adr0012PhasedImplementationStrategy

## Status
Accepted

## Context
Building a complete Smart IME is a high-risk, complex task across multiple platforms.

## Decision
Implement in three distinct, functional phases:
- **Phase 1 (Basic Intent Router)**: Focusing on the "Post-Detection" core and high-speed CLI/JP classification.
- **Phase 2 (Contextual Assistance)**: Introducing contextual probability (prior state) and basic typo correction.
- **Phase 3 (Semantic Completion)**: Full morphological integration and personal habits learning.

## Rationale
- Allows early testing of the "Raw Romaji Principle".
- Minimizes the initial complexity of the cross-platform Rust logic.
- Ensures the "Probability Engine" is stable before adding complex NLP layers.

## Consequences
- Early versions will lack deep semantic features but will solve the "Mode Switch" pain immediately.
