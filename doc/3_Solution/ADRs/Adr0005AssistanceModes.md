# Adr0005AssistanceModes

## Status
Proposed

## Context
Users want control over how much the AI interferes with their typing.

## Decision
Implement three Assistance Modes:
- Manual Mode: Traditional pre-select behavior.
- Assisted Mode: Post-commit classification + candidate display (no auto-confirm).
- Auto Mode: Auto-confirm high-probability transformations.

## Rationale
- User empowerment and control.
- Safety against "Over-Correction" (Forgiveness Principle).
- Gradual learning curve for the new paradigm.

## Consequences
- UI MUST clearly indicate the current mode or provide easy Hotkeys (e.g., Ctrl+R).
