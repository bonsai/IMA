# Adr0026: Speaker Clustering for Contextual Adaptation

## Status

Proposed

## Author

Tsuji

## Context

A user's writing style, vocabulary, and intent can vary significantly depending on the context of the conversation or the application they are using. For example, the language used in a formal email to a client is different from the language used in a casual chat with a friend or when writing code comments. To provide truly relevant predictions and corrections, the IME must adapt not just to the user, but to the user's current "persona" or communication context.

## Decision

We will implement a **Speaker Clustering** mechanism within the Personal Delta layer. This system will not identify individual people but will create and maintain distinct statistical profiles, or "personas," representing the user's different communication styles. The system will automatically switch between these personas based on the application context.

1.  **Persona Creation**: The system will create separate statistical models (N-grams, word frequencies, typo patterns) for different contexts. Initially, these contexts will be defined by the active application (e.g., `VSCode`, `Slack`, `Outlook`).

2.  **Automatic Context Switching**: The platform adapter layer (ADR0009) will be responsible for detecting the active application's process name and notifying the Rust core. The core will then load the corresponding persona from the Personal Delta.

3.  **Unified but Weighted Models**: Instead of entirely separate models, we will maintain a single base Personal Delta and apply context-specific "weighting layers" on top of it. This is more memory-efficient and allows for shared learning across personas. For example, a typo learned in one context can be recognized in another, but vocabulary specific to `VSCode` will have a higher probability when that application is active.

## Rationale

-   **Dramatically Improved Contextual Relevance**: This is the most effective way to resolve the ambiguity where a single string could be a code snippet in one context and a casual phrase in another. By switching statistical models based on the application, the accuracy of predictions, corrections, and intent classification will be significantly improved.
-   **Memory Efficiency**: The use of weighting layers instead of completely separate models prevents a linear increase in memory usage as more personas are added. It allows the system to scale to dozens of application contexts without a significant memory footprint.
-   **Unsupervised and Private**: The clustering is based on application names, not the content of the communication or the identity of the other party. It is an unsupervised process that runs entirely offline, fully preserving user privacy.

## Consequences

-   **Platform Adapter Complexity**: The platform adapters (especially on Windows and macOS) must be enhanced to reliably identify the active application's process name and window title, which can sometimes be challenging.
-   **Initial Cold Start**: When a user uses a new application for the first time, the system will fall back to the default persona. The new persona will only become effective after a sufficient amount of interaction has been recorded for that context.
-   **UI for Persona Management**: The C# configuration UI will need to be extended to allow users to view, merge, reset, or manually assign personas to specific applications, adding to the management overhead.
