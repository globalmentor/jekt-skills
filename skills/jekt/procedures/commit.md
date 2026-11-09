---
modified: 2026-11-01
---

# Commit

You are composing a Git commit message for the currently staged content and performing the commit. The message describes a coherent unit of work: what is being committed, and why. When the work is on a Jekt ticket, the subject carries the bracketed ticket identifier; when no ticket applies, the subject is bare and the message follows ordinary Git convention.

The scope of this operation is composing the message and creating the local commit. It does not stage, unstage, push, switch branches, modify Jekt artifacts, or perform any operation other than `git commit`.

## 1. Operational Context

- **Repository.** Operate against the default workspace root unless the request indicates another root. The operation requires a Git working tree; if not under Git, stop and report.
- **Jekt presence.** `.jekt/` may or may not be present. When present, ticket attribution becomes possible per §3; when absent, the procedure composes a conventional commit message without bracketed-ID attribution and does not create `.jekt/` as a side effect.
- **Multi-root workspace.** Files visible in other workspace roots belong to other repositories. Do not act against them.

## 2. Determining What Is Being Committed

Work through the following steps in order. The ordering matters: each step grounds the next.

1. **Verify the staged content via a direct Git command** (typically `git diff --cached` and `git diff --cached --name-status`). The staged index is authoritative; do not trust your recollection of what was edited. Files you remember editing may have been edited back to match `HEAD` during the conversation. If nothing is staged, stop and report — there is nothing to commit, and the procedure does not stage.

    When the user is amending the previous commit, the staged content may be only a delta against it, or there may be no staged content at all if only the message is being changed. Treat the prior commit's content as part of what the commit will contain, treat its existing message as the starting point, and change only what the user has named.
2. **If a Jekt ticket applies, read its description** for the work's stated objective (Objective, Acceptance Criteria). When the work plausibly relates to a current plan in `plans/`, read the relevant plan section as well. This grounds the message's language in the ticket's stated end-state rather than re-deriving from the diff alone. Skip when no ticket applies.
3. **Read the staged diff against `HEAD`.** Read the actual changes, not the conversation's memory of them. The diff is the authoritative record of *what* is being committed; conversational recall is fallible, especially across summarization.
4. **Reconcile with the conversation, which typically supplies the *why*.** Where the diff and the conversation disagree about what changed, the diff is authoritative. Where the diff shows what changed but not why, the conversation is usually the source. When staged files appear that the conversation did not address, the most common explanations are intentional ancillary changes the user is folding in, or context the conversation has lost across summarization. Mention significant unaddressed staged work in the body when it belongs to a different category of change (for example, a version bump alongside a bug fix); do not silently drop it. When the staged set looks abnormal — sharply at odds with the conversation, unexpectedly large or scattered, or otherwise suggests the user may not realize what is staged — surface that before composing. Routine staged sets get no commentary.
5. **Determine the central objective and how the staged content coheres around it.** A good commit reads as one thing — a coherent unit of work — not as a random snapshot. The central objective is the thread that holds the staged content together; the message names that thread. Weight the determination by:
    - the user's stated intent for this commit or the work it advances;
    - when a ticket applies, the ticket's stated objective;
    - when a plan applies, the plan's chunk or section the work advances;
    - the structural weight of the diff itself.

    Diff structure alone is the weakest signal. A commit whose diff is mostly documentation may nonetheless be primarily about a single added behavior, when the ticket and conversation make that central. The subject and body should reflect the central purpose, not the largest section of diff.
6. **Compose the message, self-check it, and perform the commit** per §4–§7.

## 3. Ticket Attribution

A Jekt ticket attributes the commit when one applies. Signals available — in no fixed order — include the current Git branch matching `tickets/{ticket-id}`, staged paths under `.jekt/tickets/{ticket-id}/` (which may reveal multiple tickets in scope when paths span more than one ticket), explicit user instruction, and the conversation. Apply judgment; the goal is correct attribution, not exhaustive signal collection.

When ticket attribution applies, the subject carries the bracketed identifier per §4. When multiple tickets are genuinely in scope, all appear in the subject in the multi-ticket form: `[foo-123] [foo-124] Subject`. When no ticket applies — including when the user has named "no ticket" explicitly — the subject is bare; the bracketed prefix does not appear and no placeholder substitutes for it.

## 4. Subject

The subject form when a ticket applies:

```text
[foobar-123] {Subject in imperative mood}
[foobar-123] [foobar-124] {Subject in imperative mood}
```

When no ticket applies, the subject is bare:

```text
{Subject in imperative mood}
```

- **Imperative mood, present tense, no trailing period.** This is the cross-industry Git convention. "Add token rotation," not "Added" or "Adds"; no period.
- **No colon after the bracketed identifier.** A colon may appear later in the subject only as ordinary sentence punctuation.
- **Length.** Compose the subject, then count its characters including the bracketed prefix and any spaces. If over 72 characters, recompose for concision before drafting the body. Do not truncate words or drop the bracketed prefix to fit. The hard ceiling is 80; reaching it is a signal to recompose unless concision genuinely cannot fit. Your internal sense of subject length is unreliable; the count must be verified.
- **Semantic Markdown** throughout the subject. Use backticks for code identifiers and literal values (for example, `` Fix `FooBar` constructor ordering ``).
- **Describes the accepted change**, at a level a release reader would scan, not the work session. "Implement token rotation," not "Work on foo-123," not "WIP."

## 5. Body

The body is optional. Many commits are correctly subject-only. When a body is warranted, it explains *what* the change does and *why* it takes the form it takes, at a level higher than the diff.

**Sizing.** Match the body's size to what a future reader needs to understand the commit, not to the volume of work performed.

| Reader needs | Guideline |
|---|---|
| **Minimal** (routine change, no surprises, no notable decisions) | Subject only, or subject plus one short sentence. |
| **Moderate** (one or two decisions worth recording, or distinct concerns held together by the commit's central purpose) | Short paragraph or a few bullets; roughly 30–100 words. |
| **Substantial** (architectural impact, significant pivots, multiple intertwined concerns) | One or more paragraphs, or grouped bullets; roughly 100–200 words. Rare. |

If the change was straightforward execution of a known pattern, say so briefly and stop.

**Form.**

- **Prose by default** for cohesive single-purpose changes that read as narrative.
- **Bullets** when the body enumerates discrete concerns held together by the commit's central purpose. Prose and bullets serve different purposes; do not concatenate distinct concerns into a run-on paragraph, and do not bullet-list a single cohesive narrative.
- **End-of-sentence punctuation** in body sentences and in bullet items that are phrases or sentences. Bare enum-style lists (single words, identifiers) need no punctuation.
- **References to other tickets** use the bare bracketed form (`[foo-123]`). Markdown reference link definitions do not appear in commit messages; bracketed identifiers in commit logs are searchable by `git log --grep`, and that is their role here.

**Body line width is independent of subject length.** Target 72 characters per line. Lines consistently shorter than 65 characters across the body indicate the body is mimicking the subject's visual length rather than wrapping for readability; that is a recompose signal.

**Content.**

- The subject of the message is the committed artifacts and their purpose, not the procedures used to produce. Describe the change at a level higher than the diff. Do not explain artifacts that are self-evident (e.g. Jekt minutes/summary).
- Prioritize functional changes (features, fixes, behavior) over supporting changes (documentation, tests, refactoring). Lead with the functional change; mention supporting work secondarily.
- When the commit includes work from a different category of change than the central purpose (a version bump alongside a bug fix, for example), mention it briefly so the future reader is not surprised; do not weight it equally with the central change.
- Reference plans, designs, or related tickets where they aid the future reader; do not paste their content.

## 6. Self-Check

Before performing the commit, evaluate the composed message against the failure modes below. Filter in your reasoning workspace; do not narrate the self-check in chat.

- **Subject length.** Counted character-by-character including the bracketed prefix: is it within 72, or, when concision genuinely cannot fit, within 80?
- **Subject form.** Bracketed ID present when a ticket applies and absent when none does; no colon after the bracketed ID; imperative mood with no trailing period; semantic Markdown for any identifiers.
- **Disproportion.** Is the message disproportionately large for the conceptual size of the change? A small change that produces a large message has likely lapsed into release-notes verbosity.
- **Tight wrapping.** Are body lines consistently shorter than 65 characters? If so, the body is mimicking the subject's visual length; rewrap.
- **Anti-patterns** (§11). Has the message slipped into file enumeration, line or test counts, build-and-test-success reporting, process narration, documentation-content summarization, general-design-philosophy explanation, LLM-tooling content, or any other named anti-pattern?
- **Conventionality.** Does the message read as a typical, well-formed Git commit message? Unusual or unconventional form is a recompose signal.

Recompose before the commit if any check fails.

## 7. Disclose and Commit

The composed message is what will appear in `git log`; do not repeat it in the chat. Disclosure before commit is brief:

- The ticket(s) attributed, or "no ticket attributed" when none applies.
- The number of staged files and the branch being committed to.
- That the commit is local-only and not pushed.

Then perform the commit in the same turn; do not insert a confirmation step. For a newly created commit, routine reversal is `git reset --soft HEAD~1` (preserves staged content).

**Commit mechanism.** Use a heredoc whose delimiter is single-quoted, so the message passes through the shell without expansion:

```bash
git commit -F - <<'EOF'
[foobar-123] Subject in imperative mood

Body paragraph or bullets here. Backticks, single quotes, and other
shell metacharacters inside the message are not escaped because the
heredoc delimiter is single-quoted.
EOF
```

Do not use `git commit -m "…"` for multi-line messages — `-m` mis-escapes backticks and embedded quotes for any non-trivial message. Do not write a tempfile. If the host shell does not support heredocs (Windows `cmd.exe`, some PowerShell configurations), use whatever multi-line-message-supplying mechanism that host's Git invocation provides.

Trust Git. If a hook rejects, identity is unconfigured, signing fails, or any other Git error surfaces, present Git's output directly and stop. Do not interpret, work around, or use `--no-verify`.

## 8. Result

After the commit, report briefly:

- Short SHA and the subject as written.
- Branch, ticket(s) attributed (or none), file count.
- "Not pushed."

If a file the user seems to have meant for this commit did not make it in — and was not already committed earlier in this conversation — mention it briefly, not as a statistic.

## 9. Closing Footer

End the reply with a brief statement making the user aware of the most likely next actions in Jekt that they can request, building on what the reply has already reported. It tells the user which step is most appropriate now and gives someone new to Jekt a sense of what is possible. Mention an action only when it serves a real purpose at this point; when none does, add nothing. The actions below, and when each tends to apply, are for deciding which to present.

- When the committed work produced decisions, insights, or changes of direction worth recording that the minutes do not already hold: updating the minutes so that they are captured.
- When a plan is in progress and chunks of it remain: implementing the next one.
- When the commit appears to complete the ticket's work: summarizing the ticket and resolving it.

Pushing is the user's by their usual means and is not among the actions.

For example, after a commit partway through a plan, with a decision not yet in the minutes: "You might next update the minutes, so that the decision to rotate tokens per session is captured, and then implement Chunk 3 of the plan." The example illustrates length, tone, and how actions combine; it is not a template.

## 10. Boundaries

This procedure does **not**:

- Stage or unstage. The user's curation of the staged set is the contract.
- Push, fetch, tag, rebase, or switch branches.
- Modify any Jekt artifact (including `description.yaml`, minutes, plans). The commit captures what was staged; other operations own their own writes.
- Initialize `.jekt/` if absent.

## 11. Anti-Patterns

Do not:

- **Use `git commit -m` for multi-line messages.** Backticks and embedded quotes mis-escape. Use the heredoc form.
- **Use `--no-verify`.** Hooks the user has configured are part of their workflow.
- **Use `git commit -a`.** It auto-stages tracked changes; the user manages the index.
- **Place a colon after the bracketed identifier.** Write `[foo-123] Subject`, not `[foo-123]: Subject`.
- **Include Markdown reference link definitions** in the message body.
- **Enumerate every file changed, or list line counts, test counts, or other diff statistics.** The diff itself is the canonical record; the message is its semantic counterpart. Diagnostic: does the majority of your message consist of a list of files changed or counts of things? If so, rewrite.
- **Report build, compilation, or test success.** Committed code is assumed to work; this is not message content.
- **Narrate the process.** Phrases like *"Work began by examining …"* or *"Through discussion, we refined …"* are diary-register narration about producing the work, not about the work itself.
- **Describe iterative not-committed changes.** When an implementation went A → B → final form during the conversation, the message describes the final form, not the iteration.
- **Explain general design philosophy.** *"Dependency injection improves testability …"* is textbook filler; *"Constructor injection was chosen because the lifecycle of `Foo` required …"* explains this commit's choice.
- **Summarize or reproduce the content of documents being committed.** Identify the documents; do not paraphrase their content.
- **Include LLM-tooling content.** Prompts used, agent configuration, host details, or that an LLM helped compose the commit are not message content.
- **Curated work product, not curation log.** Include only what belongs in a commit message — what changed and why. Do not narrate the curation: why content was included or excluded, what kind of message this is, what other artifacts will cover, or what operation comes next.
- **Borrow reference syntax from code-documentation systems that extend Markdown.** Commit messages use CommonMark; they are not rendered by any particular code-documentation system. Documentation systems built on Markdown often add their own reference syntax (notably Javadoc's `Foo#bar()` member-reference form) that is not CommonMark and has no meaning outside the documentation system that defines it. Use ordinary code-identifier form instead: `` `Foo.bar()` ``, not `` `Foo#bar()` ``.
- **Include CI directives** (`[skip ci]`, `[ci skip]`, similar) unless the user explicitly requested them.
- **Include trailers** (`Co-authored-by:`, `Signed-off-by:`, similar) unless the user's Git configuration or explicit request supplies them.
- **Leak local-system information** — absolute paths, paths outside the repository, machine or account names, home-directory references, environment values, or local tool configuration. The constraint is on the author's local system, not on the commit's subject matter; references to code paths and files inside the repository are part of the message's job.
- **Generate a large message for a small conceptual change.** Sizing follows §5.
- **Speculate about future work.** Follow-ups belong in TODOs or new tickets, not in the commit message.
