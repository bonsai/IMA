# Adr0004MorphologicalEngineSelection

## Status
Proposed

## Context
High-accuracy Japanese natural language detection needs structural analysis.

## Decision
Use MeCab as the primary morphological analyzer (via Rust FFI) for probability scoring rather than just conversion.

## Rationale
- High speed (C++ implementation).
- Reliable offline dictionary support (IPA/UniDic).
- Proven track record in Japanese NLP.
- Used as a "Probability Evaluator" to detect particles (ha, ga, wo) and sentence endings (desu, masu).

## Consequences
- Windows builds require native toolchain integration.
- Android NDK overhead.
- Rust FFI complexity.
