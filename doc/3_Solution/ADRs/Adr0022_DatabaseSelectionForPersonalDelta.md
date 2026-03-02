# Adr0022: Database Selection for Personal Delta

## Status

Proposed

## Context

The SmartIME architecture requires a persistent storage solution for the "Personal Delta" layer, as defined in [ADR0016](Adr0016LayeredDictionaryPartitioning.md) and [ADR0008](Adr0008LocalLearning.md). This component is responsible for storing user-specific data, including custom vocabulary, correction history, and learned typing habits (e.g., n-gram frequency, typo patterns). The solution must be embedded, offline-first, fast, reliable, and accessible from the core Rust engine across multiple platforms.

## Decision

We will adopt a hybrid database strategy:

1.  **Primary Storage (Personal Delta):** **SQLite** will be used as the primary database for the Personal Delta layer. The `rusqlite` crate will provide a safe and ergonomic interface from the Rust core.
2.  **High-Speed Caching (Volatile Data):** **redb**, a pure-Rust key-value store, will be used for caching frequently accessed, temporary data, such as real-time n-gram models or session-specific context.

### SQLite for the Personal Delta

SQLite is chosen for its unparalleled robustness, cross-platform support, and ACID compliance. It will store structured and relational data such as:
- User-defined dictionary entries (word, reading, part-of-speech).
- Correction history (`before` -> `after` mappings).
- Learned statistical models and probability weightings.

### redb for Caching

`redb` is chosen for its high performance in key-value operations and its pure-Rust implementation, which simplifies the build process. It will be used for:
- Caching the results of expensive calculations.
- Storing session-specific, non-critical data that can be rebuilt if lost.

## Rationale

- **Reliability and Maturity (SQLite)**: SQLite is the most widely deployed embedded database in the world. Its stability and data integrity are proven, which is critical for storing long-term user data that must not be corrupted.
- **Rich Querying Capabilities (SQLite)**: While much of the data is key-value, the ability to perform complex SQL queries will be invaluable for future features, such as analytics on user habits, advanced dictionary management, and complex data migration.
- **Performance (redb)**: `redb` is inspired by LMDB and offers extremely fast read and write performance for simple key-value lookups, making it ideal for performance-critical caching scenarios within the 10ms latency budget.
- **Pure Rust Advantage (redb)**: As a pure-Rust library, `redb` avoids the complexities of linking against external C libraries, simplifying compilation, especially for targets like WASM.
- **Ecosystem and Tooling**: Both `rusqlite` and `redb` are well-maintained crates with strong community support. SQLite also benefits from a vast ecosystem of external tools for debugging and database management.

This hybrid approach allows us to leverage the best of both worlds: the industrial-strength reliability of SQLite for essential user data and the raw speed of a pure-Rust KV-store for performance-critical caching.

## Consequences

- **Two Database Dependencies**: The project will now depend on two separate database crates (`rusqlite` and `redb`), which adds a small amount of complexity to dependency management.
- **Data Synchronization**: A clear strategy must be defined for how and when data moves between the `redb` cache and the persistent SQLite database to ensure consistency.
- **SQLite Build Requirements**: `rusqlite` depends on the native SQLite library (`libsqlite3`), which must be available on the build environment for all target platforms. This is a standard component on most systems but needs to be managed in the build process. `redb` has no such external dependency.
