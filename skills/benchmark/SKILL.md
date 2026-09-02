---
name: benchmark
description: Build and run Google Benchmark performance measurements for performance-sensitive changes or when the user requests benchmark results or a performance-regression check.
---

# Benchmarking

Use this skill for performance-sensitive changes and for requests involving benchmark
results or performance-regression checks. Do not use it for ordinary unit-test or
compilation-only work.

Benchmark sources and their CMake targets are in the repository's `benchmarks/`
directory. Follow the repository's dependency and CMake configuration rather than
assuming a particular dependency path or target name.
For this repository's concrete benchmark executables and Release configuration, see
`AGENTS.md`.

## Prepare a comparable run

1. Configure a Release build with benchmarks enabled. Do not use Debug results for
   performance conclusions. Enable the benchmark option documented by the project's
   CMake files or contributor instructions.

   ```sh
   cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
   ```

2. Build incrementally using the repository's build procedure:

   ```sh
   cmake --build build --parallel
   ```

For before/after comparisons, use the same machine, build directory configuration, and
benchmark options. Close CPU-intensive applications and avoid comparing measurements from
different machines or power modes.

## Run and inspect results

Discover the benchmark executables produced in the configured build directory,
normally under `build/bin/`, then run the relevant targets. To inspect a target's
registered benchmarks without measuring them:

```sh
<build_dir>/bin/<benchmark-executable> --benchmark_list_tests
```

Run the selected benchmark executable or executables after discovery:

```sh
<build_dir>/bin/<benchmark-executable>
```

Run a focused group while iterating with Google Benchmark's filter option:

```sh
<build_dir>/bin/<benchmark-executable> --benchmark_filter='<regular-expression>'
```

For a comparison-quality report, take repeated samples and write JSON output outside the
source tree or in an ignored build directory:

```sh
<build_dir>/bin/<benchmark-executable> \
  --benchmark_repetitions=10 \
  --benchmark_report_aggregates_only=true \
  --benchmark_format=json \
  --benchmark_out=<build_dir>/benchmark-results.json
```

Compare the aggregate `real_time` values for the same benchmark names and argument sets.
Treat small differences as noise unless repetitions show a consistent change; report the
command, build type, and whether the result is CPU-time or real-time based.
