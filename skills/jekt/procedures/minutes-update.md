---
modified: 2026-10-29
---

# Update Minutes

Append entries to the active ticket's `minutes.md` for significant events in the work stream since the last entry. This procedure is designed to be cheap enough to follow frequently. Reason silently; produce minimal output; if nothing qualifies, write nothing.

This procedure writes the file. It does not stage, commit, summarize, or produce other artifacts; it does not synchronize with external trackers; it does not infer entries from activity in other workspace roots.

## 1. Scope and Target

Operate against the default workspace root unless the request indicates another root. With multiple workspace roots open, do not silently switch; if a ticket ID resolves only in a non-default root, treat that as ambiguity and ask which root to operate against.

The target is the active ticket's `minutes.md` at `.jekt/tickets/{id}/minutes.md`. Determine the ticket as follows:

| Precondition | Action |
|---|---|
| `.jekt/` is absent at the selected repository root | Stop. Recommend Initialize Jekt. Do not create `.jekt/` and do not append to any other file. |
| The current Git branch is `tickets/{id}`, or the user has explicitly identified a ticket as the target for this update, and `.jekt/tickets/{id}/` exists | **Proceed** with that ticket. |
| A ticket is the target, but `.jekt/tickets/{id}/` does not exist | Stop. Report the missing directory and suggest Create Ticket. Do not create the ticket directory implicitly. |
| No ticket is the target | **Ask** which ticket to operate against; do not infer. A ticket merely *mentioned* earlier in conversation is a candidate to offer, not an answer. |

Explicit identification means the user has supplied the ticket as a target for this update in the request.

## 2. Definition and Purpose

Minutes preserve significant events and realizations in the evolution of work on the ticket: breakthroughs in domain understanding (especially DDD reconceptualizations — "what this thing actually is"), why the system is shaped this way, what alternatives were rejected, what was discovered along the way, operational traps, and milestones that anchor the timeline. They record the *mental state diff*, not the code diff.

Minutes bridge ephemeral conversations with persistent project state. They serve future sessions after conversation summarization has discarded detail, contributors continuing work weeks or months later, and LLMs that have lost context.

**Minutes record the semantic work stream — what happened and why, from the perspective of doing the work. Git commits record accepted changes. The two are different artifacts capturing different things. Commit order, commit timestamps, and the contents of commit messages are not the substrate of minutes and are not consulted by this procedure.**

**Test:** Would someone returning later to this ticket be at a disadvantage without this knowledge? If it is fully captured in code or commit history, skip it. The existence of an outcome in the description, in a plan, in a design, in code, or in commits does not by itself capture the deliberation that produced it — minutes are where the deliberation lives.

## 3. Substrate

Candidates are drawn from your accessible conversation context, including any summarized portions, back as far as you can see. Do not consult Git history, repository search, or other sources for candidate identification unless the user explicitly asks.

## 4. Format

Each entry is a single bullet of the form:

```markdown
- YYYY-MM-DD **Category**: First entry description.
- YYYY-MM-DD **Category**: Second entry description.
```

Use the **conversation-context date** for `YYYY-MM-DD`. Do not invoke shell commands to obtain the date.

**Categories:** **Pivot** (strategy change), **Insight** (reconceptualization), **Decision** (choice with rejected alternatives stated), **Finding** (discovery of something previously unknown that affects the work), **Lesson** (operational trap or process failure, with the avoidance that makes it actionable), **Milestone** (significant implementation completed), **Open** (deferred question), **Resolved** (previously open question now answered).

**First creation.** If `minutes.md` does not exist, create it with the ticket header, followed by a blank line, the categories comment, another blank line, and then the entries. The file ends with the appropriate newline sequence.

Header:

```markdown
# [{ticket-id}] Minutes
```

Categories comment:

```markdown
<!-- Categories: Pivot | Insight | Decision | Finding | Lesson | Milestone | Open | Resolved -->
```

## 5. Patterns to Include

- DDD breakthroughs where "what this thing is" changed
- Decisions with rejected alternatives stated
- Large-scale implementations or refactorings completed
- Operational traps and process failures (recorded with the avoidance that makes them actionable)
- Mistakes in analysis that led to incorrect work
- Timeline-anchoring milestones
- Convention decisions that establish normative patterns for future work, where the rationale would otherwise be lost (naming conventions, structural patterns, documentation standards)

## 6. Anti-patterns to Exclude

- Small-scale code tweaks (added, moved, or removed single methods or symbols)
- Implementation mechanics or debugging details
- Test fixes and minor adjustments
- Routine progress following a known plan
- Anything fully captured in code or commit history
- A "Lesson" that is not actionable, in that it does not teach how to avoid the issue. *"We learned that bad things can happen"*, with no learning of how those bad things can be avoided, is not a Lesson. *"Avoid the bad thing"* is not a Lesson in how it can be avoided.
- **Local-system leakage** — no absolute paths, paths outside the repository, machine or account identifiers, environment values, or local tool configuration in entry text.
- **Curation log content** — entries record the work's shifts in understanding, not commentary about the curation: why an entry was included or excluded, what kind of artifact this is, what other artifacts will cover, or what operation comes next.

## 7. Process

Reason silently through these steps (in the host's thinking channel if available); do not narrate them in chat.

1. **Review and enumerate.** Review the work stream since the last entry in the existing file, or since the start of the accessible conversation if the file is new, empty, or header-only. Enumerate candidate decisions, pivots, insights, findings, lessons, milestones, open questions, and resolutions before filtering; do not collapse the conversation into a synthesized verdict and filter against the verdict.
2. **Apply the discriminators.**
    - For **Insight** or **Pivot** candidates: can you state *"Before we thought X / After we realized Y"*? If you cannot articulate the prior and current states, the candidate is not a real Insight or Pivot — drop it.
    - For **Decision** candidates: a Decision is a strong candidate when it was both *deliberated* and *consequential*. **Deliberated** means alternatives were weighed or rejected and reasons were named. **Consequential** means the choice constrains future work, shapes the ticket's surface, establishes a convention, rejects a path future readers may reasonably reconsider, or explains why the system is shaped this way. A decision that is only one of the two may qualify if future continuation would be impaired without it; a decision that is neither does not qualify.
3. **Synthesize.** Related items about the same insight or decision become one entry, not several.
4. **Filter.** Apply the §2 test against this ticket.
5. **Append** at the file's true end (see §8), in chronological order, focused on realizations, decisions, and significant events rather than code actions.

**Catch-up guard.** If the file is new, empty, or header-only and the accessible conversation contains substantial planning or design deliberation, the enumeration step is load-bearing: list the candidates internally before filtering. If filtering then leaves 0 or 1 entries, re-check whether deliberated and consequential decisions were wrongly dismissed because their outcomes appear in the description, in a plan, in a design, or in code — outcomes elsewhere do not capture the deliberation (see §2).

You are not obligated to add entries. If nothing significant occurred, make no changes.

## 8. Reading and Writing the File

Use the agent's structured workspace file-reading facility — the one that takes a path and a line range and returns the file's contents through the agent runtime — for all reads of `minutes.md`. Do not reach for shell commands such as `wc -l`, `cat`, `tail`, `head`, or `stat`. Shell commands for these reads bypass the agent runtime's safeguards and are platform-specific.

The workspace facility does not expose file size directly. Use these recipes:

- To check whether the file is shorter than N lines, request lines 1 through N. A response shorter than N lines confirms it.
- To read the file's tail without reading the whole file, request a range whose end extends past where the file is expected to end. A truncated response is the EOF signal.
- To locate EOF in a file of unknown length, probe outward in widening ranges until a request returns fewer lines than asked for.

**For typical reads on long-running files,** read only the header (first ~10 lines) and a recent tail obtained via the EOF-probing recipe above — typically 100–200 lines of tail is sufficient. Reading the entire file each time is wasteful and unnecessary; the recent tail plus the header is the working set.

**Locating the true end of the file before appending.** The append target is the byte after the last existing entry, not "the end of the range I happened to read." Find the true end by:

1. Performing a tail read whose range extends well past where the file is expected to end.
2. Confirming the returned content is shorter than the requested range; equal length means EOF has not been confirmed and the read must extend further.
3. Treating the position after the last bullet in the confirmed-EOF read as the append target.

These steps must be followed. Do not assume the last entry in a partial read is the last entry in the file — lower-powered models that skip this step routinely insert new entries in the middle of the file because they treated a partial read as complete.

**Writing the file.** Use the agent's structured workspace editing facility — the one the agent runtime tracks — for the append. Do not append with shell redirection (`echo >> minutes.md`, `printf >> minutes.md`, `cat >> minutes.md`, or equivalents). Shell redirection bypasses the agent runtime's safeguards, mishandles line endings across platforms, and produces no diff visualization or undo integration.

## 9. Append-Only Discipline

Minutes are append-only. Do not modify or reorder earlier entries. Never insert new entries between existing entries — chronological sequence in the file takes priority over topical grouping. If a prior entry has been superseded by new understanding, or if a previously `Open` question is now `Resolved`, add a new entry that references the prior one and explains the change; do not edit the prior entry.

## 10. Output

Chat output is a single status line: target path and count of entries appended. If the repository is under Git, also note that the file is not staged or committed.

Do not narrate filtering, synthesis, or categorization. If no entries qualified, write nothing and report a single line saying so.

**Compaction-aware gap notice.** If you can see that earlier conversation has been summarized or compacted — a summary marker, missing turns you would expect to recall — add a brief note to your status that some events may not have been captured and that updating the minutes again with additional sources may help. Conversation length, fresh-file catch-up, and deliberation predating this update are not evidence of compaction.
