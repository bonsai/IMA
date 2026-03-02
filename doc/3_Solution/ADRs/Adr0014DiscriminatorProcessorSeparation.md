# Adr0014DiscriminatorProcessorSeparation

## Status
Accepted

## Context
Handling mixed-intent inputs (like Japanese sentences embedded in CLI commands) requires a complex pipeline. Attempting to handle both detection and transformation in a single model increases complexity and latency.

## Decision
Separate the processing into two distinct specialized layers:
1. **The Discriminator (判別モデル)**: A lightweight router that segments the input stream and assigns intent labels (JP, CLI, etc.).
2. **The Processor (処理モデル)**: Specialized handlers for each intent that perform the actual transformation (Conversion, Completion, Correction).

## Rationale
- **Efficiency**: The Discriminator can be extremely fast (1-2ms), allowing real-time feedback, while heavier Processors only run on committed blocks.
- **Robustness**: Segmentation allows different parts of a single line to be processed by different logic.
- **Maintainability**: New intent types (e.g., Markdown, Math) can be added by updating the Discriminator and providing a new specialized Processor.

## Consequences
- Requires a stable "Streaming Segmenter" that can find boundaries in raw romaji.
- State management across multiple specialized processors must be handled by the core Rust engine.
