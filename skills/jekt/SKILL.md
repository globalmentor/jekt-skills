---
name: jekt
description: Crucial, foundational orientation for understanding a project and knowing how to locate and interpret information through semantic artifacts. Defines the Jekt context architecture and describes its organization of project context (e.g. directory layout, tickets, commits). Load this skill for any activity that relates to project status, designs, plans, work, work processes, or searches examining a project. This broad skill is critical for learning initial context about the project, discovering the current ticket being worked on, and knowing when to capture significant events in ticket minutes. It MUST be loaded if the user, prompt, or goal (the request) mentions "Jekt", "Git", "project", "ticket", "plan", "minutes", or "release"; if the request requires any lookup or knowledge discovery related to a project state or objectives; or the request involves the `.jekt/` directory tree. When a conversation that used this skill is summarized, reload it and follow its Load Context procedure, whatever the request.
license: Apache-2.0
metadata:
  modified: "2026-10-30"
---

This skill comes in two parts:

1. Always-loaded foundational understanding of the Jekt model and directory layout.
2. Instructions loaded as needed: procedures for specific activities and how Jekt operations are carried out in general.

# Jekt Ambient Instructions

Jekt is a context architecture for software projects. Its specification is licensed under CC BY 4.0; the Jekt skill is copyright © 2026 GlobalMentor, Inc. and licensed under Apache 2.0. "Jekt" and the Jekt logo are trademarks of GlobalMentor, Inc.

A repository uses **Jekt** if it contains a `.jekt/` directory at its root. Jekt stores tickets, plans, designs, minutes, and other semantic working context as plain files. If `.jekt/` is not present, Jekt is not in use here; do not derail unrelated work.

The sections below summarize Jekt data. [Jekt Definitions](definitions.md) gives its exact meaning and form, including identifiers, references and headings, file names, metadata, file formats, and the minutes format. Read it whenever the exact form or meaning matters, such as before writing metadata, reference definitions, headings, or file names, or when an answer depends on what a metadata value means.

## Current Ticket

The current ticket is derived from the active Git branch when it matches `tickets/{ticket-id}` (e.g., `tickets/foobar-123`). Ticket IDs have the form `{project-slug}-{number}`; the **last hyphen** separates the project slug from a decimal number. Identifiers are lowercase by default (`foobar-123`) and caseful; only CommonMark reference-label resolution is case-insensitive.

## References

Use CommonMark shortcut reference links: `[foobar-123]` for Jekt tickets in the current repository. Other reference families (external trackers such as Jira or GitHub) differ structurally; recognize them but do **not** fabricate cross-project or external references without explicit user instruction. A ticket's own ID needs no link definition inside its own files.

## Canonical Directory Map

```
.jekt/
├── tickets/{project-slug}-{number}/
│   ├── description - {Label}.md   # prose description
│   ├── description.yaml           # structured metadata
│   ├── minutes.md                 # append-only timeline
│   ├── summary.md                 # at completion
│   ├── todo - {Label}.md          # zero or more
│   ├── plans/{YYYY-MM-DD}-{slug}.md
│   ├── designs/{Label}.md
│   └── comments/{UTC-timestamp}.md
├── releases/{release-id}/
│   ├── description - {Label}.md
│   └── description.yaml
├── projects/{slug}/
│   ├── description - {Label}.md
│   └── description.yaml
└── todos/{Label}.md
```

All files are optional. Tickets, releases, and projects share the same description form: a prose file `description - {Label}.md` and a structured metadata file `description.yaml`. The **label** is a short Title-Case lexical handle (the trailing portion of the description filename); the **heading title** is the level-one heading text of the prose file. Ad-hoc files at an entity root are tolerated.

## Artifact Roles

- **Description** — scope and intent of an entity (ticket, release, or project); label from filename, heading title from the prose file's `#` heading.
- **`description.yaml`** — structured metadata for the containing entity; an empty file is the legal degree-zero state asserting existence with no properties yet recorded.
- **Minutes** — append-only log of significant events (mental-state diff).
- **Summary** — retrospective synthesis near completion.
- **Plans** — ephemeral, file-first execution context for current work.
- **Designs** — durable reasoning that explains why a design is correct.
- **Comments** — timestamped entries for team interaction and discussion around the ticket. Other roles traditionally absorbed into comments by issue trackers (significant events, designs, summaries) belong in the artifacts above.
- **TODOs** — low-ceremony seeds of deferred work.

## Editing Discipline

- The filesystem is the source of truth; tolerate missing files.
- **File-first:** write plans and designs directly to file; do not generate in chat and transcribe afterward.
- Preserve unknown files at the ticket root and unknown properties in `description.yaml` on edit.
- `minutes.md` and `comments/*` are **append-only**; do not modify earlier entries.
- Do not invent properties for `description.yaml` or rewrite it casually. When in doubt about a metadata change, read [General Operations](operations.md).

## Dates and Timestamps

- Use the **conversation-context date** for `YYYY-MM-DD` calendar dates in filenames and entries (plan filenames, minutes entries).
- Use `date -u +"%Y-%m-%dT%H-%M-%SZ"` from the terminal **only** for UTC-timestamp filenames (comments).
- Do **not** use terminal `date` to obtain a calendar date; the terminal's UTC date may not match the user's local calendar date.

## Substantive Operations

Route creating or completing tickets, updating minutes, adding comments, writing summaries, listing or filtering tickets, and similar substantive operations through the procedures and general operations listed below rather than improvising. Recognition, reading, and incidental conservative edits remain in scope here.

# Jekt Procedures

Load the appropriate Jekt instructions only as needed to explain how to perform the current objective, prompt, or user query (collectively referred to as "request"). Each procedure below has a description; if the procedure is not relevant, do not load it but remember it and load it when needed later.

## Housekeeping

**Important: Context Reload** If there are any indications that the conversation has been recently summarized, you **MUST** reload the project context by following Load Context, below, before anything else, even if you have previously loaded the context. Indications that the conversation may have been compacted:

- A `<conversation-summary>` block is present in the first user message.
- The summary mentions "compaction" or an "uncompacted transcript", and gives a transcript file path (`.jsonl`) with a line count.
- The summary is written in the agent's own first-person voice ("I", "my earlier claims") and has fixed sections such as Overview, Progress, Recent Operations and Continuation Plan.
- It describes substantial prior tool activity, but the message history has no earlier assistant turns, tool calls or tool results.
- An empty `<context>` or `<userRequest>` block appears just before the real request.
- The summary names tools or modes that are not in the current toolset.
- The environment is newer than the summary. For example, terminal state shows commands the summary does not mention.
- The user says the conversation was "summarized" or "compacted".

### Project Context

* [Load Context](procedures/context-load.md) - Load relevant project context (project declaration, readme, and the current ticket's artifacts) from general to specific, e.g. after a conversation is compacted. **Always follow this procedure the first time any of the following occurs:**
  - The request relates to the project.
  - The request may relate to the project's subject matter.
  - The request involves performing a search and examining the contents of the project.
  - The request results in a Jekt operation, including any procedure below.

  This includes:
  - The first such time since the beginning of a new conversation.
  - The first such round after a conversation has been summarized.

  You MUST load the context _before_ the action that needs it, as a preliminary step, and then continue on to that action.

## Initialization

* [Initialize Jekt](procedures/init.md) - Initialize Jekt in a repository, declare a project, or create the Jekt management branch.

## Tickets Procedures

* [Create Ticket](procedures/ticket-create.md) - Create a new ticket, including from a TODO.
* [Start Ticket](procedures/ticket-start.md) - Start, resume, or switch to work on a ticket.
* [Summarize Ticket](procedures/ticket-summarize.md) - Write or revise a ticket's summary.

## Plan Procedures

* [Create Plan](procedures/plan-create.md) - Capture the planning discussion as a new plan for the ticket, with any designs the discussion produced or refined.
* [Assess Plan](procedures/plan-assess.md) - Assess an existing plan against the code and the ticket, without changing files.
* [Update Plan](procedures/plan-update.md) - Revise an existing plan after further discussion, with any designs the discussion produced or refined.
* [Implement Plan](procedures/plan-implement.md) - Implement a Jekt plan or named chunks of it, or a small, concrete change agreed in conversation.

## Minutes Procedures

* [Update Minutes](procedures/minutes-update.md) - Record significant events of the work in the ticket's minutes.

## TODO Procedures

* [Create TODO](procedures/todo-create.md) - Set work aside for later in a TODO document.

## Workflow

* [Commit](procedures/commit.md) - Commit the staged changes.

## General Operations

* [General Operations](operations.md) - How Jekt operations are carried out when no procedure covers them: searching or filtering tickets, answering questions about project state, and changes such as a ticket's status, resolution, priority, assignee, tags, or releases.
