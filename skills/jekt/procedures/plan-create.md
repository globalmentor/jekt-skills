---
modified: 2026-11-01
---

# Create Plan

You are creating a new plan for an active Jekt ticket. A Jekt plan is the **authoritative execution reference** for a unit of implementation work: written to file directly, ticket-scoped, structured so a future agent with degraded context can implement from it without consulting the conversation that produced it. The plan's primary value is **injection resilience** — it bridges the planning conversation and the implementation work across conversation summarization.

The scope of this operation is producing the plan file at `.jekt/tickets/{ticket-id}/plans/YYYY-MM-DD-{slug}.md`, along with any design files under `.jekt/tickets/{ticket-id}/designs/` that the planning conversation has produced or refined (§9). The operation does not create the ticket, append to minutes, commit, push, or alter the ticket's description.

## 1. Operational Context

Establish before acting:

- **Repository root and `.jekt/`.** Operate against `.jekt/` in the default workspace root unless the request indicates another root. If `.jekt/` is absent, stop and recommend Initialize Jekt.
- **Ticket directory.** The plan is written under an existing ticket directory. If the ticket cannot be located (§2) or its directory is absent, stop and recommend Create Ticket; do not create the ticket directory implicitly.
- **Multi-root workspace.** Files visible in other workspace roots belong to other repositories. Do not act against them, and do not write a plan into a ticket that resolves only in a non-default root without the user naming that root explicitly.
- **Git is not required.** The operation is a local file write under the ticket directory. The branch the user is on is incidental — the plan file lands in `.jekt/tickets/{ticket-id}/plans/` regardless. The user commits the plan (or not) by their usual means.

## 2. Ticket Resolution

Identify the ticket this plan is for from the conversation. The planning conversation has been about some ticket's work; that ticket is the target. The current Git branch is one signal; the conversation is the primary one — if the conversation is plainly about `bar-7` and the user is on `tickets/foo-12`, the plan is for `bar-7`. If the ticket is genuinely uncertain, ask.

Identifier matching against existing ticket directories under `.jekt/tickets/` is case-insensitive on the project slug.

## 3. Pre-Draft Context Gathering

Ground the plan in actual project state before drafting. Review the context relevant to the work being planned: the ticket's description, any existing plans in `plans/`, any existing designs in `designs/` that the work touches or refines, and the code the plan will modify. Model judgment selects what is relevant; the criterion is whether the source informs the plan to be drafted, not procedural coverage.

The conversation is the authoritative source for what was decided. The actual code is the authoritative source for the state the plan modifies — verify against the actual code rather than the conversation's memory of it.

## 4. Drafting Discipline

The plan is a persisted snapshot of the planning conversation at whatever maturity point the plan is created. The normal use is **iterative**: the user discusses, this procedure captures what is decided and surfaces what is open, the discussion continues, Update Plan applies the refinements, and the cycle repeats until no open issues remain. A fully-converged "one-shot" plan is the same pattern with the conversation already complete the first time this procedure is followed — not a separate mode. This procedure is shaped around the iterative case throughout: §6 names **Open Questions and Assumptions** as a load-bearing section of the plan body, and §10 structures the summary around the **decided-vs-open axis** so the user can see what the next discussion turn needs to address.

What this means for drafting:

- Carry decisions from the conversation forward as decisions. Do not re-evaluate them. Carry rejected alternatives forward as rejected, with the reasons that surfaced in the conversation.
- Record what the conversation has not addressed as **Open Questions and Assumptions** (§6). An unsettled detail is not a reason to ask the user; it is a reason for an Open Question entry. Open Questions are the channel through which the iterative loop sees what is left to discuss.
- For details the conversation genuinely did not address: when recording them as an Open Question would be noise and the inference from context and code is obvious, fill them in by inference and surface them in the conversation summary (§10) so the user can confirm or correct them.
- Ask the user only when the conversation has not identified what work is being planned at any level of specificity. An unsettled detail is not this case; an unsettled subject is.
- Design the plan **from scratch** unless the user has explicitly asked otherwise. Do not introduce backward-compatibility scaffolding, deprecation shims, or vestiges of an earlier design that the user has not requested.
- When reading the actual code surfaces a constraint, coupling, or inconsistency the conversation did not address, record it in the plan (as a Risk, Open Question, or Assumption, as appropriate) and call it out in the summary. Do not block on it. When the code contradicts or would require materially revising something the conversation explicitly decided, stop before writing the plan and surface the conflict for discussion.

## 5. Plan File: Naming, Heading, Location

The plan file is written to `.jekt/tickets/{ticket-id}/plans/YYYY-MM-DD-{slug}.md`. Create the `plans/` directory if it does not exist.

- **Date.** Use the conversation-context date for `YYYY-MM-DD`. Do not invoke shell commands to obtain the date.
- **Slug.** Short kebab-case derived from the plan's topic; typically two to five words. If the user named a slug in the request, use it.
- **Collision.** If a plan file with the same date and slug already exists, disambiguate the slug. Do not overwrite an existing plan.
- **Heading.** The first line of the plan file is `# [{ticket-id}] Plan: {Plan Title}`. The plan title is in Title Case and names the plan's topic concisely.

## 6. Plan File: Structure

The plan is the authoritative execution reference. It must be usable by a future agent with no other context.

The plan body has a fixed shape: an **Overview** (prescribed); one or more **chunk specification or step body sections** (the implementation contract); and conditional sections that capture supporting content and the current state of the planning conversation. Among the conditional sections, **Open Questions and Assumptions** is load-bearing — it is the channel through which the iterative planning loop sees what is left to discuss, and its absence is the signal that planning has converged.

**Overview is an index, not a contract.** The plan opens with an `## Overview` section that lists the implementation steps as bare bullets, grouped under chunk headings when the plan is chunked (§7). The Overview is navigational only. **A step bullet is never sufficient on its own** — the implementer reads the step's full specification in the body before writing any code for it. Do **not** include checkboxes, status markers, completion indicators, or a "Checklist" section in the Overview or anywhere else in the plan.

**Chunk specification sections** (when the plan is chunked). Each chunk has a dedicated body section headed `## Chunk N Specification: {Topic}`. The chunk identifier leads the heading because that is the search key an implementer uses to locate the contract for the chunk they are about to implement; a topic-only heading reads as supplementary reference and invites the implementer to skip it. In the Overview, each chunk heading is a Markdown link to its specification section. Example:

```markdown
## Overview

- **Chunk 1: Rename** ([specification](#chunk-1-specification-rename); independent)
  - Step 1: Pattern matching instanceof (~40 sites, 19 files)
  - Step 2: Update call sites
- **Chunk 2: Core changes** ([specification](#chunk-2-specification-core-changes); requires Chunk 1)
  - Step 3: New signature
  - Step 4: Wire through callers
```

**Monolithic plans.** When no chunk boundaries exist (§7), state that explicitly in the Overview — for example, "This plan is monolithic; the work was not decomposable into independent chunks." Step bodies still live in dedicated sections (e.g., `## Step N: {Topic}`); the bullet-never-sufficient discipline still applies.

**Step content** lives in the body of the chunk's specification section (or, for monolithic plans, in the step-headed body sections). Each step's body contains the implementation detail: file targets, code examples for the chosen approach, edge cases, test articulation (§8). Include enough specificity that a future agent can implement the step from the section alone.

**Open Questions and Assumptions** is recorded as an `## Open Questions and Assumptions` section when the conversation has left any design decision unsettled or any detail unspecified that the plan wrote past. In iterative workflows it is the load-bearing section: the user reads it each time the plan is written or updated to see what the next discussion turn needs to address; the goal of iteration is to converge it toward empty. Each entry is specific enough that the next discussion can resolve it — a question to answer, an assumption to confirm or refute, a deferred choice with its alternatives. When the conversation has fully converged, the section is absent; its absence is the convergence signal, not an omission.

**Rejected Alternatives** is recorded as a `## Rejected Alternatives` section when the conversation considered alternatives and discarded them. Each entry names the alternative and the reason it was rejected. This is defensive context: a future agent (or human) returning to the work does not re-derive and re-choose a rejected approach. Omit the section when no alternatives were considered; do not pad with hypotheticals.

**Skip / Do Not Touch** is recorded as a section when the conversation produced explicit boundaries — files, modules, behaviors, or interfaces deliberately out of scope. Omit when empty.

**References to existing artifacts** (designs the plan follows, related tickets, code locations) appear inline at the points they apply, not collected in a separate section. Reference link targets for Jekt artifacts use document-relative paths (relative to the plan file itself), following the reference syntax in [Jekt Definitions](../definitions.md).

## 7. Chunk Analysis

Apply chunk analysis when the plan has steps that modify shared interfaces (method signatures, type definitions, template parameters) consumed by other steps **and** the changes span multiple compilation units or modules. Plans where every step is additive or modifies only internal behavior within a single module are typically one natural chunk; declaring the plan monolithic in such cases is the finding, not a failure.

A valid chunk boundary exists where:

- All modified code compiles.
- All existing tests pass (updated as needed within the chunk).
- The intermediate state is **semantically coherent** — no dead code, unreachable parameters, half-wired features, or misleading intermediate states.

Chunk boundaries are *discovered* in natural step ordering. Do not engineer chunks by introducing temporary intermediate states, artificial bridges, or backward-compatibility shims to manufacture commit points. If after brief examination no boundaries are apparent, the plan is monolithic, and that conclusion is the useful finding — it surfaces concentration of risk and coupling that the implementer needs to know.

## 8. Test Articulation

Articulate, with each step whose substance warrants it, how the resulting code will be tested under the team's standing test approach. Jekt is neutral on test philosophy; the team's standing instructions and the planning conversation are what establish whether internal methods are tested directly, whether mocking is used, what coverage matters, and so on. The plan reflects that approach as given, without imposing a Jekt opinion.

Test articulation lives **with the step** in its body section. Do not create a top-level "Test Strategy" section; a per-step note is concrete and resists generic platitudes. Trivial steps (rename, file move, simple delegation without decision logic) do not need test notes.

When articulating, if a step's decision logic would require mocking infrastructure to test (network clients, paginators, file systems), treat that as a design signal under whatever test approach the team uses: when the team's approach favors testing decision logic directly, name the extraction of that logic into a directly testable internal API as an explicit step in the plan. The plan reflects the team's chosen direction, not a Jekt preference for one over the other.

The purpose of recording the test approach in the plan is anti-drift: the plan is the durable touchpoint the implementer returns to across summarizations, where a recorded "test this internal method directly" or "exercise via the public API" decision has more standing than a standing instruction that the model has been ignoring for an hour.

## 9. Design Recognition and Creation

Designs are durable ticket-level artifacts. Plans **serve** designs — a plan advances some piece of the design; successive plans in the same area produce and refine the design as the work evolves. A design captures the structural reasoning that constrains this plan and the plans that follow it; the plan captures the procedural work to advance under that design.

During plan drafting, distinguish between:

- **Procedural content** — implementation steps, file targets, code examples, decisions about how this particular work proceeds.
- **Design content** — invariant analysis, API surface decisions, interface relationships, architectural rationale, structural decisions that constrain multiple plans, explanations of *why the design is correct* rather than *what is being changed*.

Procedural content belongs in the plan. Design content belongs in a design document under `.jekt/tickets/{ticket-id}/designs/{Label}.md`, *unless* the reasoning is minimal (a sentence or two), in which case it may stay inline in the plan.

When design content from the planning conversation is substantial enough to stand as its own artifact, **write it to a design file as part of this operation**:

- **New design.** Write a new file at `.jekt/tickets/{ticket-id}/designs/{Label}.md` with heading `# [{ticket-id}] Design: {Topic}`. The label follows the label rules in [Jekt Definitions](../definitions.md). The content is the design reasoning, organized for the artifact's own readability — not as an extraction transcript from the plan.
- **Existing design refined.** When the planning conversation has refined an existing design in `designs/`, update that design file in place: refine the sections the conversation addressed; preserve unchanged content; do not silently rewrite paragraphs the conversation did not touch.

In both cases, reference the design from the plan at the points the plan relies on it. For a Markdown link to a design file, use the angle-bracketed destination form to handle the spaces in the label: `[Token Rotation Strategy](<../designs/Token Rotation Strategy.md>)`. References by the design's label in prose are also appropriate where a link is not warranted.

Do not invent a reference target for a design that does not exist and is not being written. If the plan refers to a design, that design must be present in `designs/` after this operation completes.

Multiple designs may be touched in a single plan creation when warranted. Each new design is its own file; each existing design update is its own in-place edit.

## 10. Write the Plan and Summarize

Write the plan and any design files (new or refined) directly to disk. Do not draft the content in the conversation and transcribe it to the file afterward — the transcription is a regeneration from compressed understanding and silently mutates decisions. The files are authored first; the conversation gets a projection of what was written. All files (designs and plan) are written before the conversation summary is produced.

After all files are written, produce a brief summary in the conversation. The summary is a review prompt for the user, not a duplicate of the plan. Its primary purpose is to show the user where the plan currently stands on the **decided-vs-open axis** so the user can confirm what was captured and decide what the next iteration needs to address. Structure the summary around that axis:

1. **What is decided.** The decisions the plan now records from the conversation, with enough specificity that the user can confirm each is correctly captured. Corrections come back as edits to the file.
2. **What is open.** The Open Questions and Assumptions the plan recorded — the planning loop's view of what is still unsettled and what the next discussion turn needs to address. This is the spine of the summary in iterative use. If the section is absent because the conversation fully converged, say so explicitly: "no open questions; planning has converged." That sentence is the iteration's terminal signal.
3. **Inferences the plan made.** Where the plan filled in detail by inference (per §4) rather than recording an Open Question, name the inferences so the user can challenge any that do not match intent.
4. **Concerns surfaced from the code.** Constraints, couplings, or inconsistencies the actual code revealed during drafting that the conversation did not address.
5. **Operational notes.** The plan file written (path and slug); the chunk structure or monolithic finding; designs created or refined (by label, with a one-line indication of what was captured or refined).
6. **Closing footer** (§12).

Reference the plan file's structure (chunks, steps, section headings) so the user can refer to specific sections (e.g., "in Chunk 2 Specification, the alternative approach you described…"). Corrections from the user are applied as **edits to the file**, not as agreements in chat to be transcribed later.

State what the summary must convey; do not script the wording. The summary adapts to the plan, not to a template.

## 11. Failure Handling

| Cause | Action |
|---|---|
| Plan filename collides with an existing same-date plan | Disambiguate the slug; do not overwrite. |
| A design file write fails after the plan was written, or the plan write fails after a design was written | Report the partial state explicitly: name what was written, what was not, and which references in the plan (if any) now point to absent designs. |

Resolution and subject-identification failures are handled by stopping in the earlier section where they are detected.

## 12. Closing Footer

End the reply with a brief statement making the user aware of the most likely next actions in Jekt that they can request, building on what the reply has already reported. It tells the user which step is most appropriate now and gives someone new to Jekt a sense of what is possible. Mention an action only when it serves a real purpose at this point; when none does, add nothing. The actions below, and when each tends to apply, are for deciding which to present.

- While the plan records open questions: settling them in further discussion and updating the plan.
- Assessing the plan against the code and the ticket before implementing it.
- When the user is satisfied with the plan: implementing it, whole or by chunk.
- When the planning conversation produced decisions, insights, or changes of direction worth recording that the minutes do not already hold: updating the minutes so that they are captured.

For example, after creating a plan with no open questions: "Now you might want to assess the plan, or if you're satisfied with it, you can implement it." The example illustrates length, tone, and how actions combine; it is not a template.

## 13. Anti-Patterns

Do not:

- Re-derive decisions the conversation already settled, or re-evaluate alternatives the conversation rejected. The plan persists what was decided; the planning conversation is the source.
- Generate the plan in the chat first and transcribe it to the file afterward. The transcription silently mutates decisions. Write the file directly.
- Add checkboxes, status markers, completion indicators, or a "Checklist" section anywhere in the plan, in the Overview or elsewhere.
- Treat the Overview as the contract. The Overview is an index; the body is the contract.
- Recommend a hypothetical design procedure in place of writing or updating a design document as part of this operation. Design recognition and creation is an inline capability of this procedure; subtracting it in the name of "scope separation" or "anticipated procedure" deletes a working capability.
- Update older completed plans in `plans/` to keep them consistent with the new plan.
- Impose Jekt opinions on test philosophy. The team's standing instructions and the conversation establish the test approach; the plan reflects it.
- Pad the plan with sections that the content does not warrant — empty Rejected Alternatives, empty Skip / Do Not Touch, fabricated alternatives the conversation did not consider.
- Wait for the conversation to be "fully converged" before drafting. The plan can be created at any maturity point; what is open lands in Open Questions.
- **Curated work product, not curation log.** Include only what belongs in the plan and in any design it refines — the execution approach, design reasoning, and the work's process where the role calls for it. Do not narrate the curation: why a step or alternative was included or excluded, what kind of artifact this is, what other artifacts will cover, or what operation comes next.
- **Leak local-system information into the plan or any design it refines.** Do not include absolute paths, paths outside the repository, machine or account names, home-directory references, environment values, or local tool configuration. The constraint is on information about the author's local system, not on the plan's subject matter — references to code paths and files inside the repository are part of the plan's job and not affected.
