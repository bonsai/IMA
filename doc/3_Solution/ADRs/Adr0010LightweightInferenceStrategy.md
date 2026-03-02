# Adr0010LightweightInferenceStrategy

## Status
Proposed

## Context
Running sentiment and intent classification on local devices (Windows, Android, Browser) requires minimal overhead and high performance (<10ms).

## Decision
Use small, quantized statistical models instead of large neural networks:
- N-gram language models for sequence probability.
- Logistic Regression or small Decision Trees for fast intent classification.
- Targeted model size: 30MB-50MB.

## Rationale
- High-speed inference on CPU without specialized hardware.
- Small binary footprint suitable for browser extensions and mobile apps.
- Sufficient accuracy for intent classification (CLI vs. Natural Language).

## Consequences
- Requires careful feature engineering (e.g., symbol ratios, specific token presence).
- Limitation in complex semantic understanding compared to LLMs.
