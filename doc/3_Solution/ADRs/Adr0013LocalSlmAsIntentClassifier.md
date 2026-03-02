# Adr0013LocalSlmAsIntentClassifier

## Status
Accepted

## Context
Traditional IMEs use manual mode switches. SmartIME needs an automated, low-latency way to determine if an input is Japanese, English, or a CLI command before any conversion happens.

## Decision
Implement a multi-layered Local SLM (Small Language Model) as the primary Intent Classifier.
- Layer 1: Heuristic filters for clear patterns (CLI/Paths).
- Layer 2: Character N-gram SLM for phonetic probability scoring (Romaji-to-Kana feasibility).
- Layer 3: Context-aware tiny transformer for ambiguous sequence disambiguation.

## Rationale
- Pure rule-based systems fail with mixed inputs like `git commit -m "修正"`.
- Statistical models provide a "Confidence Score" allowing the system to defer to candidates rather than forcing a wrong conversion.
- Decouples "Intent Recognition" from "Dictionary Conversion," enabling cleaner architecture.

## Consequences
- Requires a low-latency "streaming" scoring engine in the Rust core.
- Intent must be re-evaluated on every keystroke or after a small cooldown to ensure responsiveness.
- Performance cost is cumulative (Heuristic + N-gram + Transformer).
