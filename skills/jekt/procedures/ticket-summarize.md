---
modified: 2026-11-01
---

# Summarize Ticket

You are creating or revising the summary for a Jekt ticket. The summary is `summary.md` at the ticket's directory root: a synthesis of what was accomplished on the ticket, drawn from the full set of ticket artifacts and the ticket's Git commit history. It serves future readers who need to understand the ticket later — the same developer months later, another developer revisiting the area, or an LLM brought into a context where the ticket matters.

The scope of this operation is reading the ticket's artifacts and commit history, synthesizing them into `summary.md`, and writing the file. The operation does not commit, push, switch branches, change the ticket's status, or modify any artifact other than `summary.md`.

## 1. Operational Context

Establish before acting:

- **Repository root and `.jekt/`.** Operate against `.jekt/` in the default workspace root unless the request indicates another root. If `.jekt/` is absent, stop and recommend Initialize Jekt; do not initialize here.
- **Multi-root workspace.** Files visible in other workspace roots belong to other repositories. Do not act against them, and do not include paths into them in the summary content.
- **Git is helpful but not required.** When Git is available, the ticket's commit history is one of the inputs (§3). When Git is absent, or when the ticket has no committed code work, the summary is drawn from the artifacts alone.

## 2. Ticket Resolution

Identify the ticket from the request, from the current Git branch (`tickets/{ticket-id}`), or from clear conversational intent. If genuinely uncertain, ask. Identifier matching against existing ticket directories under `.jekt/tickets/` is case-insensitive on the project slug.

The ticket directory must exist in the current working tree (`.jekt/tickets/{ticket-id}/`); the summary is written there. If the directory is not present in the current working tree, stop and report — the user can switch to a branch where it is present and summarize again.

## 3. Reading Discipline

Read the ticket's full artifact set and its commit history; synthesize from the union. The artifacts of interest:

- `description - {Label}.md` and `description.yaml` — the ticket's intent and metadata.
- `minutes.md` — the chronological record of significant events.
- `designs/` — durable design reasoning.
- `plans/` — execution history.
- `comments/` — team interaction around the ticket.
- Ad-hoc work products at the ticket root (reports, analyses, migration guides, and similar).
- An existing `summary.md`, if present (see §6).

For Git commit history, read commits attributed to the ticket. The commit-subject form `[{ticket-id}] {subject}` is the durable cross-incarnation link: a ticket's commits remain identifiable after branch merges and across re-branchings by the bracketed-identifier prefix in the subject line.

Read commits, not just commit subjects, when the substance of a change is not evident from the subject alone. Verify decisions and outcomes against the actual files and commits rather than relying only on conversational memory.

**No network access.** Do not `git fetch`, `git pull`, or `git ls-remote`. Local refs and reachable commits are sufficient.

## 4. What the Summary Carries

The summary synthesizes what happened on the ticket for a future reader who has the artifact set available to drill into but needs the synthesis to know whether to drill in and where. Common consuming uses include a developer revisiting the ticket months later, an LLM picking up context for related work, and release-notes composition pulling the opening from the summary.

The summary opens with a **front-loaded statement of the outcome** — written so it can stand on its own, such that a release-notes pass could lift it verbatim. The opening is always present; everything else is shaped by what the ticket warrants.

The body covers, to the extent each is present in the ticket's work:

- **Outcome** — what the ticket actually produced, in prose, expanding the opening statement.
- **Relation to initial intent** — how the outcome compares to the description's stated intent. Included when the work drifted, scope shifted, was partially completed, or resolved as `invalid`, `duplicate`, or `abandoned`. Non-completion resolutions are first-class summary cases; the future reader's question "what happened with that?" is highest-value precisely when there is no code to read as a fallback. Omitted when intent and outcome match cleanly.
- **Key decisions and pivots** — gist-level account of the decisions and reversals that shaped the outcome, with references to the relevant minutes entries and design documents rather than restating their content.
- **What was produced** — deliverables: code changes at a high level, designs created or refined, follow-on tickets opened, deferred work captured as TODOs.
- **Handoff notes** — what a future contributor working in this area needs to know: caveats, known limitations, surprising constraints, edge cases discovered, items deferred but worth flagging. Probe for handoff content; omit only when there is genuinely none.

Sections are omitted, not stubbed empty. A trivial ticket may collapse to just the opening statement and a sentence of outcome. A complex ticket may use all five categories.

The shape above is a vocabulary of what may appear, not a fixed template. Use Markdown headings to structure longer summaries; short summaries do not need headings. Order roughly as listed when multiple categories are present.

## 5. Disclosure

Before writing the file, the user needs to understand what is about to happen. Cover this as part of the assistant's normal speech to the user — integrated into the response, not bracketed as a separate labeled step. The substance to cover:

- The target file (`.jekt/tickets/{ticket-id}/summary.md`) and whether this is a first write or an in-place revision of an existing summary.
- That the operation reads files and Git history but does not commit, push, switch branches, fetch, or modify any other artifact.
- The cancellation affordance: the user can interrupt now; after the write, the file can be removed or further edited.

If the current branch is not the ticket's branch (and the ticket directory is nonetheless present in the working tree), note it briefly so the user can summarize again from the ticket's branch if that would draw on different material.

Act in the same turn without waiting for a separate confirmation. The discipline is *disclose then act*.

## 6. Writing the Summary

Write `summary.md` directly to disk using the agent's structured file-editing facility. Do not draft the summary in the conversation and transcribe it to the file afterward; the transcription regenerates from compressed understanding and silently mutates content.

**Heading.** The file's first heading is `# [{ticket-id}] Summary` for a plain summary, or `# [{ticket-id}] Summary: {Brief Elaboration}` when a short title elaboration aids identification.

**Revision of an existing summary.** When `summary.md` already exists, treat it as the authoritative record of prior work. Refine the sections that the new material affects; preserve everything else. Do not regenerate the file from scratch.

**Reopened tickets.** When the ticket was reopened and additional work was performed since the prior summary was written, revise the file to reflect the full evolution — what was originally accomplished, what prompted the reopening, and what changed in the additional cycle. The summary represents the ticket's current closed state, not a chronological log of closure cycles; the chronological record lives in minutes.

## 7. Sizing

Size the summary to **what a future reader needs to understand**, not to the volume of work performed. Reading a thousand files to produce four routine outputs is a small summary. Making one architectural decision with lasting implications may warrant detailed explanation.

Consider:

- How many **non-obvious decisions** were made that affect future work?
- Were there **course corrections or roadblocks** that explain why the result looks the way it does?
- Are there **deferred items or gotchas** a future developer must know about?

| Reviewer Needs | Guideline |
|---|---|
| **Minimal** (routine output, no surprises, no deferred work) | 50–100 words. State what was produced and any minor observations. |
| **Moderate** (some decisions worth recording, or deferred items) | 150–300 words. Explain key choices and what was left undone. |
| **Substantial** (architectural impact, significant pivots, complex tradeoffs) | 300–500 words, with headings for scannability. |

If the work was straightforward execution of a known pattern, say so briefly and stop.

## 8. Writing Discipline

- **Neutral, impersonal voice.** Do not use *I*, *we*, *the team*, *the developer*, or proper names. The grammatical subject is the work itself or the artifact in question (*"The API was extended …"*, *"The token-rotation strategy uses …"*). Active voice when the subject is the artifact or change; passive only when there is no natural inanimate subject. Authorship is recoverable from Git.
- **Prose by default; lists sparingly.** Bulleted lists only when items form a logical group. The default form is prose that tells what happened.
- **Architectural understanding over enumeration.** Describe what changed and why. Code-level references (specific classes, methods, files) appear only when they name **key integration points** or **non-obvious design choices** a future developer would need to understand. Do not enumerate every method, file, or call site touched.
- **References, not duplication.** When substance lives in a design document, a minutes entry, or another ticket, point to it rather than restating its content. Cross-ticket references use CommonMark shortcut reference links (`[foobar-42]`); provide link definitions at the bottom of the summary pointing to the ticket directory: `[foobar-42]: ../foobar-42/`.
- **External URLs when topical.** External resources that are part of the ticket's work (a specification implemented against, a vendor document whose behavior was relied on) may be linked when they aid the future reader. External context already captured by the description is not restated.

**Markdown conventions:**

- Semantic Markdown: backticks for code identifiers, file paths, and literal values.
- End-of-sentence punctuation.
- No line wrapping; one paragraph per line.

## 9. Result

After writing, report briefly. The audience is a user who has just had the summary produced; favor signal over commentary.

A useful report covers:

- The target file path, and whether this was a first write or an in-place revision.
- A one-line characterization of what the summary captures (for example, "outcome, two key decisions, handoff for the cache invalidation").
- **TODO-source deletion reminder.** If the ticket's description records that it was created from a TODO in this same repository (recognizable by a "Created from a TODO" line in the description's *Notes* section that names a ticket in this repository or a project-level TODO), and the summary indicates the ticket has fully addressed what the TODO recorded, mention that the original TODO file can now be deleted. Do not delete it.

**Stop.** Do not commit, push, switch branches, or invoke other operations.

## 10. Closing Footer

End the reply with a brief statement making the user aware of the most likely next actions in Jekt that they can request, building on what the reply has already reported. It tells the user which step is most appropriate now and gives someone new to Jekt a sense of what is possible. Mention an action only when it serves a real purpose at this point; when none does, add nothing. The actions below, and when each tends to apply, are for deciding which to present.

- When the summary records the ticket's work as complete and the ticket is not yet resolved: resolving the ticket and committing the summary with it.
- When the ticket's work is not yet complete: revising the summary as the work continues.
- When synthesizing the summary surfaced an insight worth recording that the minutes do not already hold: updating the minutes so that it is captured.

For example, after summarizing a ticket whose work is done: "With the summary written, you can now resolve the ticket and commit the summary along with that change." The example illustrates length, tone, and how actions combine; it is not a template.

## 11. Boundaries

This procedure does **not**:

- Commit, push, stage, switch branches, or fetch.
- Change `jekt/status`, `jekt/resolution`, or any other metadata property. Status transitions and ticket closure are separate operations.
- Modify any artifact other than `summary.md`.
- Delete any file, including TODO source files (the result report mentions the deletion as a recommendation only).
- Compose release notes. The summary's opening is shaped to be liftable into release notes; the composition itself is a release-side operation.
- Promote inline observations to design documents or TODOs. If the synthesis surfaces design-grade reasoning not yet captured, the user can capture it through the appropriate planning procedure; the summary itself does not silently extract.
- Touch other tickets' artifacts. References to related tickets stay as references.

## 12. Anti-Patterns

Do not:

- **Restate the description.** A summary that opens by paraphrasing the description has paid no information dividend. The opening states the outcome; intent is brought in only when the relation between intent and outcome is itself part of the story.
- **Replay the minutes.** The summary's account of key decisions cites the minutes by reference and gives the gist; a chronological replay reinvents the minutes file.
- **Narrate the process.** Phrases like *"Work began by examining …"* or *"Through discussion, we refined …"* are diary-register narration about producing the work, not summary of the work. Narration is excluded even when impersonal phrasing satisfies the neutral-voice rule.
- **Explain design choices as general best practice.** Explanations of design choices are about *this implementation*, not about why the chosen approach is generally good practice. *"Constructor injection was chosen because the lifecycle of `X` required …"* is summary content; *"Dependency injection improves testability and decouples concerns …"* is textbook filler that carries no information about this ticket.
- **Summarize Jekt-bookkeeping commits as work.** A commit whose only effect was Jekt-artifact bookkeeping (added a comment, updated a property in `description.yaml`, appended an entry to `minutes.md`, wrote a TODO) is not summary content on its own. The underlying decision may be summary content; the bookkeeping is not.
- **Include LLM-tooling content.** Discussions of prompts, agent configuration, LLM hosts, or other LLM tooling that took place during the session are not summary content even when they were part of the conversation. The summary records the ticket's work, not the tooling that produced or assisted the work.
- **Curated work product, not curation log.** Include only what belongs in the summary — the synthesis of what was accomplished. Do not narrate the curation: why content was included or excluded, what kind of artifact this is, what other artifacts will cover, or what operation comes next.
- **No local-system leakage.** Do not include absolute paths, paths outside the repository, machine or account names, home-directory references, environment values, or local tool configuration. The constraint is on information about the author's local system, not on the ticket's subject matter. Relative paths use forward slashes (`/`).
