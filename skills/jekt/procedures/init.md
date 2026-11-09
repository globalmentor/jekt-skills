---
modified: 2026-11-01
---

# Initialize Jekt

You are setting up the current working directory to use Jekt. The scope of this operation is creating `.jekt/` and its orientation readme, and, depending on detected state and the user's request, declaring a project under `.jekt/projects/{slug}/` and creating the Jekt management branch (§8) on the remote. Nothing else is performed: no commits, no staging, no ticket creation, no host or forge configuration.

This procedure is idempotent. Each time it is followed, it addresses only what is missing or newly requested.

Most users initializing Jekt are encountering it for the first time. Reports should be thorough enough to teach what was done and why; the *enumeration* of next steps stays short.

## 1. Operational Context

Operate against the default workspace root unless the user's request names another root. Do not touch other workspace roots.

The remainder of this procedure assumes the current working directory is the operating root.

## 2. Request Parsing

Inspect the request for four independent elements; any combination MAY be present.

| Element | Recognized forms | Meaning |
|---|---|---|
| **Project label** | `the "Foo Framework" project`, `project "Foo Framework"`, `for "Foo Framework"`, `Foo Framework project` | The human-readable project name (becomes the description label, §7). |
| **Project slug** | `slug "foo"`, `slug foo`, `slug=foo` | The machine handle for the project (becomes the directory name under `.jekt/projects/`). |
| **Management-branch opt-out** | `without a management branch`, `skip the management branch`, `no management branch`, `local-only`, `local only` | The user does not want the Jekt management branch created on origin by this initialization. |
| **Readme opt-out** | `without a readme`, `skip the readme`, `no readme` | The user does not want the orientation readme (`.jekt/readme.md`) created by this initialization. |

Recognition is by natural-language match, not regex; the forms above are canonical examples, not the only acceptable phrasings. A request that names no project and declines nothing is the most common case and supplies none of the four.

## 3. State Detection

Before acting on what is detected, detect the following. Each detection is read-only; nothing is created or modified in this section.

Before running the first state-detection command, briefly explain the checks in user-facing terms: you are checking whether the directory is already under Git, whether it has an `origin` remote, whether the remote Jekt management branch exists, and whether local Jekt files or project declarations already exist. Mention that these state-detection checks do not modify project files, the index, the current branch, or any remote branch; the remote check may update Git's local `origin/jekt` tracking reference if the branch exists. If detection shows that initialization should create the management branch, that later remote-changing action is disclosed separately before it is performed.

Use the agent's workspace file-existence and file-reading facilities for checks against the working tree (steps 4–6 examine `.jekt/`, `.jekt/readme.md`, `.jekt/projects/`, and `.jekt/tickets/`); use Git for the Git-presence, remote, and management-branch checks (steps 1–3). For yes/no questions about Git state, prefer command forms whose exit status is the answer over forms whose output must be parsed. When a read-only check has equally good command choices, prefer one whose every form is read-only (e.g., `git rev-parse --abbrev-ref refs/remotes/origin/HEAD` rather than `git symbolic-ref refs/remotes/origin/HEAD`), so a user looking up the command sees a command that cannot mutate anything rather than one whose safety depends on which form was invoked. Do not reach for shell file utilities (`test`, `ls`, `stat`, `find`) for working-tree checks the workspace facility can answer.

1. **Git presence.** Is the working directory inside a Git working tree? (`git rev-parse --is-inside-work-tree`, or equivalent.)
2. **Remote presence.** If under Git, does a remote named `origin` exist?
3. **Management branch presence on origin.** If under Git with `origin`, fetch the branch named `jekt` with an explicit destination refspec so the remote-tracking ref is created or updated regardless of `origin`'s configured fetch refspec:

   ```bash
   git fetch origin +refs/heads/jekt:refs/remotes/origin/jekt
   ```

   The `+` permits non-fast-forward updates so a force-push to `jekt` on origin (which should be rare and protected against in team workflows) does not leave the local view stale. If the fetch fails because the remote has no `jekt` branch, record the branch as absent; this is not an error condition. If the fetch fails for a remote-access reason (network, authentication, permission, or similar), record the management branch state as unavailable with the failure reason; do not stop local initialization. When the state is unavailable, skip management-branch creation in §8 and report the recorded reason per §9.

4. **Local `.jekt/` and orientation readme presence.** Does `.jekt/` exist in the working directory? If so, does `.jekt/readme.md` exist?
5. **Declared projects.** If `.jekt/projects/` exists, list its child directories; their names are declared slugs. For each declared project, also record the project label if a `description - {Label}.md` file exists in that project directory. Do not infer a label from other files or repository context.
6. **Slugs in use.** If `.jekt/tickets/` exists, list its child directories and extract the slug portion of each `{slug}-{number}` directory name. A slug is *in use but not declared* when it appears here but not in step 5: in-use slugs are observable from `.jekt/tickets/`, while declared slugs are observable from `.jekt/projects/`.

## 4. Disclosure

Before performing any creation or push, the user needs to understand what you intend to do and why. Cover this as part of the assistant's normal speech to the user — integrated into the response, not bracketed as a separate labeled step. Do not announce that you are about to convey it; convey it. The substance to cover, in plain prose or short bullets:

- Each item that will be created (the `.jekt/` directory, the orientation readme `.jekt/readme.md` if absent and not opted out, the project declaration directory and its description file if applicable, the management branch on origin if applicable).
- For each, what it accomplishes for the user.
- If the management branch will be created, explicitly say that this is the one remote-changing setup action: it creates the branch named `jekt` on `origin`, enables coordinated ticket-claim allocation, does not touch project files, the index, or the current branch, and is reversible with `git push origin --delete jekt`.
- The cancellation affordance: the user can interrupt and ask again with the adjustment (e.g., `slug "X"` to choose a different slug; `without a management branch` to skip that creation). Reversibility is also noted where relevant — see the reversibility points listed in §6, §7, and §8.

Act in the order §6 → §7 → §8. The local steps (§6 and §7) do not depend on the remote; perform them first so that a later remote failure cannot block the local setup that initialization is for.

Then perform the actions in the same turn without waiting for a separate confirmation. The discipline is *disclose then act*, not *disclose, confirm, then act*.

Exception: if the user supplied a project label but no slug, derivation (§5) chooses a slug and surfaces it as part of this disclosure. No separate confirmation step is added.

## 5. Project Slug Derivation

When the user supplies a project label without a slug, derive a short candidate slug from the label and use it without asking when the candidate is obvious. A project slug is a compact project key, not a URL-style rendering of the full label; prefer a short mnemonic from the central product, component, or domain word over generic modifiers. Use hyphens only when they materially improve recognition. If no good short candidate is obvious, or the candidate is not lowercase ASCII letters, digits, and hyphens beginning with a letter, ask the user for the slug instead of inventing a long or awkward one.

The slug determines both the directory name (`.jekt/projects/{slug}/`) and the prefix for new ticket identifiers (`{slug}-{number}`). Changing it is cheap before any ticket has been created: rename the directory, or initialize again with `slug "X"` to declare additionally. Nothing has been committed by Jekt yet.

A slug derived from a label is always lowercase. An uppercase slug is used only when the user supplies it, whether by choice or to keep the casing of a migrated tracker.

## 6. `.jekt/` Creation and Orientation Readme

Create the `.jekt/` directory if it does not already exist. Do not create any subdirectory of it speculatively; all subdirectories of `.jekt/` are optional until they contain content.

If `.jekt/readme.md` does not exist (per §3 detection) and the user did not supply a readme opt-out (§2), create `.jekt/readme.md` with the exact content below:

```markdown
# Jekt

[**Jekt**](https://jekt.dev/) is a context architecture that stores tickets, plans, designs, minutes, and related working context as plain files in the repository. This directory holds that context.

The Jekt specification is stewarded by GlobalMentor, Inc. and published under the Creative Commons Attribution 4.0 International License (CC BY 4.0). "Jekt" and the Jekt logo are trademarks of GlobalMentor, Inc.
```

If `.jekt/readme.md` already exists, leave it untouched. Do not modify, refresh, or normalize it.

Do not create `.jekt/.gitkeep`, a placeholder `description.yaml`, or any other file at the `.jekt/` root beyond the orientation readme above. The `.jekt/` directory itself is not an entity in the Jekt sense; it is the container for entities.

Do not stage, do not commit, do not push anything related to `.jekt/`. The user's first content commit will sweep the directory into version control naturally.

**Reversibility:** `rm -r .jekt/`, or `rm .jekt/readme.md` to remove just the readme.

## 7. Project Declaration

If the user supplied a project slug (either directly or via derivation from a label per §5), declare the project:

1. Create `.jekt/projects/{slug}/` if it does not already exist.
2. If a label was supplied (or successfully derived from a label), write `.jekt/projects/{slug}/description - {Label}.md` with a minimal body:

   ```markdown
   # {Label}
   ```

   The label follows the label rules in [Jekt Definitions](../definitions.md). If the supplied label fails them, report the issue and stop; do not silently sanitize.

3. If no label was supplied (a slug-only request), write an empty `.jekt/projects/{slug}/description.yaml` so the entity directory has a Jekt-meaningful placeholder asserting that the project exists with no properties yet recorded.

If `.jekt/projects/{slug}/` already exists:

- If only a slug was supplied, leave the directory alone and report that the slug is already declared.
- If a label was supplied and a `description - *.md` file already exists with a *different* label, report the existing label and ask the user before renaming the file. Filename renames are Git-visible and affect any external link that targets the file (rather than the directory).
- If a label was supplied and no description file exists, write `description - {Label}.md`. Leave any existing `description.yaml` in place — it is an independent metadata artifact, not a placeholder for the prose, and removing it would discard whatever assertions it carries (including the open-world assertion that the entity exists).
- If the user supplies a different slug than the one already declared, create the additional declaration. Multi-slug coexistence is supported structurally, though not the typical case for first-time use.

**Reversibility:** `rm -r .jekt/projects/{slug}/`.

## 8. Jekt Management Branch

The Jekt management branch is the branch named `jekt` on `origin`. Its presence is the signal that coordinated ticket-claim allocation is enabled for the repository. It is created as an orphan branch (no shared history with `main`); claim refs are anchored to its tree.

Create the management branch on origin if **all** of the following hold:

- The working directory is under Git.
- A remote named `origin` exists.
- The management branch state is known and absent on origin (not present and not unavailable because of a remote-access failure).
- The user did not supply a management-branch opt-out (§2).

Procedure:

1. Create the empty-tree, parentless commit and push it to `refs/heads/jekt` on origin in a single guarded pipeline. The `&&` ensures that a failure to build the commit object (notably a missing `user.email` / `user.name`) does not fall through to a push with an empty SHA — which would otherwise be interpreted as a *deletion* push.

   ```bash
   commit=$(git commit-tree "$(git mktree </dev/null)" -m "Initialize Jekt management branch") \
     && git push origin "$commit:refs/heads/jekt"
   ```

   `git mktree </dev/null` produces an empty tree object. `git commit-tree` with no `-p` produces a parentless (orphan) commit on that tree. The push to `refs/heads/jekt` creates the branch on origin without ever touching `HEAD`, the index, or the working tree locally. A non-fast-forward rejection here would indicate the branch was just created by a concurrent initialization (rare in practice). On any failure, classify per §9 and report.

2. Update the local remote-tracking ref so subsequent operations see the new branch without a separate fetch:

   ```bash
   git fetch origin +refs/heads/jekt:refs/remotes/origin/jekt
   ```

After creation, recommend in the report (not as a separate action) that the user protect the `jekt` branch on the forge against force-push and deletion. Initialization does not configure forge-level protections; the recommendation is for the user to apply.

**Reversibility:** `git push origin --delete jekt` removes the branch entirely. There is no residue beyond what the user explicitly committed elsewhere.

If any precondition is not met, skip the operation and explain in the report which precondition was unmet and what feature consequently remains unavailable. See §10.

## 9. Failure Handling

Classify failures from remote-access operations (§3 fetch, §8 push) and local Git operations used for management-branch creation (§8 `git mktree` / `git commit-tree`). Remote-access and local Git failures stop only the management-branch path; local actions from §6 and §7 still proceed.

| Cause | Action |
|---|---|
| Branch does not exist on remote (fetch case only) | Not a failure; treat as branch absent and continue. |
| Non-fast-forward rejection (push case, ref already exists) | Treat the management branch as present. Re-detect; rerun the §3 fetch; report the existing branch in §10. |
| Local Git operation failure during management-branch creation (for example, missing `user.name` or `user.email`) | Stop the management-branch operation. Report the underlying Git error. If the cause is missing identity, recommend configuring Git identity and initializing again. |
| Network failure | Stop the management-branch operation. Report the cause; explain that the management branch could not be checked or created because the remote was unreachable, and that initializing again later will create it. |
| Authentication failure | Stop the management-branch operation. Report the cause; explain that local Jekt initialization is complete but the management branch requires successful authentication to `origin`. |
| Permission denied | Stop the management-branch operation. Report the cause; recommend the user check repository write permissions or the forge's ref-creation policy. |

Do not use `--force-with-lease` for any push in this procedure.

## 10. Reporting

After acting, report what was found, what was created, and what remains available to the user.

The audience for this report is almost always someone encountering Jekt for the first time. Err on the side of thoroughness in *explanation*. Keep *enumeration* of next steps short.

### Structure

A useful report covers:

1. **Repository state found.** Git presence; remote presence; management branch presence; existing `.jekt/` (if any); existing `.jekt/readme.md` (if any); declared projects (if any, including each slug and label when present); slugs in use but not declared (if any).
2. **Actions performed.** Each created path or branch, with a one-line explanation of what it accomplishes for the user.
3. **Actions skipped, and why.** Each unmet precondition (no Git, no remote, opt-out, etc.) paired with the feature it gates. Help the reader understand that Jekt has not failed; some surface area is unlocked progressively as preconditions accrue.
4. **Recommendations the user should apply manually.** Notably forge-level branch protection for `jekt` when that branch was created.
5. **Next steps** (the closing footer).

Where appropriate, mention how to reverse a creation (delete the `.jekt/` directory, delete a project declaration, delete the management branch with `git push origin --delete jekt`). This is teaching, not instruction; do not belabor it.

### Closing Footer

End the reply with a brief statement making the user aware of the most likely next actions in Jekt that they can request, building on what the reply has already reported. It tells the user which step is most appropriate now and gives someone new to Jekt a sense of what is possible; since the user is almost always new to Jekt, it may run to a few sentences. Mention an action only when it serves a real purpose at this point; when none does, add nothing. The actions below, and when each tends to apply, are for deciding which to present.

- When no project has been declared and the user has not yet been told how to declare one: declaring one by initializing again with the project's name and slug, so that tickets are anchored to a known slug.
- When a project is declared, or the user is content to let ticket creation infer or ask for a slug: creating tickets and then starting work on one.

When the repository has no tickets yet, the statement also explains that Jekt organizes work as tickets, each created for a piece of work and then started to work on it.

For example, after a complete first-time initialization: "Jekt work is managed with tickets. Now that you've initialized the repository, you might want to create tickets and then start work on one." The example illustrates length, tone, and how actions combine; it is not a template.

## 11. Boundaries

This procedure does **not**:

- Run `git init` or any other Git-repository-creation operation. If the working directory is not under Git, recommend `git init` in the report and proceed with the local-only portion.
- Stage, commit, or push any change other than the management-branch creation push to `refs/heads/jekt` on origin. The user's first content commit is their own concern.
- Create any ticket, plan, design, comment, minutes, summary, or TODO. Those are owned by their respective procedures; do not "helpfully" bootstrap a first ticket as part of initialization.
- Modify any file outside `.jekt/` and outside the management-branch push.
- Create `.gitkeep` or any other version-control or schema placeholder at the `.jekt/` root.
- Configure forge-level branch protection. The recommendation is reported; the action is the user's.
- Rename, undeclare, or remove a project slug. Renaming a slug after tickets accrue is a non-trivial operation that this procedure does not perform; the cheap pre-ticket rename is the user's via `mv` or by initializing again with `slug "X"`.

## 12. Anti-Patterns

Do not:

- Hunt for "soft signals" (the user sounds tentative, the remote URL is unfamiliar) to second-guess whether to create the management branch. The user's opt-out phrasings (§2) are the only override; absent them, create the branch.
- Initialize `.jekt/tickets/`, `.jekt/releases/`, or any other subdirectory speculatively.
- Write defaults into any `description.yaml` (project-level or otherwise). Open-world semantics: absence is absence of assertion.
- Silently sanitize a malformed label or slug. Report and stop or ask.
- Silently degrade from a Git-touching path to a local-only path on remote failure. Inform the user.
- Use `--force-with-lease` for any push.
- Suggest installing or configuring the skill or the host, or any other meta-step outside the working directory. That layer is upstream of this procedure.
