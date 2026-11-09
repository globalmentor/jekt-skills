---
modified: 2026-11-01
---

# Start Ticket

You are beginning work on an existing Jekt ticket. The scope of this operation is making this ticket the working ticket: switching the Git branch to `tickets/{ticket-id}` (creating it from the default branch if necessary), setting `jekt/status: started` when appropriate, and surfacing the ticket's existing context for the work session. The same procedure handles three observable cases — first-time start, resume after a break, and switch from another ticket — as a single operation.

This procedure does not create the ticket, write minutes, create a plan, commit, push, or modify any artifact other than (optionally) `jekt/status` in `description.yaml`. See §13. If the ticket is not observable from any ticket source, stop and recommend Create Ticket.

## 1. Operational Context

Establish before acting:

- **Repository root and `.jekt/`.** Operate against `.jekt/` in the default workspace root unless the request indicates another root. If `.jekt/` is absent, stop and recommend Initialize Jekt; do not initialize here.
- **Git presence.** This procedure requires a Git working tree. If the working directory is not under Git, stop and report; the branch operation cannot proceed without it.
- **Multi-root workspace.** Files visible in other workspace roots belong to other repositories. Do not act against them, and do not start a ticket that resolves only in a non-default root without the user naming that root explicitly.

## 2. Request Parsing

Inspect the request for two independent elements; either may be absent.

| Element | Recognized forms | Meaning |
|---|---|---|
| **Ticket identifier** | `foobar-123`, `FOOBAR-123`, `start FOOBAR-123` | The ticket to start work on. |
| **Reopen intent** | `reopen`, `restart work on`, `start work again on`, `resume the closed ticket` | The user explicitly intends to reopen a closed or resolved ticket. |

Recognition is by natural-language match, not regex; the forms above are canonical examples, not the only acceptable phrasings.

## 3. Ticket Resolution

Determine which ticket the user is starting work on. Use the first matching rule:

1. **Explicit identifier** from §2 — use the identifier as given.
2. **Current branch** — if the current Git branch matches `tickets/{ticket-id}`, the user is resuming that ticket; no branch switch will be needed in §7.
3. **Clear conversational intent** — if the conversation makes it clear which ticket the user wants to start work on (a ticket just created in this conversation, a ticket whose work the user has explicitly said they want to begin), use that ticket.
4. **Otherwise, ask.** A mere recent mention of a ticket is not sufficient; ambiguity warrants asking rather than guessing. The wrong-target failure here is recoverable (`git switch -` and start the intended ticket) but visible.

Locate the existing ticket source the reference identifies. A ticket may be observable from the current working tree (`.jekt/tickets/{ticket-id}/`), a local ticket branch (`tickets/{ticket-id}`), or a known remote-tracking ticket branch (`origin/tickets/{ticket-id}`). Identifier matching is case-insensitive on the project slug; the actual identifier from the selected source is the form used for the branch. If no ticket source exists, stop and recommend Create Ticket for the ticket. Do not create the directory implicitly.

## 4. State Detection

Read-only; nothing is created or modified in this section. **No network access:** remote-tracking refs (`origin/...`) are consulted only insofar as the local repository already knows about them. Do not run `git fetch`. See §15.

Before running the first detection command, briefly explain in user-facing terms what the checks do: locating the ticket in the current working tree or known ticket branches, reading its current `jekt/status` from that source, and resolving the default branch and base ref only if a new ticket branch must be created. Note that these checks do not modify project files, the index, the current branch, or any remote, and that no `git fetch` is performed.

Use the agent's workspace file-existence and file-reading facilities for checks against the working tree; use Git directly for checks against branches, refs, and the contents of a non-HEAD tree. For yes/no questions about Git state, prefer command forms whose exit status is the answer over forms whose output must be parsed. When a read-only check has equally good command choices, prefer one whose every form is read-only (e.g., `git rev-parse --abbrev-ref refs/remotes/origin/HEAD` rather than `git symbolic-ref refs/remotes/origin/HEAD`), so a user looking up the command sees a command that cannot mutate anything rather than one whose safety depends on which form was invoked. Do not reach for shell file utilities (`test`, `ls`, `stat`, `find`) for working-tree checks the workspace facility can answer.

Detection is sequenced by dependency, not batched. Evaluate the source-detection sub-questions in step 1 in precedence order and stop at the first match. Do not perform step 3 (default branch and base ref) until step 1 has selected a source; it is required only when that source is the working tree, and is skipped otherwise. Batching step 3 into the same invocation as step 1 defeats the gate.

1. **Ticket source.** Locate the first matching source in the precedence below; stop at the first match. The selected source supplies the ticket's actual identifier spelling and the branch action. For branch and ref sources, the source is valid only when its tree contains `.jekt/tickets/{ticket-id}/`. Identifier matching is case-insensitive on the project slug.

    | Ticket source | Branch action |
    |---|---|
    | Current `tickets/{ticket-id}` branch | No branch switch |
    | Local `tickets/{ticket-id}` branch | Switch to it |
    | Known `origin/tickets/{ticket-id}` remote-tracking ref | Create a local tracking branch |
    | Current working tree `.jekt/tickets/{ticket-id}/`, with no ticket branch known | Create the ticket branch from the base ref resolved in step 3 |

    If no source matches, stop and recommend Create Ticket. Do not create the directory implicitly.

2. **Ticket metadata.** Read `description.yaml` from the selected source if it exists. For the current-tree source, read from the working tree. For a branch or ref source, read the file from that source's tree (Git is the right tool for non-HEAD tree contents). A missing metadata file means `jekt/status` is absent, not that the ticket source is invalid. The current `jekt/status` value (or its absence) governs §5; other properties are surfaced in the context load (§9) and may be mentioned in the report (§11).

3. **Default branch and base ref (only when step 1 selected the working-tree source).** Skip this step otherwise. Resolve the project's default branch name by trying, in order: the configured default on origin (`refs/remotes/origin/HEAD`); a local branch named `main`; a local branch named `master`. Then resolve the base ref passed to `git switch -c`: prefer the local default branch when it exists; otherwise use the corresponding remote-tracking ref `origin/{default}`; otherwise ask the user which ref to use as the base.

## 5. Status Decision

Decide what to do with `jekt/status` based on its current value. `jekt/resolution` does not influence this decision.

| Current `jekt/status` | Action |
|---|---|
| absent or `open` | Will set `jekt/status: started` in §8. |
| `started` | No change; no write. |
| `closed` or `resolved` | If explicit reopen intent was parsed in §2, will set `jekt/status: started` in §8. Otherwise stop and ask whether to reopen the ticket; do nothing else until the user answers. |
| any other value (team-custom, e.g., `blocked`, `in-review`) | Leave alone; no write. Proceed with the branch operation and context load. |

When stopping for reopen confirmation, name the ticket's current `jekt/status` value and ask whether to reopen. Do not enumerate the next steps the procedure would take if the user confirms; the user answers when ready.

## 6. Disclosure

Before performing the branch operation or status write, the user needs to understand what is about to happen, what files or refs it touches, and how to reverse it. Cover this as part of the assistant's normal speech to the user — integrated into the response, not bracketed as a separate labeled step. Do not announce that you are about to convey it; convey it. The substance to cover:

- The branch action that will be taken: no-op (already on `tickets/{ticket-id}`), switch to an existing local branch, create a local tracking branch from `origin/tickets/{ticket-id}`, or create a new branch from the resolved base ref. Name the branch by its `tickets/{ticket-id}` form.
- For new-branch creation, the base ref the new branch will be based on. Note that the procedure does not fetch; if the user wants a fresh remote view of the default branch, they may `git pull` and start the ticket again.
- The `jekt/status` change, if any, per §5. For a reopen, mention that `jekt/resolution` (if set) is left unchanged so the user can choose to clear it.
- The cancellation affordance: the user can interrupt now; after the operation, `git switch -` returns to the previous branch, and (for a newly created branch) `git branch -D tickets/{ticket-id}` removes the branch.

Act in the order §7 → §8 → §9 in the same turn without waiting for a separate confirmation. The discipline is *disclose then act*.

## 7. Branch Operation

Perform the branch action determined in §4. Trust Git: do not pre-check working-tree cleanliness. If Git refuses the operation, surface its error directly and stop — do not proceed to §8 or §9.

| Ticket source | Command |
|---|---|
| Current `tickets/{ticket-id}` branch | None. Proceed to §8. |
| Local `tickets/{ticket-id}` branch | `git switch tickets/{ticket-id}` |
| Known `origin/tickets/{ticket-id}` ref | `git switch --track -c tickets/{ticket-id} origin/tickets/{ticket-id}` |
| Current working tree `.jekt/tickets/{ticket-id}/` | `git switch -c tickets/{ticket-id} {base-ref}` |

Use `git switch` for entering an existing branch and `git switch -c` for creating from an explicit base. Do not use `git checkout`. Naming the base ref explicitly on create avoids depending on the current branch and on Git's ambiguous-remote auto-creation behavior.

After any branch switch or branch creation, verify that `.jekt/tickets/{ticket-id}/` exists in the resulting working tree before proceeding to §8. If it does not, report the partial state and stop; do not create the ticket directory.

## 8. Status Write

If §5 determined that `jekt/status` should be set to `started`, write it; otherwise skip this section.

- If `.jekt/tickets/{ticket-id}/description.yaml` exists, modify it in place using the structured file-editing facility to set `jekt/status: started`. Preserve every other property exactly, including unknown properties under any namespace.
- If `.jekt/tickets/{ticket-id}/description.yaml` does not exist, create it containing the single line `jekt/status: started`.

Do not delete the file and recreate it. Do not use shell redirection (`> description.yaml`, `cat > description.yaml`) to replace the file. Both lose unknown properties, comments, and preserved ordering, and both are unrecoverable from the next read.

Never modify any property other than `jekt/status`. In particular, do not touch `jekt/resolution`, even when reopening a ticket.

## 9. Context Load

After the branch operation and any status write, surface the ticket's existing context. Skip any artifact whose contents are already evident in the current conversation — for example, the description that was just produced by Create Ticket earlier in the same session. The goal is to give the work session what it needs to proceed, not to reproduce material the user can already see.

Load in this priority order; stop when the user has what they need:

1. The prose description (`description - {Label}.md`) — Objective, Acceptance Criteria, any Constraints or Approach.
2. `description.yaml` — kind, priority, assignee, status, and any other recorded scalar properties.
3. `minutes.md` — full file if short; otherwise the most recent entries plus a count of earlier entries.
4. The most recent plan in `plans/`, if any.
5. The presence (not contents) of `designs/`, `comments/`, and related-ticket reference definitions in the description. Read on demand only.

Surface the load as a brief synthesis in the chat response. Nothing is written to file.

## 10. Failure Handling

Classify failures distinctly. The branch operation either succeeds in full or fails before changing branches; the status write occurs only after a successful branch operation and may fail independently.

| Cause | Action |
|---|---|
| `.jekt/` absent | Stop before §4. Recommend Initialize Jekt. |
| Not under Git | Stop before §4. Report the requirement. |
| No current-tree ticket directory, local ticket branch, or known remote-tracking ticket branch matches the requested ticket | Stop after §4 step 1. Recommend Create Ticket for the ticket. |
| Closed or resolved ticket without explicit reopen intent | Stop after §5. Ask whether to reopen. |
| Base ref unresolved when a new branch must be created | Stop after §4 step 3. Ask which ref to use as the base. |
| `git switch` refused (uncommitted changes that would conflict, untracked files that would conflict, detached HEAD, in-progress rebase or merge, etc.) | Stop after §7. Surface Git's error directly. Do not attempt to interpret or work around it. |
| Ticket directory absent after a successful branch switch or branch creation | Report the partial state: the branch operation succeeded but `.jekt/tickets/{ticket-id}/` is absent in the resulting working tree. Do not write status or create the directory. |
| `description.yaml` write failure (permissions, etc.) after a successful branch operation | Report the partial state: the branch was switched but the status write failed. Recommend the user retry the status write manually or start the ticket again. |

## 11. Result

After the operation, report briefly. The audience is a user about to begin work on the ticket; favor signal over commentary.

A useful report covers:

- **Branch outcome:** switched in place (no-op), switched to existing branch, created a local tracking branch from a remote ref, or created and switched to a new branch from the resolved base ref. Name the branch.
- **Status change, if any:** what was, what is now. If the ticket was reopened, mention that `jekt/resolution` (if set) was left unchanged and recommend the user clear it manually if it no longer applies.
- **Missing description, when relevant:** if the ticket has no description prose file (`description - {Label}.md` absent), note this prominently and suggest the user fill in a description before substantive work begins. A `description.yaml` containing only the just-written status does not count as a description.
- **Ticket synthesis from §9:** a brief paragraph or short list capturing what the ticket is about, the most recent minutes highlights, and the most recent plan if any. Skip artifacts that were skipped in §9 because they were already evident in conversation.
- **Closing footer** (see §12).

Where appropriate, mention how to reverse the branch operation (`git switch -` to return to the previous branch; `git branch -D tickets/{ticket-id}` to remove a newly created branch). This is teaching, not instruction; do not belabor it.

If the operation was a pure no-op (current branch already matched the ticket and `jekt/status` was already `started`) *and* the ticket's context is already evident in the current conversation, keep the report to a single short line stating the state — for example, "Already on `tickets/foo-43`; status `started`; nothing to do." Otherwise, still surface the §9 context load even though no operational change occurred; resuming a long-dormant ticket in a fresh session is exactly when the context load is most needed.

## 12. Closing Footer

End the reply with a brief statement making the user aware of the most likely next actions in Jekt that they can request, building on what the reply has already reported. It tells the user which step is most appropriate now and gives someone new to Jekt a sense of what is possible. Mention an action only when it serves a real purpose at this point; when none does, add nothing. The actions below, and when each tends to apply, are for deciding which to present.

- When the ticket has no plan: discussing the approach and capturing it in a plan, or, for a small, well-defined change, implementing it directly.
- When the ticket has a plan in progress: continuing it as its state calls for, whether settling open questions and updating it, assessing it, or implementing it or its remaining chunks.
- Updating the minutes as the work proceeds, so that decisions and other significant events are captured.

For example, after starting a ticket that has no plan: "Since the ticket has no plan yet, you might start by discussing the approach and then creating a plan from that discussion; you can update the minutes along the way, so that decisions are captured." The example illustrates length, tone, and how actions combine; it is not a template.

## 13. Boundaries

This procedure does **not**:

- Create the ticket directory or any artifact within it beyond the optional `jekt/status` write to `description.yaml`.
- Append a minutes entry for the start event itself (per the Update Minutes procedure's discipline). Significant events that arise during the session are captured later by Update Minutes.
- Modify any property other than `jekt/status`. `jekt/resolution` is read as part of general metadata but is never written, even when reopening a ticket.
- Initialize `.jekt/` if absent; recommend Initialize Jekt instead.

## 14. Anti-Patterns

Do not:

- Refuse to start because the user is not on the default branch. Git decides what switches are allowed; this procedure does not impose a starting-branch requirement.
- Pre-check working-tree cleanliness and refuse on `git status --porcelain` non-empty output. Let `git switch` decide and surface its error if it refuses.
- Use `git checkout` when `git switch` or `git switch -c` is appropriate.
- Branch a new `tickets/{ticket-id}` implicitly from the current branch; always pass the resolved base ref explicitly to `git switch -c`.
- Rely on Git's auto-creation from an ambiguous-remote tracking branch; name the remote ref explicitly with `git switch --track -c tickets/{ticket-id} origin/tickets/{ticket-id}`.
- Fetch from origin to "ensure a fresh base" or "check for the remote ticket branch." The user fetches and starts again if they want a fresh remote view.
- Delete and recreate `description.yaml`, or rewrite it via shell redirection. Modify it in place.
- Modify `jekt/resolution`, even when reopening a ticket.
- Push the newly created branch.

## 15. Network Access

This procedure does not access the network. Branch operations and source detection use local refs and the working tree; remote-tracking branches (`origin/...`) are consulted only insofar as the local repository already knows about them. If the user wants a fresh remote view before starting, they `git fetch` and start again. The procedure does not perform the fetch; routine network access in this procedure would introduce auth prompts, latency, and offline-mode failures that do not fit the operation's profile.
