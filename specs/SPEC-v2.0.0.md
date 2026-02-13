# Specification

**Version:** 2.0.0

This document is the single source of truth for the `.backlogmd/` system — a markdown-based agile/kanban workflow for agentic development. Agents must read this file before interacting with the backlog.

## Directory Structure

```
.backlogmd/
├── backlog.md               # Active items, ordered by priority
├── work/                    # Max 50 open items
│   ├── <item-slug>/
│   │   ├── index.md         # Item task list
│   │   ├── 001-task-slug.md
│   │   ├── 002-task-slug.md
│   │   └── ...
│   └── ...
└── .archive/                # Cold storage — read-only, skipped by parsers
    └── <YYYY>/
        └── <MM>/
            └── <item-slug>/ # Archived item folders (moved intact)
```

All paths in this document and throughout the system are relative within `.backlogmd/`.

## Backlog Format

File: `backlog.md`

```md
- [001-chore-project-foundation](work/001-chore-project-foundation/index.md)
- [002-ci-initialize-github-actions](work/002-ci-initialize-github-actions/index.md)
- [003-chore-implement-design](work/003-chore-implement-design/index.md)
- [004-feat-user-admin](work/004-feat-user-admin/index.md)
```

- `backlog.md` is a bullet list of markdown links to each item's `index.md`.
- Each entry uses the format `- [<item-slug>](work/<item-slug>/index.md)`.
- Items are ordered by priority — first item is highest priority.
- The slug format is `<NNN>-<name>`, e.g. `001-project-foundation`.
- An optional type segment following [Conventional Commits](https://www.conventionalcommits.org/) can be included after the number: `<NNN>-<type>-<name>`, e.g. `001-chore-project-foundation`. Common types: `feat`, `fix`, `refactor`, `chore`. The type is not required.
- Completed items are removed from `backlog.md` and their folder is moved to `.archive/`.

## Item Format

File: `work/<item-slug>/index.md`

```md
- [001-task-slug](001-task-slug.md)
- [002-task-slug](002-task-slug.md)
```

- The item `index.md` is a bullet list of relative links to task files.
- Item slug is lowercase kebab-case with priority number prefix (e.g. `001-project-foundation`). An optional [Conventional Commits](https://www.conventionalcommits.org/) type can follow the number (e.g. `001-chore-project-foundation`).
- Each line links to a task file within the same directory.

## Task Format

File: `work/<item-slug>/<NNN>-<task-slug>.md`

````md
<!-- METADATA -->

```
Task: <Task Name>
Status: <status>
Priority: <NNN>
DependsOn: [<task-slug>](relative-path-to-task.md)
```

<!-- /METADATA -->
<!-- DESCRIPTION -->

## Description

<detailed description>

<!-- /DESCRIPTION -->

<!-- ACCEPTANCE CRITERIA -->

## Acceptance criteria

- [ ] <criterion>

<!-- /ACCEPTANCE CRITERIA -->
````

- Task filenames use the pattern `<NNN>-<task-slug>.md` where `NNN` is a zero-padded priority number.
- Priority numbers are unique within an item.
- Valid task statuses: `open`, `block`, `in-progress`, `done`.
- Metadata is wrapped in HTML comment markers (`<!-- METADATA -->` / `<!-- /METADATA -->`) and uses a fenced code block for key-value pairs.
- `DependsOn` is optional. When present, it is a markdown link to the dependency task file (relative path). Use `—` or omit the field when there are no dependencies.
- Description is wrapped in `<!-- DESCRIPTION -->` / `<!-- /DESCRIPTION -->` comment markers.
- Acceptance criteria are wrapped in `<!-- ACCEPTANCE CRITERIA -->` / `<!-- /ACCEPTANCE CRITERIA -->` comment markers and use markdown checkboxes.

## Archive

Directory: `.archive/`

- The archive is **cold storage** — agents and parsers must skip `.archive/` during normal operations.
- When an item is completed, its entire folder is moved from `work/<slug>/` to `.archive/<YYYY>/<MM>/<slug>/` (using the current year and month) and the slug is removed from `backlog.md`.
- Archive contents are **read-only**. Agents never modify archived items, only move things into the archive.

## Limits

- **Max 50 open items** in `work/` at any time. A new item folder cannot be created if 50 open items already exist.
- Both archival actions (item folder move to `.archive/<YYYY>/<MM>/` + slug removal from `backlog.md`) happen together.

## Versioning

The spec follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`):

- **MAJOR** — breaking changes (directory renames, format changes, removed fields).
- **MINOR** — backward-compatible additions (new optional fields, new item types, new sections).
- **PATCH** — clarifications, typo fixes, and examples that don't change behavior.

Rules:

- The spec version is stated at the top of this file (**Version:** 2.0.0).
- For change history and migration guides, see `SPEC-CHANGELOG.md`.
- When a new major version is released, the previous spec is preserved in `specs/` (e.g. `specs/v1.md`). `SPEC.md` always describes the current version.
- Agents and parsers may check the version and reject or adapt when they do not support it.

## Conventions

- All paths are relative within `.backlogmd/`.
- Priority numbers are unique per scope (items in backlog, tasks within an item folder).
- Item `index.md` task list must stay in sync with individual task files.
- Agents must read `SPEC.md` before interacting with the backlog.
- No YAML frontmatter — the format uses HTML comment sections and fenced code blocks for metadata.

## Workflow Rules

1. Tasks move forward through statuses: `open` → `in-progress` → `done`. Use `block` when a task is blocked.
2. A task cannot move to `in-progress` if it has a `DependsOn` task that is not `done`.
3. Circular dependencies are not allowed.
4. When an agent changes a task's status, it must update the task file.
5. When all tasks in an item are `done`, the item may be archived (folder moved to `.archive/<YYYY>/<MM>/`, slug removed from `backlog.md`).
