---
name: build
description: Standard incremental CMake build of the repository. Use when the user asks to build, compile, or check that changes compile.
---

# Build

Perform an incremental build using the existing configured build directory. Reuse the
build cache and rebuild only what changed.

## Command

```sh
cmake --build build --parallel
```

## Prerequisites

- A configured CMake build directory must exist, normally `build/`. If it does not,
  configure it according to the repository's build instructions; configuration is
  separate from this incremental-build step.
- Do not assume a compiler, generator, cache tool, build type, or nested build-directory
  layout unless the repository configuration specifies one.

## Resulting artifacts

Executables and other artifacts are produced according to the repository's CMake targets,
normally under `<build_dir>/bin/`. Discover target names from the build files or the build
output instead of assuming fixed executable names.
This repository's concrete targets, supported build layouts, and CMake options are listed
in `AGENTS.md`.

## Notes

- Don't suggest `--clean-first` here — that's the separate clean-build skill.
- If optional tests or benchmarks should be omitted, use the corresponding CMake options
  documented by the repository.
