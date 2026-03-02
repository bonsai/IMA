# Adr0011DataStructuresForOfflineSearch

## Status
Proposed

## Context
Fast prefix matching and typo-tolerant search are essential for a smooth typing experience without network latency.

## Decision
Utilize highly efficient memory-resident data structures:
- **Trie (Prefix Tree)**: For fast auto-completion and command vocabulary lookup.
- **Bloom Filters**: For quick membership tests to skip unnecessary heavy processing.
- **Levenshtein Distance Automata**: For efficient typo-tolerant candidate generation.

## Rationale
- Tries provide O(k) lookup (k=key length), independent of dictionary size.
- Bloom filters prevent checking morphological engines for blatant CLI strings.
- Minimizes memory usage while maintaining high throughput.

## Consequences
- Dictionary compilation step required during build or update.
- Memory consumption depends on dictionary density.
