# Adr0024: Word and Phrase Prediction

## Status

Proposed

## Author

Tsuji

## Context

The current architecture focuses on post-input classification and correction. To further enhance typing efficiency and reduce keystrokes, the system should provide real-time word and phrase completion suggestions based on the user's personal typing habits and context. This capability must adhere to the strict sub-10ms latency and offline-first constraints.

## Decision

We will implement a real-time prediction engine that leverages the existing **Personal Delta** and **Dictionary** layers. The prediction mechanism will be primarily based on the N-gram statistical data stored in the high-speed `redb` cache and the Trie data structures.

1.  **Prediction Source**: The primary source for predictions will be the user-specific N-gram model (up to 3-gram) and the personalized vocabulary stored within the Personal Delta (SQLite/redb). The System and Domain dictionaries will serve as a fallback.
2.  **Engine**: The prediction engine will utilize the existing Double-Array Trie for extremely fast prefix searches (`O(k)`). As the user types, the engine will perform a continuous lookup on the Trie to find matching words and the N-gram cache to find probable next words.
3.  **UI**: Predictions will be displayed as unobtrusive, inline ghost text or in a small, passive suggestion box. The user can accept a suggestion with a single key press (e.g., `Tab`). Automatic completion will be strictly forbidden to uphold the principle of user sovereignty.

## Rationale

-   **Leverages Existing Architecture**: This approach requires no new, heavy dependencies. It builds directly upon the high-performance data structures already specified in ADR0011 (Trie), ADR0017 (mmap), and ADR0022 (redb), ensuring the latency budget is met.
-   **High Degree of Personalization**: By prioritizing the Personal Delta, the suggestions will be highly relevant to the individual user's vocabulary, jargon, and habitual phrases, making the feature genuinely useful.
-   **Performance**: Trie-based prefix searching is one of the fastest known methods for this task, guaranteeing that the prediction lookup will not introduce noticeable latency into the input loop.
-   **No Generative Models**: This approach avoids the complexity, resource consumption, and non-deterministic nature of large generative models, aligning with the project's core principles of being lightweight and offline.

## Consequences

-   **
Prediction quality will be directly proportional to the richness of the user's Personal Delta. New users will experience less effective predictions until the system has learned their habits.
-   **Limited Semantic Understanding**: The prediction is based on statistical co-occurrence (N-grams) and prefix matching, not deep semantic understanding. It cannot infer complex intent or suggest contextually appropriate but statistically rare words.
-   **UI/UX Challenge**: The user interface for displaying suggestions must be carefully designed to be helpful without being visually distracting or intrusive to the typing flow.
