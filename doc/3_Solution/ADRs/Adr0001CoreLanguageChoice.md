# Adr0001CoreLanguageChoice

## Status
Accepted

## Context
Develop an offline Smart IME for Windows, Android, and Browser with low-latency classification (CL vs. JP Prompt) and post-input processing.

## Decision
Use Rust for the core language processing engine.
- Windows: C++ bridge (TSF/Win32)
- Android: Kotlin + JNI to Rust
- Browser: WebAssembly (WASM)

## Rationale
- Rust offers memory safety and high performance.
- Easy FFI for cross-platform core logic sharing.
- Low latency required for IME (no GC overhead).

## Consequences
- Single shared detection engine.
- Deterministic performance.
- JNI/TSF/WASM complexity.
- Consistent behavior across OS and Browser.
