# Adr0017MemoryMappedZeroCopyStorage

## Status
Accepted

## Context
Loading several megabytes of dictionary data into RAM on every startup or for every classification is slow and increases the memory footprint significantly, especially in browser extensions.

## Decision
Use **Memory-Mapped I/O (mmap)** for dictionary storage and access.
- Map the binary dictionary directly from disk to the process address space.
- Access data using zero-copy structured pointers.

## Rationale
- **Instant Startup**: No need to read the entire file into memory; the OS handles paging on demand.
- **Shared Memory**: Multiple instances (e.g., different browser tabs/processes) can share the same physical RAM for the read-only dictionary pages.
- **Low Memory Overhead**: Only the accessed parts of the dictionary occupy physical memory.

## Consequences
- Dict binary format must be "Fixed-Layout" (endian-aware and pointer-less).
- Requires platform-specific mmap handles (Windows `MapViewOfFile`, Unix `mmap`).
