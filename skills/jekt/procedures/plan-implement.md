---
modified: 2026-11-01
---

# Implement Plan

You are implementing the work a plan calls for. The plan may be a persisted Jekt plan under `.jekt/tickets/{ticket-id}/plans/`, or — for small scopes that did not warrant persistence — a concrete scope the user and you arrived at in conversation. The disciplines below apply to both forms.

This procedure is for **developer-initiated implementation**: the developer is present and reachable when implementation surfaces something the plan did not anticipate. Implementation initiated by an autonomous agent has different needs and is not the subject of this procedure.

## 1. Target Resolution

Identify what is being implemented, in this precedence:

1. **Explicit user direction.** If the user named specific chunks ("implement chunks 1 and 3"), a specific plan, or a conversational scope ("just do the copyright change we discussed"), the user's words are the target.
2. **A persisted plan is present in the current ticket's `plans/`.** Default to the most recently modified plan. The persisted plan is the safer default when present; if the user wanted a conversational scope instead, they would have named it.
3. **No persisted plan applies and the conversation has produced a concrete actionable scope.** Use the conversational scope. "Concrete actionable" means a specific, identifiable change — files or behaviors named, a specific outcome stated. "We should improve error handling" is not concrete; "change the copyright year in all Java file headers from 2025 to 2026" is.
4. **Ambiguous or exploratory.** Stop and ask.

Before implementing, state in one short sentence what you understood the scope to be (e.g., "Implementing Chunk 2 of `2026-02-04-token-rotation`" or "Implementing the copyright-year change from 2025 to 2026 across Java file headers"). Then proceed; the user can interrupt if wrong. This is not a confirmation step.

When implementing against a persisted plan, identify the ticket from the conversation; the current Git branch is one signal, the conversation is primary; ask if genuinely uncertain.

This procedure does not commit, stage, push, or switch branches. Whether and how to commit implementation output is the user's choice.

## 2. Read Before Implementing

Ground implementation in actual project state before writing code.

**Persisted plan.** Read the plan in full, including the chunk specification body (`## Chunk N Specification: {Topic}`) for each chunk being implemented. The chunk specification body is the implementation contract; the Overview bullet is an index entry, not a specification, and is never sufficient on its own. Read the designs the plan references for the chunks being implemented.

**Conversational scope.** Re-read the conversation's statement of the scope, not your summarizing recollection of it. The user's words are the contract.

In both cases, read the code the implementation will modify, sufficient to ground the work in actual current state rather than imagined or remembered state. Where the plan articulates how a step is tested, honor that articulation against the model's defaults; absent such articulation, the user's standing instructions govern.

## 3. Implementability Check

Before writing code, verify that the target scope is actually implementable from the plan as it stands. Two checks:

- **Open Questions and Assumptions blocking the target.** A plan may be persisted mid-iteration with Open Questions and Assumptions outstanding. Their mere existence is not blocking. The question is whether any of them is *load-bearing for the target chunks or scope*: would implementing the target require a decision the open question has not yet made, or rely on an assumption the code contradicts? If so, stop and surface the specific item; for a persisted plan, recommend Update Plan to resolve it; for a conversational scope, ask. Open Questions that concern chunks the target does not include are not blocking and do not stop implementation.
- **Chunk dependencies.** When the user named specific chunks, check whether the chunk specification body or the plan's chunk ordering names prior chunks as prerequisites. If a prerequisite chunk has not been implemented and the target chunk requires its outcome, stop and surface the prerequisite; do not silently bleed into chunks the user did not name. Implement only the named chunks; if the work cannot be done without out-of-target changes, that is a divergence to surface, not a license to expand the scope.

## 4. Implementation Discipline

- **The plan file is read-only during implementation.** Do not add checkboxes, progress markers, completion notes, "small" tidies, or records of decisions made during implementation. Completion state lives in Git; insights live in minutes; plan refinements belong to Update Plan. The plan stays as it was written so a future reader sees the contract that governed the work.
- **On substantive divergence, stop and surface.** If implementation reveals something the plan or conversational scope did not anticipate — a code constraint, a coupling that forces a different approach, an assumption contradicted by the actual code, a design decision the conversation did not settle that the work requires — pause and surface it. Do not silently adapt; do not paper over with an undocumented local decision; do not proceed by analogy to existing code in a direction the plan did not name. For a persisted plan, recommend Update Plan; for a conversational scope, ask the user how to proceed.
- **On a blocker, stop and consult; do not destroy work.** If implementation hits a roadblock — a failing test that resists explanation, an approach that won't converge, a tool that won't cooperate — stop and consult the user. Do not back out work-in-progress changes to try a different approach. The cost of a back-out is unrecoverable; the cost of stopping is a question. Describe the blocker concretely: what was attempted, what happened, the working hypothesis for why.
- **Use inline TODO markers for local undiscussed decisions and deferred work.** For a small decision the implementation must make in passing where a reasonable alternative exists, or for work the scope deliberately defers, leave a language-appropriate inline TODO marker at the point in the code where it applies — for example, `// TODO design discussion: chose X over Y because Z; revisit if W` in `//`-comment languages, or `# TODO cache the read values once the caching layer lands` in `#`-comment languages. Inline TODOs are a developer convenience in the code; they are not Jekt artifacts and this procedure does not hand them to any other procedure.

## 5. Completion Report

After implementation, report:

- **What was implemented**, by chunk (for persisted plans) or by scope element (for conversational scope).
- **What was skipped or left incomplete, if anything**, with a brief reason. If nothing was skipped, say so.
- **Inline TODO markers added**, so the user knows what to revisit.
- **Divergences surfaced**, if any, from the plan or the agreed scope.

The report adapts to the work, not to a template.

## 6. Closing Footer

End the reply with a brief statement making the user aware of the most likely next actions in Jekt that they can request, building on what the reply has already reported. It tells the user which step is most appropriate now and gives someone new to Jekt a sense of what is possible. Mention an action only when it serves a real purpose at this point; when none does, add nothing. The actions below, and when each tends to apply, are for deciding which to present.

- When the implementation produced decisions, insights, or changes of direction worth recording that the minutes do not already hold: updating the minutes so that they are captured; updating them before committing lets the commit include them.
- When the implementation diverged from the plan in a way the user may want to keep: updating the plan to absorb the divergence.
- When the implemented work is not committed: committing it, whole or in parts.

For example, after implementing a chunk that settled a decision: "Next you might update the minutes, so that the switch to per-session token caching is captured, and then commit part or all of the implementation." The example illustrates length, tone, and how actions combine; it is not a template.

