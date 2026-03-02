# Adr0009PlatformAdapters

## Status
Accepted

## Context
Windows and Android have vastly different input APIs.

## Decision
Build separate native Adapters:
- Windows: TSF (Text Services Framework) with C++ Bridge.
- Android: IME Service using Kotlin + JNI.
- Browser: Web Extension (Manifest v3) + WASM.

## Rationale
- TSF is the modern standard for Windows but lacks high-level Rust wrappers.
- Android's `InputMethodService` is Java-native.
- Browser: Web Extension allows global keystroke handling or input-field-level interception.
- JNI/FFI/WASM allows all platforms to share the same core Rust engine (`SmartImeCore`).

## Consequences
- Native development skills required for both platforms.
- Debugging bridge calls can be complex.
