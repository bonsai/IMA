# Adr0015HybridDictionaryAndModelStrategy

## Status
Proposed

## Context
For post-detection, we need both high-speed lookup (is this a known command?) and probabilistic estimation (does this look like a Japanese sentence?). The choice is between a pure Dictionary (Trie/HashMap) and a learned Neural Model (Transformer/RNN).

## Decision
Adopt a **Hybrid Strategy**:
1.  **Fast Path (Trie-based Dictionary)**: For deterministic lookup (CLI commands, common particles, fixed phrases).
2.  **Probabilistic Path (Lightweight Model)**: For unknown sequences, typo-correction, and intent-scoring when the exact string isn't in the dictionary.
3.  **Metabolism (Automatic Corpus Updates)**: The corpus and dictionary undergo "metabolism"—they are not static. New user habits are first cached in the Dictionary (Trie), then periodically used to fine-tune the local lightweight model (Personal Delta).

## Rationale
- **Speed**: Dictionaries are $O(k)$ for exact matches, much faster and lighter than running a model for every keystroke.
- **Flexibility**: Models (SLM/Tiny Transformer) can handle "unseen" patterns and typos that a strict dictionary would miss.
- **Corpus Metabolism**: Users' language changes. A hybrid approach allows the system to remain snappy (Dict) while evolving its understanding (Model).

## Consequences
- Requires a management layer to sync the Dictionary and the Model.
- Training/Fine-tuning must be done background-only to avoid UI freezing.
- Increased binary complexity to support both Trie-engines and Inference-engines.
