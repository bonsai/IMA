# Adr0008LocalLearning

## Status
Proposed

## Context
Individual users have unique typing habits and vocabulary.

## Decision
Implement a "Personal Delta" system using local Trie and small N-gram models.
- Automatically learn from user corrections.
- Store habit compression data locally.

## Rationale
- Personalized experience without cloud syncing.
- "Habit Compression": IME adapts to the user, not vice versa.

## Consequences
- Local storage management for user-specific models.
- Reset/Management UI required for privacy controls.
