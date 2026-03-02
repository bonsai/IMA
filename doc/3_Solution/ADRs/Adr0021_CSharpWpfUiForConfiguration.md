# Adr0021: C# UI for Configuration

## Status

Proposed

## Context

The core IME engine is built in Rust for maximum performance, low latency, and cross-platform compatibility, as established in [ADR0001](Adr0001CoreLanguageChoice.md). However, the project also requires a user-friendly and feature-rich configuration interface, especially for the Windows platform. Developing complex UIs in Rust can be time-consuming, and the ecosystem for native GUI development is less mature than in other languages. We need a solution that leverages a robust UI framework without compromising the performance of the Rust core.

## Decision

We will use **C# with WPF (Windows Presentation Foundation)** to build the configuration UI and other management tools for the Windows version of SmartIME. The C# application will run as a separate process and interact with the Rust core engine via a well-defined Foreign Function Interface (FFI).

- **UI Layer (C#)**: Handles all user interactions, settings display, dictionary management, and visualization of personal learning data.
- **Core Logic (Rust)**: Remains the single source of truth for all IME operations, classification, and learning, compiled as a native DLL (`.dll`).
- **Interop Layer**: C# will call Rust functions using **P/Invoke**. The `csbindgen` tool will be utilized to automatically generate the required C# `DllImport` boilerplate from the Rust code, ensuring type safety and reducing manual effort.

## Rationale

- **Productivity and Rich UI**: WPF is a mature and powerful framework for building modern, complex, and visually rich desktop applications on Windows. It provides a vast library of controls, data binding, and styling capabilities, significantly accelerating UI development compared to available Rust options.
- **Separation of Concerns**: This architecture cleanly separates the high-performance, real-time IME logic (Rust) from the non-real-time configuration and management tasks (C#). This prevents the UI from ever blocking or impacting the core typing experience.
- **Existing Expertise**: The C# and .NET ecosystem is widely adopted, and finding developers with expertise in WPF is straightforward.
- **Safe Interoperability**: Tools like `csbindgen` automate the creation of the FFI bridge, minimizing the risk of manual errors in data marshalling and function signatures between C# and Rust.

## Consequences

- **Windows-Specific UI**: The WPF-based UI will be specific to Windows. If a configuration UI is needed for other platforms (like a potential Linux version), a different UI toolkit (e.g., GTK-rs) or a web-based UI would be required.
- **Increased Deployment Complexity**: The final product will consist of the main Rust IME service and a separate .NET-based executable for the settings panel, which must be packaged together in the installer.
- **Interop Overhead**: While minimal for configuration tasks, there is a slight performance overhead associated with P/Invoke calls. This is not a concern for user-initiated actions in a settings panel but makes this approach unsuitable for the real-time input path. The core input loop remains purely within Rust and the TSF adapter.
