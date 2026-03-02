# Adr0007CommandDiscovery

## Status
Proposed

## Context
Identifying CLI/Code intent is a primary goal to prevent accidental JP conversion.

## Decision
Use structural pattern recognition for Command Line (CL) detection:
- Hyphen flags (`--`, `-m`).
- Path separators (`/`, `\`).
- Common vocabulary (`git`, `npm`, `cd`).
- High symbol-to-text ratio.

## Rationale
- Rules are faster than ML models for deterministic patterns.
- Context window can boost accuracy (previous command was CLI -> current is likely CLI).

## Consequences
- Requires a curated "Command Dictionary".
- Needs special handling for mixed inputs (e.g., `git commit -m "日本語本文"`).
