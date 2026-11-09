---
modified: 2026-10-29
---

# Jekt Definitions

## 1. Releases and Projects

| Artifact | Role |
|---|---|
| Releases (`.jekt/releases/{release-id}/`) | Delivery groupings of tickets; release identifier is arbitrary, commonly a version string (e.g., `2.0.3`) and, in multi-project repositories, conventionally slug-prefixed (e.g., `foo-2.0.3`) |
| Projects (`.jekt/projects/{slug}/`) | Declared projects; the directory's existence declares the slug, and the description files (if any) describe the project |

## 2. File System Layout

- A ticket exists when its directory exists. Every file inside is optional.
- `tickets/` is flat; ticket directories with different project slugs coexist as siblings under a single `.jekt/tickets/`. The project's identity is encoded in the ticket ID (`{slug}-{number}`), not in the path.
- A project slug is **declared** when `.jekt/projects/{slug}/` exists, and **in use** when any ticket directory matches `{slug}-{number}`. A slug may be in use without being declared — declaration is not a prerequisite for using a slug.
- Files at the ticket root not matching a Jekt-prescribed name or prefix are ad-hoc work products.
- **Prescribed filenames** (`description.yaml`, `minutes.md`, `summary.md`, `tags.lst`, `jekt/releases.lst`) are exact; do not vary them.
- **Description form, uniform across entities:** the prose description file is `description - {Label}.md` and the metadata file is `description.yaml`. Either is optional. When creating a prose description for any entity, write the labeled form; the bare form `description.md` is not used.
- **Labels:** Title Case; short (soft cap of 40 characters, ideally two to four words); letters, digits, spaces, and hyphens are the ordinary characters, and common abbreviations and code-style identifiers such as `IPv6` are acceptable. Never `/`, `\`, `:`, `<`, `>`, `"`, `|`, `?`, `*`, backticks, `$`, control characters, or invisible characters (except those that join or modify a visible character); avoid a leading `-`, `.`, or `~`, trailing whitespace, and a final `.`.
- A **comment filename** may be date-only when the precise time is unknown.

## 3. Identifier Format

```
ticket-id    = project-slug "-" number
number       = decimal integer (1+ digits)
```

The boundary between the project slug and the number is the **last hyphen** followed by an all-decimal suffix. A regex such as `^(.+)-(\d+)$` parses the form reliably even when the project slug contains internal hyphens.

**Casing:** the default is the canonical lowercase form (`foobar-123`) in paths, branch names, and Git refs. A project may instead use an uppercase slug, whether by choice or to keep the casing of a migrated tracker; declare it as used (e.g., `.jekt/projects/FOO/`) and keep that casing in every identifier. A reference in text may be written in either case; preserve the case its author used. Differently cased identifiers remain different.

**Project-slug guidance:** lowercase ASCII letters, digits, and hyphens; begin with a letter; short (commonly 3–8 characters).

## 4. Reference Syntax

Use CommonMark shortcut reference links.

| Subject | Form | Example |
|---|---|---|
| Jekt ticket | `[{ticket-id}]` | `[foobar-123]` |
| Person | `[@{name}]` or `[@{local}@{domain}]` | `[@jdoe]`, `[@jdoe@example.com]` |
| External Jira | `[{KEY-NUMBER}]` | `[AUTH-456]` |
| External GitHub issue | `[{owner}/{repo}#{number}]` | `[acme/widget#789]` |

- CommonMark reference-label resolution is **case-insensitive**.
- Reference targets for tickets point to the **ticket directory**, not a file, and are relative to the file that contains the definition: `[foobar-123]: ../foobar-123/` from a ticket's root, `[foobar-123]: ../../foobar-123/` from its `plans/`, `designs/`, or `comments/`. Targets MUST remain within the repository; absolute paths and paths that escape the repository root are prohibited.
- Multiple definitions for the same label: first wins.
- A person reference that resolves to no one (e.g., an imported mention of a removed source account) uses the reserved placeholder `[@?]` in place of an identifier.

**Self-reference rule:** do not warn about a missing definition for a ticket's own ID within its own files, do not require one, and do not insert one during edits.

### Headings

A heading or commit subject begins with the bracketed ticket ID. The bracketed ID is **never** followed by a colon. When a colon appears later in the heading, it separates a **kind token** (`Minutes`, `Summary`, `Plan`, `Design`, etc.) from a free-form elaboration of that kind. The description heading and ordinary commit subjects carry no kind token because the ID combined with the elaboration is already self-identifying. Minutes and summary carry one because they have no topic of their own; plans and designs take their topic as the heading, and may add a kind prefix when viewed apart from their directory.

```markdown
# [foobar-123] Auth Token Refresh         # description: heading title only, no kind token
# [foobar-123] Minutes                    # kind token, no elaboration
# [foobar-123] Summary: Token Rotation Strategy  # kind token + elaboration after the colon
```

## 5. Vocabulary

Vocabularies are **open**. Extension prefixes (`acme/...`, `github/...`, `jira/...`) are valid and must be preserved.

### `jekt/kind`

| Value | Meaning |
|---|---|
| `bug` | Defect correction |
| `feature` | New capability |
| `improvement` | Existing capability made better |
| `task` | Technical or operational work, not directly user-visible |

Default: none.

### `jekt/status`

Workflow stage label. Resolving a ticket sets `jekt/resolution` and moves `jekt/status` to `resolved`; closing it moves `jekt/status` to `closed`.

| Value | Meaning |
|---|---|
| `open` | Permitted but not needed; absence is operationally equivalent. |
| `started` | Work has begun. |
| `resolved` | The disposition is decided; conventionally set together with `jekt/resolution`. |
| `closed` | Finalized. Strongly recommended as the terminal stage. |

A ticket is operationally **open** if `jekt/status` is absent or any value other than `closed`, and operationally **resolved** if it is `resolved` or a later stage of the team's workflow. Teams MAY use other values.

### `jekt/resolution`

| Value | Meaning |
|---|---|
| `completed` | Work done as intended |
| `rejected` | Evaluated and deliberately declined |
| `invalid` | Not a real issue or working as designed |
| `duplicate` | Subsumed by another ticket |
| `abandoned` | Work started but stopped |

Default: none. Setting a resolution does not by itself move a ticket that is already operationally **resolved**. Persists across `jekt/status` transitions unless explicitly changed.

### `jekt/resolvedAt`

Scalar (instant). The instant `jekt/resolution` was most recently set, independent of the ticket's `jekt/status`.

### `jekt/priority`

| Value | Meaning |
|---|---|
| `blocker` | Must be resolved before release can proceed |
| `urgent` | Requires immediate attention |
| `high` | Must be resolved in the current cycle |
| `medium` | Should be resolved (default) |
| `low` | Resolve when convenient |
| `trivial` | No urgency; address opportunistically |

Default: `medium`.

### User Identifiers

A **user identifier** is a **username** (`jdoe`) or an **email address** (`jdoe@example.com`), told apart by whether `@` appears. A username is the value of an email address's local part: letters, digits, `.`, and the other local-part characters, plus `:` where an identifier is adopted from another system; never `@`, `[`, `]`, `` ` ``, `*`, `"`, `\`, whitespace, or a control character. Case-insensitive, canonical lowercase — normalize in `description.yaml` and frontmatter, keep the author's case in prose. Stored bare (`creator: jdoe`); the `@` belongs to a person reference (§4), not to the value.

### `creator` and `author`

User identifiers (see *User Identifiers* above). `creator` records who created the artifact (audit); `author` attributes its content to whoever composed or stated it (attribution). Optional; an artifact may carry both where they differ. A comment carries `author` and no `creator`.

### Foreign-Identity and Import Properties

Scalar. Intrinsic. Recorded on import from an external system; absent for a natively-created ticket.

| Property | Type | Notes |
|---|---|---|
| `source` | string (URI) | Stable identifier for the ticket's counterpart in the external system. A ticket has at most one source at a time — never overwrite a conflicting existing value. |
| `createdAt` | instant | The ticket's original creation time. Overrides the Git-derived creation date when they would differ, as for imported content. |
| `importedAt` | instant | The time the ticket was most recently populated from the external system. |

### `jekt/assignee`

User identifier (see *User Identifiers* above). Scalar (one assignee per ticket). Default: none.

### Intrinsic and namespaced list properties (no enumerated values)

| Property | Storage | Notes |
|---|---|---|
| `tags` | `tags.lst` at the ticket root | Intrinsic. Categorical classifiers attached to a ticket (e.g., `security`, `breaking-change`). Open vocabulary. |
| `jekt/releases` | `jekt/releases.lst` at the ticket root | Namespaced. Release identifiers a ticket is assigned to. |

## 6. Metadata Storage

- **Scalar properties** live in `description.yaml`; a comment's own live in its frontmatter (§7).
- **List properties** are stored as `.lst` files when their values are commonly searched, edited, or merged independently, as `tags` and `jekt/releases` are; otherwise they may be YAML lists in `description.yaml`. A `.lst` file's path in the ticket directory mirrors its property name (`tags.lst`, `jekt/releases.lst`).
- **Namespacing:** `prefix/local` with slash; local part in **camelCase**. The slash is the only namespace separator.
- **Intrinsic vs. namespaced.** Bare names for properties that would exist independent of any tracker (`tags`, `creator`); `jekt/...` for system-defined; `{org}/...` for extension.
- **Cardinality is fixed per property.** Scalar properties never appear as lists; list properties never appear as scalars. Polymorphic scalar-or-list properties are forbidden.
- **Open-world semantics.** Absence = absence of assertion. No `null`, no `~`, no empty strings or other sentinels. An empty `description.yaml` (a zero-byte file, or one containing only `{}` or comments) is the open-world degree-zero state: the entity exists, no properties yet asserted. Do not delete it on the grounds that it carries no values.
- **Defaults are applied at consumption, never written** into files.
- **No inverse relations stored.** Tickets refer outward only (no `jekt/blocks` mirror of `jekt/dependsOn`).
- The **label** and the **heading title** (§7) are filesystem and document affordances, not stored properties. They do not appear in `description.yaml`.

### Preservation vs. Tolerance

- **Preservation** (on round-trip edit): do not destroy unknown YAML properties (including extension prefixes such as `acme/...`), ad-hoc files at the ticket root, or existing reference link definitions.
- **Tolerance** (on read): do not reject conforming variations — flow vs. block YAML on input, date-only vs. full-timestamp comment filenames, kebab-case vs. Title-Case ad-hoc filenames.

## 7. Storage Formats

Line endings in Jekt files follow the repository's Git configuration.

| Artifact | Format | Rules |
|---|---|---|
| Prose (description, summary, minutes, plans, designs, comments, TODOs) | Markdown | CommonMark |
| `description.yaml` | YAML | Bare unquoted keys (slash does not require quoting); block-form sequences on output; no nulls/sentinels |
| `*.lst` | Plain text | One value per line, no header, terminated by a final newline; UTF-8 without a byte order mark |

Comment files MAY carry a `---`-delimited YAML frontmatter block for `author`, `createdAt`, and `inReplyTo`; no other prose artifact uses frontmatter. `inReplyTo` is a relative reference to the answered comment's file — a sibling in the same ticket by filename alone — and is omitted where the comment answers none.

### `description.yaml` example

```yaml
jekt/kind: feature
jekt/status: closed
jekt/resolution: completed
jekt/priority: high
jekt/assignee: jdoe@example.com
creator: asmith
```

### `.lst` example

```
security
backend
performance
```

### Comment frontmatter example

```yaml
author: jdoe
createdAt: 2026-02-03T21:45:00.353Z
inReplyTo: 2026-02-03T20-10-00Z-54320.md
```

### Derived properties

| Property | Source |
|---|---|
| Label | Description filename, the portion after the `description - ` prefix and before `.md` |
| Heading title | Level-one heading text of the description prose file, following the bracketed ID |
| Created date | Git history of the ticket directory |
| Minutes-entry author | Git commit author of the containing commit |

`author` is never derived. Write it explicitly or leave it unstated.

## 8. Jekt Refs

A **claim ref**, `refs/jekt/ticket-claims/{ticket-id}`, reserves a ticket number on the remote. Refs under `refs/jekt/` are not branches: they do not appear in `git branch` or forge branch UIs, and `git push --all` does not push them, so explicit refspecs are required.

## 9. Minutes

A ticket's `minutes.md` has this form; the Update Minutes procedure writes it.

```markdown
# [{ticket-id}] Minutes

<!-- Categories: Pivot | Insight | Decision | Finding | Lesson | Milestone | Open | Resolved -->

- YYYY-MM-DD **Category**: …
```

The comment lists the recommended categories; the vocabulary is open, and a team may add its own.
