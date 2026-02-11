# Protocol

**Version:** 1.0.0

This document is the single source of truth for the `.backlogmd/` system — a markdown-based agile/kanban workflow for agentic development. Agents must read this file before interacting with the backlog.

## Directory Structure

```
.backlogmd/
├── PROTOCOL.md              # This file — rules, formats, conventions
├── backlog.md               # Active items (todo + in-progress), ordered by priority
├── items/                   # Max 10 open items
│   ├── <item-slug>/
│   │   ├── index.md         # Item metadata + task summary table
│   │   ├── 001-task-slug.md
│   │   ├── 002-task-slug.md
│   │   └── ...
│   └── ...
└── .archive/                # Cold storage — read-only, skipped by parsers
    ├── backlog.md            # Completed items
    └── items/                # Archived item folders (moved intact)
        └── <item-slug>/
```

All paths in this document and throughout the system are relative within `.backlogmd/`.

## Roadmap Format

File: `backlog.md`

```md
# Roadmap

## Items

### <NNN> - <Item Name>
- **Type:** <type>
- **Status:** <status>
- **Item:** [<item name>](items/<item-slug>/index.md)
- **Description:** <one-line summary>
```

- `backlog.md` only contains active items (`todo` and `in-progress`). Completed items are moved to `.archive/backlog.md`.
- Items are ordered by priority number (`001` = highest).
- Priority numbers are zero-padded to three digits and unique within the roadmap.
- Each roadmap entry links to its item folder. Items without a folder use `—` as the value.
- Valid types: `feature`, `bugfix`, `refactor`, `chore`. Projects may extend this list as needed.
- Valid item statuses: `todo`, `in-progress`, `done`.
- An item's status is derived from its tasks:
  - All tasks `done` → item is `done`
  - Any task `in-progress`, `ready-to-review`, or `ready-to-test` → item is `in-progress`
  - Mix of `done` and `todo` tasks → item is `in-progress`
  - All tasks `todo` → item is `todo`

## Item Format

File: `items/<item-slug>/index.md`

```md
# <Item Name>

- **Type:** <type>
- **Status:** <status>
- **Goal:** <one-line goal>

## Tasks

| # | Task | Status | Owner | Depends on |
|---|------|--------|-------|------------|
| 001 | [Task name](001-task-slug.md) | <status> | <owner> | — |
```

- Item slug is lowercase kebab-case.
- Valid types: `feature`, `bugfix`, `refactor`, `chore`. Projects may extend this list as needed.
- Valid item statuses: `open`, `archived`.
- Only `open` items accept new tasks.
- The task table must stay in sync with the individual task files.

## Task Format

File: `items/<item-slug>/<NNN>-<task-slug>.md`

```md
# <Task Name>

- **Status:** <status>
- **Priority:** <NNN>
- **Owner:** <owner>
- **Item:** [<Item Name>](../../backlog.md#NNN---item-name-slug)
- **Depends on:** <linked task list or `—`>
- **Blocks:** <linked task list or `—`>

## Description

<detailed description>

## Acceptance Criteria

- [ ] <criterion>
```

- Task filenames use the pattern `<NNN>-<slug>.md` where `NNN` is a zero-padded priority number.
- Priority numbers are unique within an item.
- Valid task statuses, in order: `todo` → `in-progress` → `ready-to-review` → `ready-to-test` → `done`.
- Owner is a handle (e.g. `@claude`) or `—` when unassigned.
- The Item field links back to the parent item in `backlog.md`.
- Acceptance criteria use markdown checkboxes.
- `Depends on` and `Blocks` are optional. Omit them from task files and the task table column when the project doesn't need dependency tracking. When used, `Depends on` lists tasks that must be `done` before this task can start (markdown links: `[001 - Task name](001-task-slug.md)`). `Blocks` is the inverse. Use `—` for tasks with no dependencies when the column/field is present.

## Archive

Directory: `.archive/`

- The archive is **cold storage** — agents and parsers must skip `.archive/` during normal operations.
- `.archive/backlog.md` uses the same format as `backlog.md`. Completed items are appended to it preserving their original priority number.
- `.archive/items/` contains archived item folders, moved intact from `items/`.
- Archive contents are **read-only**. Agents never modify archived items, only move things into the archive.

## Limits

- **Max 10 open items** in `items/` at any time. A new item folder cannot be created if 10 open items already exist.
- When an item is archived, its entire folder is moved from `items/<slug>/` to `.archive/items/<slug>/`.
- When a roadmap item reaches `done` status, it is removed from `backlog.md` and appended to `.archive/backlog.md`.
- Both archival actions (item folder move + roadmap cleanup) happen together when an item is archived.

## Versioning

The protocol follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`):

- **MAJOR** — breaking changes (directory renames, format changes, removed fields).
- **MINOR** — backward-compatible additions (new optional fields, new item types, new sections).
- **PATCH** — clarifications, typo fixes, and examples that don't change behavior.

Rules:

- The protocol version is stated at the top of this file (**Version:** 1.0.0).
- For change history and migration guides, see `PROTOCOL-CHANGELOG.md`.
- When a new major version is released, the previous protocol is preserved in `protocols/` (e.g. `protocols/v1.md`). `PROTOCOL.md` always describes the current version.
- Agents and parsers may check the version and reject or adapt when they do not support it.

## Conventions

- All paths are relative within `.backlogmd/`.
- Priority numbers are unique per scope (items in roadmap, tasks within an item folder).
- Owner is optional; use `—` when unassigned.
- Item `index.md` task table must stay in sync with individual task files.
- Agents must read `PROTOCOL.md` before interacting with the backlog.
- No YAML frontmatter — the format is pure markdown throughout.

## Workflow Rules

1. Tasks move forward through statuses, never backward.
2. Only `open` items accept new tasks.
3. A roadmap item's status is derived from its tasks (see Roadmap Format above).
4. When all tasks in an item are `done`, the item may be archived. Archiving moves the item folder to `.archive/items/` and moves any newly `done` roadmap entries to `.archive/backlog.md`.
5. A task cannot move to `in-progress` unless all of its dependencies are `done`.
6. Circular dependencies are not allowed.
7. When an agent begins work on a task, it must update the task status to `in-progress` in both the task file and the item's task table.
8. When an agent completes a task, it must update the task status to `done` in both the task file and the item's task table.
9. When all tasks in an item reach `done`, the item's roadmap status in `backlog.md` must be updated to reflect the derived status.
