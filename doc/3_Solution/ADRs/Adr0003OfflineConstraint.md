# Adr0003OfflineConstraint

## Status
Accepted

## Context
Privacy, speed, and availability are critical for an IME.

## Decision
The system must be fully offline:
- Model size limit: 30MB-50MB.
- Latency target: <10ms per classification.
- No external API calls.
- Pure CPU inference.

## Rationale
- Privacy (user data never leaves the device).
- Zero network dependency.
- Immediate response time.

## Consequences
- Requires lightweight statistical models (n-grams, small transformers).
- Quantization mandatory for any neural components.
