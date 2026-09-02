---
name: code-coverage
description: Generate a project's configured code-coverage report when coverage validation is requested.
---

# Code Coverage

Read `AGENTS.md` first for the supported compilers, required CMake options, coverage
target, configured build directory, output location, and tool prerequisites.

Coverage instrumentation affects compilation. Use a dedicated coverage build directory
unless the project explicitly documents a safe reuse strategy. Configure the documented
options, build the documented coverage target, and verify the expected report exists
before reporting success.

Do not run coverage concurrently against the same build directory when the coverage tool
rewrites profiling or report output. Report unsupported toolchains, missing tools, disabled
tests, absent targets, and report-generation failures explicitly.
