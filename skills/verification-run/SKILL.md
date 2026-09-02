---
name: verification-run
description: Verify a built repository change by running its documented test suite and any required runtime smoke check.
---

# Verification Run

Read the host repository's `AGENTS.md` before using this skill. It defines the required
test target, test executable, runtime smoke-check executable, arguments, timeout, and
pass criteria for that project.

Build first using the `build` skill. Then run every documented verification command. A
change is verified only when every required command exits successfully without the
documented failure output; report skipped or unavailable checks explicitly.
