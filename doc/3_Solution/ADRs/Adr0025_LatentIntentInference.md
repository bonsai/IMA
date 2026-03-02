# Adr0025: Latent Intent Inference

## Status

Proposed

## Author

Tsuji

## Context

The current architecture is designed to classify the explicit intent of an input stream (e.g., distinguishing Japanese from a CLI command). However, user input often contains a secondary, or "latent," intent that is not explicitly stated. For example, typing a product name might imply a desire to search for it online, or typing an address might imply a desire to open a map. To be a more effective "intent router," the system should be able to infer these potential next actions.

## Decision

We will introduce a **Latent Intent Inference** module that operates as a secondary processor, running *after* the primary text block has been classified and committed. This module will not perform actions automatically but will present actionable suggestions to the user.

1.  **Pattern-Based Entity Recognition**: The primary mechanism will be a highly optimized, rule-based entity recognizer that scans the committed text for common patterns. This includes regular expressions for entities like URLs, dates, tracking numbers, ISBNs, and dictionary-based lookups for locations, product names, or specific commands.

2.  **Secondary Action Mapping**: Each recognized entity pattern will be mapped to a potential secondary action (e.g., `URL` -> `ACTION_OPEN_BROWSER`, `Location` -> `ACTION_OPEN_MAP`). This mapping will be user-configurable.

3.  **Non-Intrusive Suggestions**: The inferred action will be presented as a small, passive, and actionable UI element (e.g., a clickable icon or a subtle button) adjacent to the committed text. This respects the principle of user sovereignty, as the user must explicitly choose to trigger the action.

## Rationale

-   **Enhanced Utility Without Intrusion**: This approach transforms the IME from a simple input tool into a lightweight command palette, proactively offering shortcuts for common next steps. By making suggestions optional and non-intrusive, it enhances utility without disrupting the user's flow or performing unwanted actions.
-   **Performance and Simplicity**: Relying on optimized pattern matching (regex and Trie lookups) as the core mechanism ensures that this feature adds negligible performance overhead. It avoids the need for heavy, probabilistic models for this secondary task, adhering to the project's lightweight constraints.
-   **High Extensibility**: The system is highly extensible. New intents can be added by simply defining a new pattern and mapping it to an action in a configuration file. This allows for easy customization and expansion without altering the core Rust engine.

## Consequences

-   **Scope Limitation**: This is a heuristic-based system, not a true AI assistant. It cannot understand complex, multi-step, or novel intents. Its capabilities are limited to the predefined patterns.
-   **Risk of False Positives**: The pattern matching may occasionally misidentify text as an entity, leading to irrelevant suggestions. The UI design must make it easy for the user to ignore or dismiss these suggestions effortlessly.
-   **Configuration Management**: A system will be needed for users to manage and customize the entity patterns and their associated actions, which adds a minor complexity to the C# configuration UI (ADR0021).
