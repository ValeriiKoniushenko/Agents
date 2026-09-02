---
name: clean-build
description: Full clean CMake rebuild from scratch, bypassing build caches. Use when the user reports stale or inconsistent build state, after structural build changes, or when an incremental build fails inexplicably.
---

# Clean Build

Removes and recreates only the configured build output, then rebuilds from scratch. Use
only when an incremental build is untrustworthy; preserve source, test, documentation,
benchmark, and configuration directories.

## Command

```bash
rm -rf build
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build --parallel
```

Use the repository's documented generator, build type, optional-feature flags, and
dependency setup when they differ from this default. If dependencies are provided
through Git submodules and are not initialized, run `git submodule update --init
--recursive` before configuring. Do not update submodules to a remote or force their
contents as part of a clean build unless the user explicitly requests that operation.
For this repository's submodule synchronization procedure and build layouts, see
`AGENTS.md`.

## When to actually reach for this

- CMake build files changed structurally (new targets, changed compiler/linker flags)
- Switching between MSVC/GCC/Clang toolchains
- A compiler or build-cache problem leaves the incremental state unreliable
- "it builds in CI but not locally" type reports
