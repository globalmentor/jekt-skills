---
modified: 2026-11-01
---

# Create Ticket

You are creating a new ticket in this Jekt-enabled repository. The scope of this operation is the ticket directory and its contents. Starting work on the ticket, committing, and other downstream operations are separate from creation and are not attempted here (see §10).

## 1. Operational Context

Establish before acting:

- **Repository root and `.jekt/`.** Operate against `.jekt/` in the default workspace root unless the request indicates another root. If `.jekt/` is absent, stop and report. Do not initialize `.jekt/` here; recommend Initialize Jekt (optionally with a project to declare) instead.
- **Ticket identifier context.** If the user supplied a ticket ID, validate it in §5 and use it as supplied. Otherwise §5 resolves the active project slug spelling before selecting a number. Gather declared project directories and existing ticket directories as needed for that resolution.
- **Multi-root workspace.** Files visible in other workspace roots belong to other repositories. Do not treat them as part of this project, do not write to them, and do not include paths into them in this ticket's content (see §8). Reading is permitted when the user has directed attention there (for example, to a source TODO).

## 2. Pre-Draft Context Gathering

Before drafting the description, read in this order:

1. The repository's `readme.md` (or equivalent orientation document), to understand what the project is.
2. If the ticket is a follow-up from or related to other tickets, read those tickets' descriptions, summaries, minutes, designs, and TODOs for relevant context.
3. The actual code or files the ticket concerns, to ground the description in current reality.
4. If the ticket originates from a TODO, read the full TODO content (see §8).

Reading is for grounding the description, not for assembling a plan.

## 3. Internal Planning, Not in the Ticket

Briefly think through the probable shape of the work the ticket implies. This is for your own grounding; it does not go into the ticket. The description is end-state-oriented; procedural reasoning belongs in a plan artifact (Create Plan) once work starts.

This partition is essential. If procedural reasoning leaks into the description, the ticket becomes a plan, and the design and planning conversation is silently skipped during execution. Write the description as what is *true* when the ticket closes, not as what *work* will happen.

## 4. Clarification Loop

If the user's request does not provide enough information to write a useful description, ask follow-up questions across as many turns as needed. Continue until you have enough information, or until the user says to proceed (e.g., "proceed", "go ahead", "create it").

When you first decide to ask, state the multi-turn contract plainly in the conversation, not only inside this procedure. Instructions in the procedure fade across turns relative to recent conversational content; a contract that you yourself have stated is what subsequent turns will attend to. Restate the missing items briefly on each follow-up turn.

A first-ask utterance has the shape:

> I don't have enough to create the ticket yet. I'll ask follow-up questions until I do, or until you tell me to proceed.
>
> Still missing: …
>
> Questions: …

## 5. Ticket Identifier

Ticket identifier determination produces one **candidate**, then secures it as the **ticket ID** — claimed on the remote in coordinated mode, or confirmed free of collisions in local mode.

The ticket ID names the ticket directory and branch — the canonical lowercase form (`foo-123`) by default. Use an uppercase slug only when the user supplies it explicitly, or the declared project (`.jekt/projects/FOO/`) or existing tickets establish uppercase as the actual slug.

Use these internal terms:

- **Observed ID** — a ticket ID visible from a local ticket directory or remote claim ref.
- **Slug family** — observed IDs whose slug spellings share the same ASCII case-folded form. A slug family is an inference device, not identifier equality.

### Observed IDs and Coordination

Collect observed IDs from local ticket directories under `.jekt/tickets/`.

Before running coordination commands, briefly explain the checks in user-facing terms: you are checking whether coordinated ticket-number allocation is enabled, updating Git's local `origin/jekt` tracking reference if present, and listing existing ticket-claim refs on the remote so the next number can be chosen. These checks do not modify project files, the index, the current branch, or any remote branch.

To check whether coordinated mode is available, first fetch `origin/jekt` with an explicit destination refspec so the remote-tracking ref is created or updated regardless of the remote's configured fetch refspec:

```bash
git fetch origin +refs/heads/jekt:refs/remotes/origin/jekt
```

The `+` allows non-fast-forward updates, so a force-push to `jekt` on origin (which should be rare and protected against in team workflows) does not leave the local view stale. Then list existing claims:

```bash
git ls-remote origin "refs/jekt/ticket-claims/*"
```

If `origin` is absent or `origin/jekt` is absent, coordinated mode is unavailable; report that normal uncoordinated state without treating it as a command failure. If the fetch or claim-listing command fails for any other reason, inform the user before falling back; do not silently degrade. If both commands succeed, add remote claim refs to the observed IDs. In local mode, use local observed IDs only and report that the ticket number is uncoordinated.

### Candidate ID

Determine the candidate ID:

1. **Explicit ID.** If the user supplied a ticket ID, validate that it parses as `{project-slug}-{number}`; reject IDs that do not match the grammar explicitly. Use the ID exactly as supplied, preserving its casing, as the candidate.
2. **Slug spelling.** Otherwise resolve the slug spelling:
    - If exactly one project is declared (`.jekt/projects/{slug}/` exists for one slug), use that directory name exactly. Ticket directories for other slug families coexisting under `.jekt/tickets/` do not override this; they belong to separate slug families and do not affect slug selection here.
    - If multiple projects are declared, infer from the subject matter of the request; ask if the subject does not disambiguate. If declared project directories differ only by case, ask rather than choosing.
    - If no project is declared, infer from observed IDs. If one slug family is in use, use the spelling from the highest-numbered observed ID in that family. If multiple slug families are in use, infer from subject matter and ask if still ambiguous.
    - If no slug can be inferred, ask the user.
3. **Number.** For an explicit ID, use the supplied number. Otherwise use one greater than the highest observed number in the selected slug family, or `1` if none exists.

Before securing the candidate, stop and report if the exact ID already exists, if the same number already appears in the selected slug family with different casing, or if the local ticket directory path would collide with an existing differently cased path on a case-insensitive filesystem.

### Coordinated Claiming

Claim the candidate ID when coordinated mode is available:

Before claiming, briefly explain that this is the coordination write: it creates a small remote claim ref under `refs/jekt/ticket-claims/{ticket-id}` to reserve the ticket number. Claim refs are not branches and do not contain project files; the push does not modify project files, the index, or the current branch.

```bash
claim_commit=$(git commit-tree "origin/jekt^{tree}" -p origin/jekt -m "Claim ticket ID {ticket-id}") \
  && git push origin "$claim_commit:refs/jekt/ticket-claims/{ticket-id}"
```

`commit-tree` produces a unique commit hash per attempt, so two simultaneous claims for the same number always produce different commits and the second push fails as non-fast-forward. The `&&` prevents a local commit-creation failure from falling through to a push with an empty source ref. Do not use `--force-with-lease`; it does not prevent the silent same-commit no-op.

Failure classification:

| Cause | Action |
|---|---|
| Non-fast-forward (collision) | If auto-allocated, retry with the next number. If user-provided, inform the user the number is already claimed and stop. |
| Local Git operation failure during claim creation (for example, missing `user.name` or `user.email`) | Stop and report the underlying Git error. |
| Network failure | Stop and report. |
| Authentication failure | Stop and report. |
| Permission denied | Stop and report. |

### Example

```
$ git fetch origin +refs/heads/jekt:refs/remotes/origin/jekt
From github.com:example/project
 * [new branch]      jekt       -> origin/jekt
# or no output when already up to date — both exit 0

$ git ls-remote origin "refs/jekt/ticket-claims/*"
a1b2c3d  refs/jekt/ticket-claims/acme-39
d4e5f6a  refs/jekt/ticket-claims/acme-40
b7c8d9e  refs/jekt/ticket-claims/acme-41
# highest claimed is 41; next is 42

$ claim_commit=$(git commit-tree "origin/jekt^{tree}" -p origin/jekt -m "Claim ticket ID acme-42") \
  && git push origin "$claim_commit:refs/jekt/ticket-claims/acme-42"
To github.com:example/project.git
 * [new reference]   83f2a1b -> refs/jekt/ticket-claims/acme-42
# success — acme-42 is claimed; proceed to create .jekt/tickets/acme-42/
```

## 6. Label and Heading Title

A new ticket has two related short strings, distinct in role:

- **Label** — the short Title-Case lexical handle materialized in the description filename's trailing portion: `description - {Label}.md`. It conveys the central concept and follows the label rules in [Jekt Definitions](../definitions.md); common abbreviations and code-style identifiers (`App`, `Config`, `TableDb`, `IPv6`) are acceptable. Examples: `description - TableDb Abstraction.md`, `description - Widget Persistence.md`, `description - Auth Token Refresh.md`. The label is a filesystem and display affordance, not a stored property; it does not appear in `description.yaml`.
- **Heading title** — the human-meaningful elaboration shown in the level-one heading of the description prose. May be longer than the label; the same words are common.

The body's first heading uses the bracketed ID form with no colon between the ID and the elaboration:

```markdown
# [foobar-123] Auth Token Refresh
```

If the user did not provide a label, synthesize a candidate from the request. Never produce a bare `description.md` — the description filename always carries a label.

## 7. Description Content

The description carries scope and intent. It is *not* a plan; it does not enumerate code changes, step-by-step procedures, or implementation alternatives.

Between the title heading and the first section heading, include a short headingless summary that expands the title into plain project-facing prose: the current gap, problem, or motivation, and the needed project state. Keep it as short as the subject allows; a trivial ticket may need only one sentence.

### Sections

Two are required; the rest are conditional. Conditional section headers are omitted entirely when no content fills them — do not write empty placeholders. Section heading level is `##` (the `#` heading is the ticket title).

| # | Section | Required | Purpose |
|---|---|---|---|
| 1 | **Objective** | yes | The end state the ticket reaches, in one or two sentences. Written as what is true when the ticket closes. Not a higher-level project outcome; not a description of work to be done. |
| 2 | **Background** | conditional | Why the ticket exists. For defects: what is broken and the user-visible impact. For features or improvements: the gap or motivation. Frame-neutral across `jekt/kind` values. |
| 3 | **Acceptance Criteria** | yes | Testable statements that mark "done." Bulleted. |
| 4 | **Constraints** | conditional | Binding limits on the solution space. Each item constrains; none recommends. "Must remain compatible with v1 on-disk format" is a constraint; "use a versioned envelope" is not. |
| 5 | **Orientation** | conditional | Where the relevant code, files, or concepts live today. Pure "read here first" pointers; no edits implied. |
| 6 | **Approach** | conditional | Endorsed direction when implementation is determined by the ticket's nature (a known task), policy, prior research, or a TODO the writer is committing to. Not a plan; not speculation. Omit when none of these conditions hold; do not generate a recommendation to fill it. |
| 7 | **Out of Scope** | conditional | Explicit exclusions to prevent scope creep at planning time. |
| 8 | **Notes** | conditional | Cross-references, terminology, related tickets, TODO provenance (see §8). Lower-ceremony catch-all. |

A two-section ticket (Objective + Acceptance Criteria) is a valid Jekt ticket. Most tickets will use three or four sections, not all eight.

### Acceptance Criteria form

Each criterion is a testable statement of observable behavior. Examples:

- *CORS preflight requests from the configured web site return a valid `Access-Control-Allow-Origin` header.*
- *Dev mode continues to allow `localhost` origins on any port.*
- *No CORS headers are added when no web domain is configured.*

Three to six criteria is typical. Fewer is fine if the objective is genuinely simple.

### Content discipline

- **End state, not implementation.** Describe what is true when the ticket closes, not the work to do. If you find yourself writing "first do X, then do Y", stop — that belongs in a plan.
- **Orientation, not prescription.** Pointers to where things are today are useful; instructions on what to change are not.
- **Implementation details only when** essential to a constraint, non-obvious enough to be lost during planning, or required context that would otherwise need rediscovery. Default to omitting.
- **No speculative alternatives or uncommitted recommendations.** Comparing approaches the writer is not committing to belongs in a plan or a design discussion, not the description. Endorsed direction with a clear reason belongs only in *Approach* (§7 above) — used when prescription is genuinely warranted, never as a default.
- **Curated work product, not curation log.** Include only what belongs in a ticket description — the work's scope, end state, and supporting orientation. Do not narrate the curation: why something was included or excluded, what kind of artifact this is, what other artifacts will cover, or what operation comes next.
- **No local-system leakage.** Do not include absolute paths, paths outside the repository, machine or account names, home-directory references, environment values, or local tool configuration. The constraint is on information about the author's local system, not on the ticket's subject matter. Relative paths use forward slashes (`/`).

### Markdown conventions

- Semantic Markdown: backticks for code identifiers (`ClassName`, `methodName()`, `com.package.name`), file paths, and literal values.
- End-of-sentence punctuation.
- No line wrapping; one paragraph per line.
- Cross-ticket references use CommonMark shortcut reference links (`[foobar-12]`). Provide link reference definitions at the bottom of the description for every referenced ticket, pointing to the ticket directory: `[foobar-12]: ../foobar-12/`.
- The ticket's own ID requires no link definition; do not insert one.

## 8. TODO Provenance

If the ticket is being created from a TODO, two disciplines apply: preserve the TODO's substance in the description, and record provenance in a safe form.

### Substance preservation

Carry forward all substantive content from the TODO: concrete examples, enumerations, illustrative details, and domain-significant tags or markers (e.g., "security", "breaking change"). The original TODO is meant to be removed once the work fully covers it, so the description must be self-sufficient.

Do **not** make scoping decisions the TODO left open — do not mark items as out-of-scope, do not choose among alternatives the TODO presented, do not silently drop content that seems peripheral. Prior context is not authoring discretion.

### Source categorization

Before recording provenance, categorize the TODO's location:

| Source location | Path category | Safe to link | Notes |
|---|---|---|---|
| Same repo, ticket-scoped (`.jekt/tickets/{id}/todo - X.md`) | Inside this `.jekt/` | The source ticket only (not the TODO file itself) | TODO file destined for deletion |
| Same repo, project-level (`.jekt/todos/X.md`) | Inside this `.jekt/` | Nothing | Record the TODO's title in prose; file destined for deletion |
| Other repo (another workspace root) | Outside this repository | Nothing | Cross-root paths leak system configuration and break when shared |
| Non-Jekt / pasted content with no traceable file | Not in any Jekt directory | Nothing | Common when the user pastes a TODO into the conversation |

### Provenance note

In the **Notes** section, include exactly one line in the appropriate shape:

- Same-repo, ticket-scoped: `Created from a TODO in [foo-12].` (Optionally name the TODO: ``Created from the `Extract Byte Utils` TODO in [foo-12].``)
- Same-repo, project-level: ``Created from a project-level TODO (`Streaming Support`).``
- Other repo: `Created from a TODO in another repository.`
- Non-Jekt / pasted: `Created from a TODO supplied during ticket creation.`

### Path and link discipline

- **Never** include absolute filesystem paths in the description.
- **Never** include paths into other workspace roots, even relative.
- **Never** link to the source TODO file itself when it is in the same repository — the file is destined for deletion, so the link is broken by design.
- **Never** record cross-repository sources in a form that requires the source developer's local IDE configuration to resolve.

## 9. `description.yaml`

Write `description.yaml` *only* if at least one property is asserted at creation time. If nothing is asserted, do not create the file at all.

### Property rules

| Property | Rule |
|---|---|
| `jekt/kind` | Write if the user explicitly stated the kind, or if the content unambiguously implies it (`X crashes when Y` → `bug`; `add support for Z` → `feature`; `speed up the W path` → `improvement`; `rotate the staging cert` → `task`). Omit if ambiguous. Do not ask. Do not infer from tone. |
| `jekt/priority` | Write only if the user explicitly stated a priority. The single exception is an unambiguous blocker signal (`this is blocking [foo-12]`, `stop everything for this`) → `blocker`. Do not infer from tone. Do not ask. |
| `author` | Write only if the ticket describes someone else's report or statement — the user is filing on another person's behalf. Do not ask. |
| `creator` | Write only if the person who created this ticket record differs from whoever Git will attribute it to. Rare at creation time; do not ask. |
| `jekt/assignee` | Write only if the user explicitly assigned. Do not ask. |
| `jekt/status` | Never write at creation. An absent `jekt/status` is operationally open. |
| `jekt/resolution` | Never write at creation. |
| `jekt/releases` (or `tags.lst`, or any other list property) | Never write at creation. |
| All other properties | Never write at creation. |

### Format

```yaml
jekt/kind: feature
jekt/priority: high
jekt/assignee: jdoe@example.com
author: asmith
```

- Bare unquoted property keys (the slash does not require quoting).
- No `null`, `~`, or empty-value sentinels. Omit absent properties.
- Block-form sequences on output if any list property were written (none is written at creation per the rules above).

## 10. Boundaries

This procedure does **not**:

- Create a working branch. Branch creation is owned by Start Ticket; ticket creation does not imply starting work on it.
- Bootstrap a plan, minutes, summary, or any other ticket artifact beyond the description.
- Delete or modify the source TODO, if any. Removing it once the work covers it is a separate concern; a procedure for the various actions involved in finishing a ticket is anticipated.
- Initialize `.jekt/` in a repository that lacks it; recommend Initialize Jekt instead.

Commit handling is out of scope; Commit owns it. The new files are left unstaged for the user to commit when ready.

## 11. Completion

After writing the files, report the result, then close with the next actions as §12 describes. Keep both brief.

### Result report

State concisely:

- The assigned ticket ID, bracketed as a reference; preserve its casing unless repository prose already has a clear different convention.
- The ticket directory path.
- Whether `description.yaml` was written, and if so, which properties were set.
- Whether a property the user might have expected was omitted (e.g., "no kind set — ambiguous from content"), so the user can correct.
- If the repository is under Git, that the new files are not committed.

## 12. Closing Footer

End the reply with a brief statement making the user aware of the most likely next actions in Jekt that they can request, building on what the reply has already reported. It tells the user which step is most appropriate now and gives someone new to Jekt a sense of what is possible. Mention an action only when it serves a real purpose at this point; when none does, add nothing. The actions below, and when each tends to apply, are for deciding which to present.

- Starting work on the ticket, which puts it on its own branch.
- Creating other tickets first.

For example, after creating a ticket: "You can now start the ticket, or create other tickets first." The example illustrates length, tone, and how actions combine; it is not a template.

## 13. Anti-Patterns

Do not:

- Silently fall back from coordinated to local mode without informing the user.
- Use `--force-with-lease` for the claim push.
- Write any `description.yaml` property whose rule above says "never write at creation".
- Write `description.yaml` at all when no property is asserted.
- Silently sanitize a malformed label or slug. Report and stop or ask.
- Include speculative solution options or step-by-step code changes anywhere in the description. (Endorsed direction, when genuinely warranted, belongs only in *Approach* — see §7.)
- Map out probable implementation steps in the description, even in service of being thorough.
- Link to a source TODO file in the same repository (the file is destined for deletion).
- Auto-delete the source TODO.
- Add a self-reference link definition for the ticket's own ID.
- Use a colon between the bracketed ID and the elaboration in the description heading (`# [foobar-123] Title`, not `# [foobar-123]: Title`).
