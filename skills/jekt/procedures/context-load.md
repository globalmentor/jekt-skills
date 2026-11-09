---
modified: 2026-10-30
---

# Load Context

Read the following before continuing with the request, in order from general to specific. Operate against the default workspace root unless the request indicates another root. Skip a step whose target does not exist or does not apply; do not fabricate content for a file that is missing.

**Warning: verify glob-search completeness.** A workspace file search may return only a subset of a glob's matches and indicate the omission only with a trailing `...` or a reported total larger than the displayed list. This matters whenever the result is being used to decide whether a requested file exists. If the output is truncated, repeat the search with the maximum number of results raised to at least the reported total, or narrow the glob if the result set is large. Never treat a file as missing merely because it is absent from truncated output.

1. **Project declaration** — each project declared under `.jekt/projects/`, if any: its slug and its description.
2. **Project readme** — the readme at the repository root, for its purpose, scope, and conventions.
3. **Current ticket**, if any: the ticket's description, its `minutes.md`, any plan currently in progress under `plans/` (most recent by date), and any design documents under `designs/` relevant to the request.
