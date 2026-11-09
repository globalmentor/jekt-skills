---
modified: 2026-11-01
---

# Update Plan

You are updating an existing plan for an active Jekt ticket. The plan is the authoritative execution reference for a unit of implementation work, evolved iteratively through planning conversation. This operation applies the refinements that have arisen in conversation since the plan was last written (or last updated): refining decisions, resolving Open Questions, adding new steps or chunks when the conversation has produced them, and updating related designs.

The scope is editing the existing plan file at `.jekt/tickets/{ticket-id}/plans/YYYY-MM-DD-{slug}.md` in place, along with any design files under `.jekt/tickets/{ticket-id}/designs/` that the conversation has refined or newly produced. The operation does not create the ticket, append to minutes, commit, push, or alter the ticket's description.

## 1. Operational Context

Establish before acting:

- **Repository root and `.jekt/`.** Operate against `.jekt/` in the default workspace root unless the request indicates another root. If `.jekt/` is absent, stop and recommend Initialize Jekt.
- **Ticket and target plan.** This procedure updates an existing plan in an existing ticket. If no plan exists to update, stop and recommend Create Plan. If the ticket directory itself is absent, stop and recommend Create Ticket.
- **Multi-root workspace.** Files visible in other workspace roots belong to other repositories. Do not act against them.
- **Git is not required.** The operation is a local file edit under the ticket directory. The branch the user is on is incidental; the user commits the change (or not) by their usual means.

## 2. Ticket and Plan Resolution

Identify the ticket and the plan to update from the conversation. The conversation has been about some ticket's work and some specific plan within it.

For the ticket: identify from the conversation; the current Git branch is one signal but the conversation is primary. If genuinely uncertain, ask.

For the plan: in most cases the conversation has been refining the most recently modified plan in `plans/`, and that is the default target. When the conversation has been explicit about updating a different plan (by date, slug, or topic), update that one. If genuinely uncertain, ask.

## 3. Pre-Update Context Gathering

Ground the update in actual project state. Read the plan being updated and review related context (other plans in `plans/`, related designs in `designs/`, the ticket description, the code the plan touches) as relevant. Model judgment selects what is relevant.

The conversation is the authoritative source for the refinements to apply. The existing plan file is the authoritative source for the current state being refined. The actual code is the authoritative source for the state the plan modifies — verify against the actual code rather than the conversation's memory of it.

## 4. Update Discipline

The plan update is one step in the iterative planning loop: the user has discussed further, and this operation applies the refinements that emerged. The normal use is iterative, with successive updates converging the plan's Open Questions toward empty. This procedure is shaped to that loop throughout: §5 preserves the structural defenses that make the plan implementable from cold, §6 carries refined design content into the durable design artifacts, and §7 reports the iteration's progress on the **decided-vs-open axis** so the user can see what was resolved and what remains.

What this means for applying updates:

- **Apply only the refinements the conversation has produced.** Do not re-evaluate decisions the conversation did not address. Do not improve, restructure, or rewrite sections the conversation did not touch.
- **Supersession.** When the conversation has changed a decision the plan recorded, the new decision takes its place in the plan body, and the superseded decision moves to (or extends) **Rejected Alternatives** with the reason the conversation surfaced for the shift. Earlier rejected alternatives stay rejected unless the conversation explicitly resurrects them. Supersession is what preserves the plan's defensive context across iteration: a future reader sees the current choice *and* what was once chosen and why.
- **Resolved Open Questions move out of the section.** When the conversation has resolved an entry from **Open Questions and Assumptions**, the entry moves: into the body if it became a decision, into Rejected Alternatives if it was an alternative considered and rejected. The section converges toward empty as iteration progresses.
- **New content (chunks, steps, decisions) follows the same disciplines as Create Plan.** Best-from-scratch unless the user explicitly requested otherwise; the structural defenses (the bullet-never-sufficient discipline, the `## Chunk N Specification: {Topic}` heading form, the Overview-to-specification linking, no checkboxes anywhere) apply to additions the same way.
- **Fill incidental gaps by inference; surface the inference in the summary** (§7). Same posture as Create Plan: an unsettled detail is an Open Question entry, not grounds to ask the user.
- **When reading the code surfaces a constraint, coupling, or inconsistency the conversation did not address**, record it (as Risk, Open Question, or Assumption) and call it out in the summary. Do not block on it.

Edit the file directly using a structured editing facility; preserve sections the conversation did not address; do not regenerate the file from a compressed understanding of it.

## 5. Plan Structure Reminders

The plan body's structure must be preserved through update. The disciplines that apply:

- **Overview** as the navigational index; **chunk specification sections** (or step body sections for monolithic plans) as the implementation contract; conditional sections (**Open Questions and Assumptions**, **Rejected Alternatives**, **Skip / Do Not Touch**) as supporting content. The Overview is updated to reflect added, removed, or reworded chunks and steps.
- New chunks added during update use the `## Chunk N Specification: {Topic}` heading form and the Overview-to-specification linking shape used by the existing chunks.
- **No checkboxes** anywhere in the plan, ever — including additions and edits. This prohibition matters under update because the drift toward "let me add some progress tracking now that I'm editing the file" appears specifically during update, even from a session that did not add checkboxes during creation. Completion state lives in Git history and minutes, not in the plan.

## 6. Design Recognition and Creation

Designs are durable ticket-level artifacts; plans serve and refine them. During update, distinguish procedural content (implementation steps, file targets, decisions about how this work proceeds) from design content (invariant analysis, API surface decisions, interface relationships, architectural rationale, structural decisions that constrain multiple plans, explanations of *why the design is correct* rather than *what is being changed*). Procedural content belongs in the plan; design content belongs in a design document under `.jekt/tickets/{ticket-id}/designs/{Label}.md` — unless the reasoning is minimal (a sentence or two), in which case it may stay inline in the plan.

When design content from the follow-up conversation is substantial enough to stand as its own artifact, **write or update the design file as part of this operation**:

- **Existing design refined.** When the conversation has refined a design already in `designs/`, update that file in place: refine the sections the conversation addressed; preserve unchanged content; do not silently rewrite paragraphs the conversation did not touch. Refining an existing design is the common case under iteration, as the planning loop produces design refinements as it converges.
- **New design.** Write a new file at `.jekt/tickets/{ticket-id}/designs/{Label}.md` with heading `# [{ticket-id}] Design: {Topic}`. The label follows the label rules in [Jekt Definitions](../definitions.md). Creating a brand-new design during update is less frequent than refinement; it is appropriate when the conversation has surfaced design content for an aspect the original plan did not address.

Reference the design from the plan at the points the plan relies on it, using document-relative paths, following the reference syntax in [Jekt Definitions](../definitions.md). For a Markdown link to a design file, use the angle-bracketed destination form to handle the spaces in the label: `[Token Rotation Strategy](<../designs/Token Rotation Strategy.md>)`.

Do not invent a reference target for a design that does not exist and is not being written. If the plan refers to a design, that design must be present in `designs/` after this operation completes.

Multiple designs may be touched in a single update when warranted. Each new design is its own file; each existing design update is its own in-place edit.

## 7. Apply Updates and Summarize

Apply the updates to the plan file directly. If the conversation has produced or refined designs, write those changes as part of the same operation. Do not draft updates in the conversation and transcribe them to the file afterward; the transcription silently mutates content. The file is updated first; the conversation gets a projection of what changed.

After all files are updated, produce a brief summary in the conversation. The summary's primary purpose is to show the user what changed in this update and where the plan now stands on the **decided-vs-open axis**, so the user can see what was resolved by this iteration and decide what the next iteration needs to address. Structure the summary around that axis:

1. **What was resolved.** Decisions newly made, Open Questions resolved (moved into the body or to Rejected Alternatives), and prior decisions superseded (moved to Rejected Alternatives with the new choice taking their place). This is the iteration-loop's progress in this turn.
2. **What is now open.** The Open Questions and Assumptions remaining after this update, including any new ones surfaced by the discussion or by code reading. If the section is now empty because the iteration has converged, say so explicitly: "no open questions; planning has converged." That sentence is the iteration's terminal signal.
3. **Inferences this update made.** Where the update filled in detail by inference rather than recording an Open Question, name the inferences so the user can challenge any that do not match intent.
4. **Concerns surfaced from the code.** Constraints, couplings, or inconsistencies the actual code revealed that the conversation did not address.
5. **Operational notes.** The plan file updated; designs created or refined (by label, with a one-line indication of what was captured or refined).
6. **Closing footer** (§9).

Reference the plan file's structure (chunks, steps, section headings) so the user can refer to specific sections. Corrections from the user are applied as **edits to the file**, not as agreements in chat to be transcribed later.

State what the summary must convey; do not script the wording.

**No-op case.** If the conversation since the last plan-write has not produced any refinements to apply (the user has been exploring options that did not lead to decisions), report that briefly without writing: name the plan, note that no refinements were found in the conversation, and surface the current Open Questions so the user sees what is still on the loop.

## 8. Failure Handling

| Cause | Action |
|---|---|
| No plan exists under `plans/` to update | Stop. Recommend Create Plan. |
| A design file write fails after the plan was updated, or the plan update fails after a design was written | Report the partial state explicitly: name what was updated, what was not, and which references in the plan (if any) now point to absent designs. |

Resolution and subject-identification failures are handled by stopping in the earlier section where they are detected.

## 9. Closing Footer

End the reply with a brief statement making the user aware of the most likely next actions in Jekt that they can request, building on what the reply has already reported. It tells the user which step is most appropriate now and gives someone new to Jekt a sense of what is possible. Mention an action only when it serves a real purpose at this point; when none does, add nothing. The actions below, and when each tends to apply, are for deciding which to present.

- While the plan records open questions: settling them in further discussion and updating the plan again.
- Assessing the updated plan against the code and the ticket before implementing it.
- When the user is satisfied with the plan: implementing it, whole or by chunk.
- When the planning conversation produced decisions, insights, or changes of direction worth recording that the minutes do not already hold: updating the minutes so that they are captured.

For example, after an update that left no open questions, following significant discussion: "The plan is ready, so you can now implement it; it would be a good idea to update the minutes soon as well, before or after implementing." The example illustrates length, tone, and how actions combine; it is not a template.

## 10. Anti-Patterns

Do not:

- Re-evaluate decisions the conversation did not address. The update applies refinements; it does not redo the planning.
- Restructure, reword, or "tidy" sections the conversation did not touch.
- Add checkboxes, status markers, completion indicators, or a "Checklist" section to the plan during update. The drift toward "now that I'm editing, let me add some progress tracking" is the observed update-specific failure mode.
- Recommend a hypothetical design procedure in place of writing or updating a design document as part of this operation.
- Update older completed plans to keep them consistent with this update.
- Decide between updating in place and creating a successor plan. The request makes that choice; a new plan, when one would serve better, comes from Create Plan.
- Invent refinements where the conversation produced none. The no-op response (§7) is the right answer when the conversation has not produced anything to apply.
- **Curated work product, not curation log.** Include only what belongs in the plan and in any design it refines — the execution approach, design reasoning, and the work's process where the role calls for it. Do not narrate the curation: why a refinement was applied, what kind of artifact this is, what other artifacts will cover, or what operation comes next.
- **Leak local-system information into the plan or any design it refines.** Do not include absolute paths, paths outside the repository, machine or account names, home-directory references, environment values, or local tool configuration. The constraint is on information about the author's local system, not on the plan's subject matter — references to code paths and files inside the repository are part of the plan's job and not affected.
