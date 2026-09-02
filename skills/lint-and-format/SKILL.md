---
name: lint-and-format
description: Run clang-format and clang-tidy locally on changed C++ files. Use before opening a PR, after finishing a change, or when the user asks to check code style or lint issues.
---

# Lint & Format

Run the repository's configured formatting and static-analysis checks locally. Prefer the
same versions, options, and file scope used by the project's CI when those are documented.
For this repository's CI helpers and runtime test executable, see `AGENTS.md`.

## Format check

```sh
git diff --name-only --diff-filter=ACMR -z -- \
  'sources/**/*.cpp' 'sources/**/*.h' 'sources/**/*.hpp' \
  'tests/**/*.cpp' 'tests/**/*.h' 'tests/**/*.hpp' \
  'benchmarks/**/*.cpp' 'benchmarks/**/*.h' 'benchmarks/**/*.hpp' \
  | xargs -0 -r clang-format --dry-run --Werror
```

## Format fix

```sh
git diff --name-only --diff-filter=ACMR -z -- \
  'sources/**/*.cpp' 'sources/**/*.h' 'sources/**/*.hpp' \
  'tests/**/*.cpp' 'tests/**/*.h' 'tests/**/*.hpp' \
  'benchmarks/**/*.cpp' 'benchmarks/**/*.h' 'benchmarks/**/*.hpp' \
  | xargs -0 -r clang-format -i
```

## Tidy check

Auto-detect the active build directory by finding the most recently modified
`compile_commands.json` under `build/`:

```sh
BUILD_DIR=$(find build -mindepth 1 -maxdepth 3 -name compile_commands.json \
-printf '%T@ %h\n' 2>/dev/null | sort -rn | head -1 | cut -d' ' -f2-)

if [ -z "$BUILD_DIR" ]; then
  echo "No compile_commands.json found under build/ — run cmake configure first."
  exit 1
fi

run-clang-tidy -p "$BUILD_DIR" -j "$(nproc)" \
  'sources/.*\.(cpp|cxx|cc)$' \
  'tests/.*\.(cpp|cxx|cc)$' \
  'benchmarks/.*\.(cpp|cxx|cc)$'
```

If the platform does not provide `nproc`, replace it with the local CPU-count
command or a conservative fixed value.

## Optional runtime check

Run the repository's test executable from the configured build directory when a
runtime memory check is needed:

```sh
valgrind --leak-check=full --show-leak-kinds=definite --track-origins=yes \
  <build_dir>/bin/<test-executable>
```

## Notes

- Only formats changed C++ files and analyzes the configured source trees by default.
  Full-tree analysis may be slow; use it when explicitly requested or required by CI.
- Run this before `git push`, not before every local build.
- Set `BUILD_DIR` explicitly when more than one configured build directory exists.
