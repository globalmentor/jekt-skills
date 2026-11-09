---
modified: 2026-11-01
---

# Assess Plan

You are assessing an existing plan for an active Jekt ticket. A Jekt plan is the authoritative execution reference for a unit of implementation work, evolved iteratively through planning conversation. This operation reads the plan as a whole against the actual code and the ticket's stated end-state, both to verify the plan's claims and to surface risks the plan did not anticipate — gotchas, gaps, and consequences the planning conversation did not foresee. It reports findings under what is load-bearing for the plan and lays out the decision surface so the user can decide whether to **proceed**, **defer**, **restructure**, or **mitigate first**.

The plan need not be converged for this operation to apply. Assessment is available at any point in the iterative planning loop — useful mid-iteration as a check before continuing to refine, and useful after convergence as a final look before implementation.

The scope of this operation is producing a report in conversation. **This procedure is read-only.** It does not edit the plan, refine designs, append minutes, branch, commit, or write any file. Findings that imply downstream action are surfaced in the report; the user decides what to do next.

## 1. Operational Context

Establish before acting:

- **Repository root and `.jekt/`.** Operate against `.jekt/` in the default workspace root unless the request indicates another root. If `.jekt/` is absent, stop and recommend Initialize Jekt.
- **Ticket and target plan.** This procedure assesses an existing plan in an existing ticket. If no plan exists, stop and recommend Create Plan. If the ticket directory itself is absent, stop and recommend Create Ticket.
- **Multi-root workspace.** Files visible in other workspace roots belong to other repositories. Do not act against them.
- **Git is not required.** The operation is a local read. The branch the user is on is incidental.

## 2. Ticket and Plan Resolution

Identify the ticket and the plan to assess from the conversation. The conversation has been about some ticket's work and, in most cases, about a specific plan within it.

For the ticket: identify from the conversation; the current Git branch is one signal, the conversation is primary. If genuinely uncertain, ask.

For the plan: the default target is the most recently modified file in `plans/`. When the conversation has been explicit about assessing a different plan (by date, slug, or topic), assess that one. If genuinely uncertain, ask.

Identifier matching against existing ticket directories under `.jekt/tickets/` is case-insensitive on the project slug.

## 3. Pre-Assessment Context Gathering

Ground the assessment in actual project state. Read:

- **The plan being assessed**, in full.
- **The designs the plan references**, in full.
- **The ticket's description** — the end-state the plan is meant to reach. Use the Objective and Acceptance Criteria sections when present (the conventional form produced by Create Ticket); otherwise use whatever end-state the description conveys. If the description is absent, sparse, or does not convey an end-state sufficient to assess alignment against, treat that as an assessment constraint (surfaced under §5 plan-versus-ticket alignment), not as a reason to stop.
- **The code the plan would touch**, sufficient to test the plan's assumptions about current state against reality.

Sibling plans in `plans/` are read only when the target plan references them or the conversation has named them; the assessment is of one plan, not a multi-plan reasoning operation.

The conversation is the authoritative source for what the user is asking about. The plan file is the authoritative source for what the plan currently says. The actual code is the authoritative source for the state the plan would modify — verify against the actual code, not the conversation's or the plan's memory of it.

## 4. Assessment Discipline

The assessment is a structured second look at the plan. The discipline is:

- **Test the plan without re-planning it.** The assessment has two complementary tasks: verify the plan's claims against the code and the ticket, and probe for what the plan did not anticipate — unforeseen gotchas, gaps in coverage, downstream consequences the conversation did not surface, code realities the plan did not account for. Both are first-class. Probing for what the plan missed is not redesigning the work or proposing an alternative plan; it is identifying what an implementer would encounter that the planning conversation did not foresee. Where the plan reflects a decision the conversation settled, the assessment does not re-litigate the decision unless the code or the ticket's stated end-state contradicts it. Where the assessment surfaces a concern — whether by verification failure or by discovery — it names the concern; it does not substitute a different decision.
- **Filter candidate concerns silently before reporting.** Use your reasoning workspace to consider possible concerns, weigh them against the actual code and the ticket's end-state, and conclude whether each is real before drafting the report. The report contains conclusions, not deliberations. Candidates you considered and dismissed do not appear in the report — not as expanded entries, not as a tally, not as a count, not in any form. If a candidate resolves to "not a concern," it leaves no trace in the report.
- **State conclusions decisively.** A finding is something you concluded is real. Phrase it as a conclusion, not as a possibility you then walk back. If after analysis a concern resolves to "not a problem," it does not belong in the report at all.
- **Be guided by what is load-bearing for this plan**, not by a uniform checklist. Some plans will have substantial findings under code-drift; others under chunk coherence; others under risk concentration; others under nothing at all. Report what surfaced, not what a generic plan review would cover.
- **Open Questions and Assumptions in the plan are working state, not defects.** Their *existence* is the iterative loop's view of what is still under discussion; do not flag it as a failure. Their *substance* is in scope: if a specific open question blocks a chunk from being implementable, or if an assumption the plan made is contradicted by the code, surface that as a finding.
- **Iteration-stage neutral.** The plan may be mid-loop or converged. The assessment does not assume either; it reports what it finds. A plan with many open questions is not by that fact unfit; a plan with none is not by that fact safe.

## 5. What to Look At

The assessment considers — at the model's judgment of what is load-bearing for this plan — the following kinds of observation. Findings under any category may be ones the plan anticipated and the assessment verified or contradicted, or ones the plan did not consider at all and the assessment surfaces; both are first-class. The list is illustrative, not exhaustive, and not a checklist. Empty categories produce no findings; they are not reported.

- **Plan-versus-code reality.** Has the code drifted under the plan since it was written? Do referenced files, symbols, and call sites still exist in the form the plan assumes? Are assumptions about current behavior still accurate? Does the code reveal couplings, invariants, or constraints the plan does not account for?
- **Plan-versus-ticket alignment.** Does the plan, executed as written, reach the ticket's stated end-state? When the description provides Objective and Acceptance Criteria, assess against those specifically. When the description is sparse or absent, surface the gap as a finding (the assessment can compare the plan against the available end-state, but cannot confirm alignment that was never specified); do not invent criteria the ticket did not state. Are there parts of the stated end-state the plan does not address? Are there parts of the plan that exceed the stated end-state without justification?
- **Chunk coherence.** If the plan is chunked, do the chunk boundaries still hold up as semantically coherent intermediate states under the current code? If the plan is monolithic, does that finding still describe the work, or has the structure of the code shifted enough that boundaries now exist? Re-examining chunking on the existing plan is a useful pass distinct from the chunk analysis performed at plan creation.
- **Internal coherence.** Do Overview, chunk specifications, and step bodies agree? Do referenced designs in `designs/` exist and match what the plan says they say? Are there contradictions between sections?
- **Open Questions and Assumptions, substantively.** Which of the open questions, if any, must be resolved before the plan can be implemented? Which assumptions, if any, are contradicted or made fragile by the code? Are there assumptions the plan made implicitly — not recorded as Open Questions or Assumptions — that the code reveals as load-bearing?
- **Design content sitting in the plan.** When the plan body contains substantial design-grade reasoning that ought to live in a design document under `designs/`, surface this as a finding. **Do not extract it.** Recommend Update Plan to perform the extraction.
- **Risk concentration and unforeseen consequences.** Where does risk concentrate — shared-interface changes spanning modules; dependencies on external services; verification difficulty; coupling between chunks; reversibility cost of specific steps? What downstream effects of the plan's changes are not represented in any chunk — callers the plan did not enumerate, behaviors the plan changes incidentally, contracts the plan tightens or loosens without naming?
- **Test articulation realism.** Where the plan articulates how steps will be tested, does the team's standing approach as expressed in the plan actually apply to the code as it stands? Are there steps whose decision logic would be hard to test under the articulated approach without a structural change the plan does not name?

## 6. Scope and Reach

Independently of the findings under §5, the assessment carries a concrete description of the work's **shape and reach**, sufficient for the user to form their own effort judgment.

Describe the work in dimensional terms:

- **Surface.** Number of files touched; modules or packages spanned; concentration in one area vs. scatter.
- **Change character per site.** A few standalone lines; rename or signature change with mechanical propagation; method-body refactoring; class-level restructuring; cross-cutting API surface change; new module introduction.
- **Coupling.** Whether changes are independent or each step depends on prior steps; whether shared interfaces force coordinated change across compilation units.
- **Novelty.** Routine application of an existing pattern in the codebase vs. introducing a pattern the codebase does not already have; presence or absence of comparable prior work the implementer can lean on.
- **Verification cost.** Whether existing tests cover the surface, whether new test infrastructure is required, whether changes touch areas where verification is itself nontrivial.
- **Reversibility.** Whether each step is locally reversible or commits the codebase to a direction that is costly to back out.

**Do not state effort in time, story points, or any comparable scalar measure.** Time-based estimates from an LLM carry no calibration and mislead more than they inform. The dimensional description above is what supports the user's effort judgment.

## 7. Decision Surface

After the findings (§5) and scope description (§6), the report closes with a short **Decision surface** section. Its purpose is to lay out, on the basis of what the assessment found, what the user's options look like — without picking among them.

The decision surface frames the choice as one or more of:

- **Proceed.** The plan is implementable as written. Name any concerns that should be monitored during execution but do not block proceeding.
- **Defer.** Conditions favor waiting. Name what would change conditions (a dependency completes, a refactor lands first, the ticket's priority shifts) and what state the plan should be left in until then.
- **Restructure.** The plan's shape needs work. Name what restructuring would help (rechunk against the current code, split the plan, extract a design, narrow scope) and the update to the plan that would carry the change.
- **Mitigate first.** A specific risk warrants reducing before execution. Name the risk and the mitigation (often a smaller, separate piece of work) without prescribing how the mitigation is itself planned.

The assessment lays out which of these the findings point toward and what each would entail. The user makes the call. Do not present this as a recommendation between options; present the options as the assessment surfaced them.

If the findings are minimal and the decision surface is straightforwardly **proceed**, the section is a single short statement to that effect.

## 8. Produce the Report

After §3 reading and §4–§7 reasoning are complete, write the report directly to conversation. No files are written.

The report's structure:

1. **What was assessed.** The plan file (path), the designs read, the code area examined. One short paragraph.
2. **Findings.** Organized around the load-bearing findings that surfaced. Use the §5 category names as section headings when they fit the findings naturally; do not force the report into the §5 taxonomy or use it as a checklist of categories to fill. A finding may also stand on its own under a heading that names the concern directly. Each finding is a conclusion the assessment reached — stated decisively, grounded in the plan or code, with the implication for the user named.
3. **Scope and reach.** The dimensional description from §6.
4. **Decision surface.** The closing section from §7.

State what the report must convey; do not script the wording. The report adapts to what was found, not to a template. Empty sections are omitted entirely; sections do not appear as headings with "no findings" underneath.

**No-finding case.** If the plan reads clean against the code, alignment with the ticket holds, the chunking still works, and the decision surface is straightforwardly proceed, the report is short and says so explicitly — naming what was assessed, the scope and reach (which is always reportable), and a one-sentence proceed conclusion. Padding a clean assessment with manufactured concerns is the failure mode to head off.

## 9. Failure Handling

| Cause | Action |
|---|---|
| Referenced design files are missing from `designs/` | Surface as a finding (under internal coherence); do not stop. |

Resolution and existence failures (missing `.jekt/`, unresolvable ticket, absent ticket directory, no plan to assess, ambiguous plan) are handled by stopping in §1 and §2.

## 10. Closing Footer

End the reply with a brief statement making the user aware of the most likely next actions in Jekt that they can request, building on what the reply has already reported. It tells the user which step is most appropriate now and gives someone new to Jekt a sense of what is possible. Mention an action only when it serves a real purpose at this point; when none does, add nothing. The actions below, and when each tends to apply, are for deciding which to present.

- If the user takes up findings that change the plan, including design content to move into a design document: updating the plan.
- If the user decides to proceed: implementing the plan.
- When the assessment itself produced insights or changes of direction worth recording that the minutes do not already hold: updating the minutes so that they are captured.

Present each action as following from the user's decision on the decision surface (§7), not as a choice among its options; the other options, such as deferring, are left to the report.

For example, after an assessment with findings the user may want to apply first: "You can update the plan to take up these findings first, or, if you decide to proceed as it stands, implement it." The example illustrates length, tone, and how actions combine; it is not a template.

## 11. Anti-Patterns

Do not:

- List candidate concerns in the report and then conclude they are not concerns. Filter in the reasoning workspace; the report contains conclusions only. The walk-back form ("here is a potential issue… on reflection, this is fine") is the specific failure mode to avoid; if a candidate resolves to "not a concern," it does not appear in the report — not as an entry, not as a tally, not as a count.
- Produce a uniform-template report with every category present whether or not it surfaced findings. Empty categories are omitted entirely.
- Pad a clean assessment with manufactured concerns. The no-finding case is a legitimate and common outcome.
- Generate extra findings to make the assessment look thorough. A finding's validity test is whether it corresponds to a concrete observation about the plan, the code, or the ticket; report only findings that survived analysis.
- State effort in time, story points, or any scalar measure. Describe shape and reach; let the user form effort conclusions from the description.
- Re-plan, redesign, or propose an alternative plan. The assessor tests what is there; it does not substitute its own.
- Extract design content from the plan into a design file. Recognize when design content is sitting in the plan and recommend Update Plan; do not perform the extraction.
- Edit any file, including the plan and any design file, in response to findings. The assessor is read-only; downstream action is the user's move.
- Treat the existence of Open Questions and Assumptions as a defect. Their substance is in scope; their existence is the iterative loop's working state.
