# Adr0023: ML Strategy for Intent Classification

## Status

Proposed

## Context

The core functionality of SmartIME is its ability to classify user intent from a raw romaji input stream, distinguishing between Japanese, CLI commands, and other languages. This requires a machine learning (ML) strategy that is extremely fast (<10ms latency), runs entirely offline on the CPU, and has a minimal memory and binary footprint. This ADR defines the selection of the morphological analyzer and the inference engine to achieve these goals, building upon the principles in [ADR0010](Adr0010LightweightInferenceStrategy.md) and [ADR0013](Adr0013LocalSlmAsIntentClassifier.md).

## Decision

Our ML strategy will be based on a combination of a high-performance, pure-Rust morphological analyzer and a lightweight, pure-Rust inference engine.

1.  **Morphological Analyzer:** **Vibrato** will be adopted as the primary Japanese morphological analyzer. It is a pure-Rust, MeCab-compatible tokenizer known for its high performance.
2.  **Inference Engine:** **Tract** will be used as the inference engine for running our lightweight classification models (e.g., N-gram models, logistic regression classifiers) which will be exported to the ONNX format.
3.  **Model Strategy:** Instead of large language models, we will rely on small, highly optimized statistical models. The primary model will be a character-based N-gram model to calculate the "Japanese-ness" probability of the input string. These simple models are sufficient for the initial classification task and can be executed with extremely low latency by Tract.

## Rationale

- **Pure-Rust Ecosystem**: Both Vibrato and Tract are written entirely in Rust. This eliminates the need for C/C++ dependencies and FFI bridges for the core ML stack, which drastically simplifies the cross-platform build process (especially for Windows, Android, and WASM) and ensures memory safety.
- **Performance**: Benchmarks show that pure-Rust tokenizers like Vibrato and Lindera outperform traditional C++ based solutions. Vibrato, being a reimplementation of MeCab, is noted to be exceptionally fast. Tract is specifically designed for high-performance inference on CPUs, making it a perfect fit for our low-latency requirement.
- **Offline and Lightweight**: This stack is perfectly aligned with the strict offline constraint. The models are small statistical files, and the engines themselves have a minimal binary footprint, making the entire package suitable for deployment in constrained environments like browser extensions.
- **MeCab Compatibility (Vibrato)**: By using a MeCab-compatible engine, we can leverage the vast ecosystem of pre-trained dictionaries like IPADIC and UniDic, while benefiting from a modern, memory-safe, and faster implementation.
- **Standardized Inference (Tract & ONNX)**: Using Tract with the ONNX format decouples model training from inference. We can train or define our statistical models using any framework (e.g., Python's scikit-learn) and deploy them consistently across all platforms via the single Rust-based Tract engine.

## Consequences

- **Model Conversion**: All models must be converted to the ONNX format before they can be used by Tract. This adds a step to the model development workflow.
- **Limited Model Complexity**: While sufficient for intent classification, Tract and the lightweight model strategy are not intended for complex generative tasks. Future expansion into more advanced semantic understanding might require integrating a more powerful engine like `candle`, but this would be a separate, future architectural decision.
- **Vibrato Dictionary Management**: The dictionary for Vibrato needs to be compiled and bundled with the application, which will be part of the build process. The choice of dictionary (e.g., IPADIC vs. UniDic) will impact both accuracy and final binary size.
