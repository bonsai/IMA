# Adr0006TypoTolerance

## Status
Proposed

## Context
Small typos (mother-vowel drop, consonant doubling) shouldn't break the experience.

## Decision
Apply Keyboard Proximity and Levenshtein Distance (<= 2) for typo correction.
- Prioritize distance based on QWERTY layout.
- Use phonetic similarity (かな音素) for JP natural language.

## Rationale
- "Forgiveness Principle": A software that understands human imperfection.
- Enhances speed by reducing "Backspace" events.

## Consequences
- May increase classification ambiguity.
- Potential performance cost of editing distance calculations.
