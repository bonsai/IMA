# Adr0016LayeredDictionaryPartitioning

## Status
Accepted

## Context
A single monolithic dictionary file is difficult to update without full redeployment and makes it hard to separate system data from private user data.

## Decision
Partition the dictionary system into three logical and physical layers:
1.  **System Layer**: Static, immutable global vocabulary (Read-only binary).
2.  **Domain Layer**: Context-specific vocabulary (e.g., Medical, IT) loaded as plugins.
3.  **Personal Delta Layer**: User-specific words and habits stored in a local, mutable database (SQLite or KV-store).

## Rationale
- **Maintainability**: System updates don't wipe out user data.
- **Privacy**: Personal data is isolated and can be easily deleted or encrypted.
- **Performance**: High-frequency system words can be optimized differently from user-added words.

## Consequences
- Requires a "Virtual Dictionary" layer that aggregates lookups across all active partitions.
- Priority management: Personal Delta must generally take precedence over System Layer.
