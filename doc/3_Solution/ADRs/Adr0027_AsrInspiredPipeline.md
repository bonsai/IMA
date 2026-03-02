# Adr0027: ASR-Inspired Text Input Pipeline

## Status

Proposed

## Author

Tsuji

## Context

The foundational idea of SmartIME is to treat text input like modern Automatic Speech Recognition (ASR) systems handle audio. Traditional IMEs operate on a stateful, pre-determined model ("What mode am I in?"), while ASR systems operate on a stateless, post-processing model ("What was most likely said?"). This ADR formalizes the decision to explicitly model the core input pipeline of SmartIME on the principles of ASR processing, as first conceptualized in the project's initial sketches and `PollinationIdeas.md`.

## Decision

We will structure the core input pipeline in the Rust engine to mirror the key stages of an ASR system: Signal Processing, Language Identification, Decoding, and Rescoring.

1.  **Signal Processing (Keystroke → Raw Romaji Buffer)**: Just as ASR digitizes audio into a waveform, SmartIME will accept all keystrokes as a raw, uninterpreted stream, buffering them into a simple Romaji string. This is the "Raw Input Principle" and treats the keyboard as a signal source, not a state machine.

2.  **Language/Intent Identification (Discriminator)**: ASR systems run a fast Language Identification (LID) model to determine the language of the audio. Similarly, SmartIME's `Discriminator` (ADR0014) acts as an "Intent ID" model, using N-grams and heuristics to rapidly classify the raw Romaji buffer as `CLI`, `Japanese`, `English`, etc. This is a direct parallel.

3.  **Decoding (Processor)**: After identifying the language, an ASR system uses a large acoustic and language model to decode the audio into the most likely sequence of words (a lattice of possibilities). SmartIME's `Processor` layer performs the equivalent function: based on the intent identified, it invokes the appropriate specialized model (e.g., Vibrato for Japanese, a CLI parser for commands) to convert the Romaji buffer into a structured output.

4.  **Rescoring & Formatting (Final Output)**: ASR systems often perform a final pass to add punctuation, capitalization, and correct grammatical errors based on a larger language model. The final stage of the SmartIME pipeline will do the same, applying typo corrections, personal habit adaptations (from the Personal Delta), and formatting before rendering the final text.

## Rationale

-   **Proven Model for Ambiguity**: The ASR pipeline is a battle-tested architecture for resolving ambiguity in a noisy, continuous input stream. By adopting this model, we inherit a robust, logical framework for handling the inherent ambiguity of mode-less text input.
-   **Conceptual Clarity**: This analogy provides a powerful and clear mental model for the entire development team. It clarifies why we separate the `Discriminator` from the `Processor` and why we treat input as a probabilistic stream rather than a series of deterministic commands.
-   **Future-Proofing**: This architecture is extensible. Just as ASR systems can add new languages, SmartIME can add new "intents" (e.g., Markdown, SQL, calculations) by adding a new classifier to the Discriminator and a new specialized Processor, without altering the fundamental pipeline structure.

## Consequences

-   **Shift in Mindset**: This requires developers to stop thinking in terms of traditional IME state machines and fully embrace the probabilistic, stream-processing paradigm.
-   **Emphasis on the Discriminator**: The success of the entire pipeline hinges on the speed and accuracy of the initial `Discriminator` stage, just as LID is critical for ASR. Significant effort must be invested in making this component as efficient and reliable as possible.
-   **Lattice Generation**: For more advanced features like suggesting multiple alternative corrections for a whole sentence, the `Processor` stage may need to evolve to output a lattice of possibilities (a graph of potential word sequences) rather than a single string, which adds complexity.
