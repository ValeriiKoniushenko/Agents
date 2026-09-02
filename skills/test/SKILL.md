---
name: test
description: Build and run the repository's unit-test suite after behavior changes or when the user asks to verify functionality.
---

# Test

Build and run the repository's unit-test suite. Use this skill after changes that affect
behavior, when adding or updating tests, or when the user asks whether the code works.

## Prerequisites

- Read `AGENTS.md` and relevant guidance under `.agents/` for the repository's test target,
  build layout, and required setup.
- Use an existing configured CMake build directory, normally `build/`.

## Build the test target

Build the documented test target, or discover its name from the CMake files:

```sh
cmake --build <build_dir> --parallel --target <test-target>
```

If the repository does not define a separate test target, use the normal incremental build
procedure before running the test executable.

## Run the tests

Run the documented test executable from `<build_dir>/bin/`. For this repository's concrete
test target and executable, see `AGENTS.md`.

```sh
<build_dir>/bin/<test-executable>
```

If CTest tests are registered, prefer the configured CTest command with failure output enabled:

```sh
ctest --test-dir <build_dir> --output-on-failure
```

Use the test framework's filtering options only for focused iteration. Run the complete suite
before reporting a behavior change as verified. Report build failures, test failures, and
skipped or unavailable tests explicitly.
