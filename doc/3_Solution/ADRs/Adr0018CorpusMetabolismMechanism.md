# Adr0018CorpusMetabolismMechanism

## Status
Proposed

## Context
User typing habits (corpus) change over time. A static dictionary becomes stale, while a purely dynamic model might forget stable patterns.

## Decision
Implement a **Corpus Metabolism** lifecycle:
1.  **Ingestion**: Real-time caching of new words/corrections into the "Short-term Memory" (Mutable Trie/Delta Layer).
2.  **Consolidation**: Background background process that analyzes high-frequency Delta words and periodically updates the SLM weighting.
3.  **Eviction**: Automatic pruning (purging) of low-usage personal vocabulary to maintain a "Healthy" and compact local database.

## Rationale
- Mimics biological "Metabolism" where the system self-optimizes for current usage.
- Prevents database bloating from one-off typos or temporary jargon.
- Keeps the inference engine aligned with the user's evolving language style.

## Consequences
- Background processing must not compete with the IME's 10ms real-time target.
- Requires logic to determine the "Decay Rate" of words.
