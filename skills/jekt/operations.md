---
modified: 2026-10-28
---

# General Operations

How Jekt operations are carried out when no procedure covers them.

## 1. Operating Context

Before acting, establish:

- **Repository root and `.jekt/`.** Operate against `.jekt/` in the default (primary) workspace root unless the request indicates another root or repository. With multiple workspace roots open, a ticket ID that resolves only in a non-default root is ambiguous: ask the user which root to operate against. Do not silently switch.
- **Active project slug** (when multiple coexist). Resolve from `.jekt/projects/`: if exactly one slug is declared (`.jekt/projects/{slug}/` exists), use it; if multiple slugs are declared, infer from the subject matter of the request, otherwise ask. If `.jekt/projects/` is absent or empty, fall back to inference from existing ticket directories under `.jekt/tickets/` (any `{slug}-{number}` form indicates the slug is in use); if still ambiguous, ask. Do not invent unstated heuristics; Jekt does not prescribe a default-project mechanism.
- **Missing or empty `.jekt/`.** If the request needs Jekt data and the chosen root has no `.jekt/`, note the absence and ask whether to (a) run the Initialize Jekt procedure, (b) operate against a different repository, or (c) treat the request as conversational. Do not invent paths or fabricate ticket data.
- **Conversational queries** (definitions, "what is …", general guidance) are answered from this skill regardless of `.jekt/` presence.

## 2. Context Assembly

| Request kind | Read |
|---|---|
| Queries about a ticket | description prose, `description.yaml`, `minutes.md`, most recent plan, relevant designs, recent comments |
| Code work on the current ticket | description prose, `description.yaml`, current plan, relevant designs |
| Cross-ticket discovery | `.jekt/tickets/` enumeration + queries (§3) |
| Completion review | description, minutes, designs, plans, commits in the ticket's range |

Read minutes first as a timeline index, then follow references into designs, plans, or comments as needed.

## 3. Queries

The filesystem is the source of truth. Do not maintain local indexes or caches; query fresh.

Search the files under `.jekt/` with the workspace's search tools, and the commit history with Git. Absent properties take their defaults when read, and what "open" means is defined in [Jekt Definitions](definitions.md). A search must match as follows:

- A value in `tags.lst` or `jekt/releases.lst` matches only a whole line, so `2.0.3` does not match `2.0.30`.
- Release membership is recorded only on tickets, so finding a release's tickets means searching the tickets.
- A ticket's commits carry its bracketed ID in the subject. Match it case-insensitively and exactly, so `foobar-123` does not match `foobar-1234`.

**Read-time vs. edit-time.** Text search is acceptable for read-time filtering. Structured edits (changing values in `description.yaml`, modifying `.lst` membership) must use structured tooling, not text substitution.

## 4. Changes

Read [Jekt Definitions](definitions.md) before any metadata change. A change preserves what it does not touch and tolerates conforming variations; see [Preservation vs. Tolerance](definitions.md#preservation-vs-tolerance).

## 5. Request Handling

- Classify each request as a **query** (read-only) or a **change** (mutating).
- **Queries are strictly read-only.** Do not produce or modify Jekt artifacts (summaries, minutes entries, comments, TODOs, descriptions) as a side effect of answering a query.
- For ambiguous requests, ask one focused clarifying question rather than guess.
- For non-trivial changes, summarize the intended action (paths to be created or modified) before executing.

## 6. Response Style

- Concise, factual, in Jekt's terminology.
- Render ticket references in bracketed form (`[foobar-123]`). Use backticks for paths, filenames, commands, property names, and literal values.
- When reporting a property value, distinguish *stored* from *derived* (e.g., the label, the heading title) only when the distinction matters to the user.
- Omit rationale, history, and alternatives unless asked.

## 7. Anti-Patterns

Do **not** fabricate Jekt conventions not specified in this skill.
