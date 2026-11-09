---
modified: 2026-11-01
---

# Create TODO

When work comes up that would distract from the task at hand, set it aside in a TODO document, so that the task can proceed and the work can be picked up later without this conversation.

This procedure writes a Jekt TODO document, not an inline `TODO` comment in code. Write the file directly, without proposing it first. The procedure does not create tickets or branches, stage, commit, or edit other artifacts.

## 1. Operational Context

- **Repository root and `.jekt/`.** Operate against `.jekt/` in the default workspace root unless the request indicates another root. If `.jekt/` is absent, stop and recommend Initialize Jekt; do not create it.
- **Multi-root workspace.** Other workspace roots belong to other repositories; do not write to them. Work for another repository is still recorded here, where it came up.

## 2. Target

Record the TODO where the work came up, unless the request says where:

- **In a ticket:** `.jekt/tickets/{ticket-id}/todo - {Label}.md`, for the ticket whose work surfaced it: the one the conversation is about, or else the one the `tickets/{ticket-id}` branch identifies. If the ticket is genuinely uncertain, ask.
- **At the project level:** `.jekt/todos/{Label}.md`, when no ticket is involved. Create `.jekt/todos/` if it does not exist.

Check the target with the workspace file facilities, matching ticket directories case-insensitively on the project slug. If the ticket's directory does not exist in the working tree, stop and report it; do not create it. If a TODO on the same topic is already there, refine it in place rather than overwriting it or adding another.

## 3. Grounding

Identify the deferred work from the request, or infer it from the conversation; ask only if it cannot be identified.

Verify the names the TODO uses, such as files, classes, methods, and library facilities, against the actual files, including those in other workspace roots. Read to describe the work, not to start it.

## 4. Content

Open with the work itself as a direct objective ("Evaluate how to munge the widgets"), not a description of the document ("This TODO captures deferred work to …").

Then give whoever picks up the work, without this conversation or any knowledge of the current ticket and its discussion, what they need to continue where it was set aside:

- Why the work matters and what surfaced it.
- The research done and what it established. Leaving it out means redoing it without its context, if a later reader even knows it was needed.
- What was actually decided, stated as decided.
- What remains open, stated as open: questions, alternatives, and any direction under consideration. Do not present the discussion's current leaning as its outcome, or settle what the eventual ticket, its acceptance criteria, or its behavior will be.
- Pointers to relevant files, tickets, and external resources.

Synthesize rather than transcribe the conversation. The TODO is not a ticket and has no required sections; its length follows what is known about the work.

- **Curated work product, not curation log.** Include only what belongs to the deferred work. Do not narrate the curation: why something was included or left out, or what will become of the TODO.
- **No local-system leakage.** Do not include absolute paths, paths outside the repository, machine or account names, home-directory references, environment values, or local tool configuration. The constraint is on information about the author's local system, not on the TODO's subject matter. Relative paths use forward slashes (`/`).

## 5. Form

- **Label.** A short Title-Case handle naming the deferred work, following the label rules in [Jekt Definitions](../definitions.md): `todo - Extract Byte Utils.md`, or `Streaming Support.md` at the project level.
- **Heading.** `# [{ticket-id}] TODO: {Title}` in a ticket, `# TODO: {Title}` at the project level. The title may be longer than the label.
- **References.** Cross-ticket references use shortcut reference links with definitions at the bottom, as described in [Jekt Definitions](../definitions.md). A TODO in a ticket needs no definition for that ticket's own ID.
- **Markdown.** Semantic Markdown: backticks for code identifiers, file paths, and literal values. End-of-sentence punctuation. No line wrapping; one paragraph per line.

## 6. Report

Report briefly: the TODO written or refined and, if the repository is under Git, that it is not committed.

## 7. Closing Footer

End the reply with a brief statement making the user aware of the most likely next actions in Jekt that they can request, building on what the reply has already reported. It tells the user which step is most appropriate now and gives someone new to Jekt a sense of what is possible. Mention an action only when it serves a real purpose at this point; when none does, add nothing. The actions below, and when each tends to apply, are for deciding which to present.

- When setting the work aside was a decision worth recording that the ticket's minutes do not already hold: updating the minutes so that the decision is captured.
- Unless the TODO is for the current ticket's own work: creating a ticket from it when the work is taken up.

For example, after setting aside work that came up during a ticket: "You might update the minutes, so that the decision to defer the cache work is captured; when you're ready to take the work up, you can create a ticket from this TODO." The example illustrates length, tone, and how actions combine; it is not a template.
