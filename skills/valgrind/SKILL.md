---
name: valgrind
description: Run a project's test suite under Valgrind to investigate memory errors or definite leaks.
---

# Valgrind

Use this skill after memory-sensitive changes, for nondeterministic test failures, or when
the user requests a memory check. It complements normal builds and tests; it does not
replace them.

Read `AGENTS.md` first for the supported build configuration, test executable, project
wrapper command, filters, and CI-aligned settings. Build the test executable with suitable
debug information, confirm Valgrind is available, then run the documented check.

Treat invalid accesses, uninitialized reads, and definite leaks as failures. Do not add a
suppression merely because a report involves a dependency; first determine whether project
code triggered it. Report unavailable tools, test failures, and infrastructure failures
explicitly.
