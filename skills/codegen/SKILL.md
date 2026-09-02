---
name: codegen
description: Regenerate repository-produced source metadata or generated code when the documented triggers or generated artifacts are stale.
---

# Code Generation

Read the host repository's `AGENTS.md` before using this skill. It defines the generator,
its trigger conditions, cache or state files, command, generated-output locations, and
the follow-up build required for that project.

Run the documented regeneration procedure only when its documented triggers apply. Avoid
editing generated files by hand unless the host repository explicitly designates them as
maintained source. Follow regeneration with the documented incremental build and report
any generated changes separately.
