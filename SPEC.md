# Specification

**Version:** 4.0.4

Single source of truth for the `.backlogmd/` system — markdown-based agile/kanban for agentic development. Agents must read this file before interacting with the backlog.

## Directory Structure

```
.backlogmd/
├── work/
│   ├── <item-id>-<slug>/
│   │   ├── index.md
│   │   ├── 001-task-slug.md
│   │   ├── 001-task-slug-feedback.md   # optional, agent feedback when stuck
│   │   ├── 002-task-slug.md
│   │   └── ...
│   └── ...
└── z-archive/
    └── <YYYY>/<MM>/<item-id>-<slug>/
```

All paths in this document and throughout the system are relative within `.backlogmd/`.

## Open items

- **Open items** are the directories under `work/`. Agents discover work by listing `work/`, then for each item directory reading `index.md` (metadata, including item `status`, and `<!-- CONTEXT -->`) and listing task files (filenames matching `<tid>-<task-slug>.md`). Items with `status: plan` are not ready for agents; items with `status: open` may have tasks ready to start.
- **Archived items** are under `z-archive/`; agents skip them for active work.
- When every task in an item has `status: done`, archive the item by moving its folder to `z-archive/<YYYY>/<MM>/<item-id>-<slug>/`.

## IDs and Naming

- **Item IDs** (`item-id`): Zero-padded integers, minimum 3 digits (e.g., `001`, `012`, `999`, `1000`). Unique across the backlog.
- **Task IDs** (`tid`): Zero-padded integers, minimum 3 digits. Unique per item.
- **Slugs**: Lowercase kebab-case. An optional Conventional Commit type may follow the ID as the first slug segment (e.g., `001-chore-project-foundation`).
- **Priority** (`priority`): Integer, unique per item. **Lower number = higher priority.**

## Item Format (`work/<item-id>-<slug>/index.md`)

- Item-level metadata, description, and a **CONTEXT** section for agents. It does **not** list tasks. Agents discover tasks by listing the item directory for files matching `<tid>-<task-slug>.md` (excluding `index.md`).
- Three HTML comment markers: `<!-- METADATA -->`, `<!-- DESCRIPTION -->`, `<!-- CONTEXT -->`. The **CONTEXT** block is read and used by agents when working on any task in this item (e.g. repo conventions, pointers to docs, environment notes).

### Structure

````md
<!-- METADATA -->

```yaml
work: Add login flow # work item title
status: open # plan | open | in-progress | done
```

<!-- DESCRIPTION -->

<optional item description>

<!-- CONTEXT -->

<context for agents: conventions, links, env notes, etc.>
````

### Item status

- **`status`**: Item-level lifecycle, distinct from task `status`. Values:
  - **`plan`** — Grooming/draft; not ready for agents to work on.
  - **`open`** — Ready for agents to start tasks in this item.
  - **`in-progress`** — At least one task in the item is being worked on.
  - **`done`** — Every task in the item has `status: done`; item is ready to archive (or already considered complete).

### Rules

- Metadata is YAML in the fenced block; keep metadata lines ≤ 120 chars. No YAML frontmatter outside it.
- **CONTEXT** may be empty; agents should still read `index.md` when picking up a task in the item so they see any context that is present.

## Task Format (`work/<item-id>-<slug>/<tid>-<task-slug>.md`)

````md
<!-- METADATA -->

```yaml
task: Add login form
status: plan # plan | open | in-progress | review | block | done
priority: 10 # priority within item (lower = higher priority)
dep: ["work/002-ci-initialize-github-actions/001-ci-cd-setup.md"] # optional: paths (relative to .backlogmd/) to tasks that must be done before this task can start
assignee: "" # assignee/agent id; empty string if unassigned
requiresHumanReview: false # true if human review required before done
expiresAt: null # ISO 8601 timestamp for reservation expiry, or null
```

<!-- DESCRIPTION -->

## Description

<detailed description>

<!-- ACCEPTANCE -->

## Acceptance criteria

- [ ] <criterion>
````

### Dependencies

- **`dep`** lists paths to **other tasks** that must be **done** before this task can move to `in-progress`. Each entry is a path relative to `.backlogmd/`: `work/<item-id>-<slug>/<tid>-<task-slug>.md` (e.g. `work/002-ci-initialize-github-actions/001-ci-cd-setup.md`). Dependencies may be in the same item or in another item (cross-item). Agents must wait for each referenced work/task to be complete (`status: done`) before starting this task.

### Rules

- Filenames: `<tid>-<task-slug>.md`; `tid` zero-padded, unique per item. The only required file for a task is the task file itself. A task may have an optional sibling **feedback file** (see below).
- Status codes:
  - `plan` — groomed/draft, not ready for agents to pick up.
  - `open` — ready for an agent to start work.
  - `in-progress` — being worked on. Requires `assignee` (non-empty).
  - `review` — awaiting human approval. Set when `requiresHumanReview: true` and work is complete. Requires `assignee`.
  - `block` — blocked by an external dependency. `assignee` is preserved.
  - `done` — complete. `assignee` is cleared.
- **`dep`**: See **Dependencies** above. No self-reference; no duplicates; no cycles (backlog must remain a DAG).
- Three HTML comment markers only: `<!-- METADATA -->`, `<!-- DESCRIPTION -->`, `<!-- ACCEPTANCE -->`.
- Acceptance criteria use markdown checkboxes.
- No YAML frontmatter outside the fenced code block; keep metadata lines ≤ 120 chars.

### Feedback file (`<tid>-<task-slug>-feedback.md`)

- **Purpose:** Optional file next to a task file. Agents use it to record why work stopped (e.g. stuck or blocked) so the next agent or a human has context. It is not used for task discovery; task files are discovered by listing `<tid>-<task-slug>.md` only (feedback files are excluded).
- **Path:** `work/<item-id>-<slug>/<tid>-<task-slug>-feedback.md` (same directory as the task file, same `tid` and slug with `-feedback` before `.md`).
- **When to write:**
  - **Blocking:** When setting task `status: block`, the agent MUST create or append to this file with: what was tried, why progress is blocked, and optionally what would unblock (e.g. "wait for PR merge", "need API key").
  - **Releasing (stuck):** When stopping work without completing (releasing the task back to `open`), if the agent is stuck the agent SHOULD append a short note: what was tried, why stuck.
- **Format:** Freeform text or markdown. **Separator:** each new entry MUST start with a level-2 heading with an ISO 8601 date so entries are ordered and easy to scan. Use `## YYYY-MM-DD` or `## YYYY-MM-DDTHH:mm:ssZ` (UTC). Example:

  ```md
  ## 2026-02-16T14:30:00Z
  Tried X and Y. Blocked: API returns 403. Would unblock: valid API key.

  ## 2026-02-17T09:00:00Z
  Retried after key rotation. Still stuck on rate limit.
  ```

  Append each entry (newest at bottom); do not overwrite. The next reader sees prior attempts and blockers in order.
- **Who reads it:** When starting work on a task, the agent SHOULD read the feedback file if present so previous attempts or blockers are visible. Humans may read it when triaging or unblocking.

## Human-in-the-Loop Protocol

### Write ordering

File-based systems cannot provide true atomicity. To minimize inconsistency:

- Task files are the authoritative source for task status, assignment, and content. The item's `index.md` is the source for item metadata, description, and CONTEXT.

1. **Task edits:** write only the task file.
2. **Item-level edits** (item metadata, description, or CONTEXT): write only `index.md`.
3. **Task feedback:** when recording feedback (e.g. when stuck or blocking), create or append to the task's `-feedback.md` file only; path is `work/<item-id>-<slug>/<tid>-<task-slug>-feedback.md`.
4. There is no task list in `index.md`; nothing to regenerate from task files.

### Starting work

1. Agent lists directories under `work/`, then per item lists task files (`<tid>-<task-slug>.md`) and reads their metadata to find tasks with `status: open`. When working on a task, the agent SHOULD read that item's `index.md` (especially `<!-- CONTEXT -->`) and, if present, the task's feedback file `<tid>-<task-slug>-feedback.md` (so previous attempts or blockers are visible).
2. To start work on an `open` task: set `status: in-progress`, `assignee: <agent-id>`, and optionally `expiresAt` (ISO 8601, short horizon — e.g., 30 min). A task cannot move to `in-progress` until **every** dependency in its `dep` list is done: each `dep` entry is a path `work/<item-id>-<slug>/<tid>-<task-slug>.md`; read that task file and require `status === "done"` for each. Update only the task file.
3. Only `open` tasks may be started. `plan` tasks must first be promoted to `open` by a human or authorized agent.

### Completing work

4. If `requiresHumanReview: false`: set `status: done` and clear `assignee`.
5. If `requiresHumanReview: true`: agent MUST set `status: review` and **stop**. Direct `in-progress → done` is **invalid** when `requiresHumanReview: true`. Only a human (or authorized role) may move `review → done`.

### Releasing

6. To stop working on a task without completing it, set `status: open` and clear `assignee` and `expiresAt`. If the agent is stuck (e.g. giving up without completing), the agent SHOULD append to the task's `-feedback.md` file first with a short note (what was tried, why stuck) so the next agent or human has context.

### Expiry

7. If `expiresAt` is non-null and in the past, another agent may take over by setting `status: in-progress`, overwriting `assignee` with its own ID, and setting a fresh `expiresAt`.
8. `expiresAt` applies only in `in-progress`; ignore it while a task is in `review`.

### Blocking

9. Set `status: block` when an external dependency prevents progress. `assignee` remains set. When setting `status: block`, the agent MUST create or append to the task's `-feedback.md` file (path: `work/<item-id>-<slug>/<tid>-<task-slug>-feedback.md`) with: what was tried, why progress is blocked, and optionally what would unblock (e.g. "wait for PR merge", "need API key"). Valid transitions out of `block`: back to `in-progress` (when unblocked) or to `open` (if releasing).

## Archive

- Cold storage; agents skip `z-archive/`. The name sorts after `work/` in listings so active work appears first.
- An item is archived **only when every task in the item has `status: done`**.
- Archive procedure: move the item folder to `z-archive/<YYYY>/<MM>/<item-id>-<slug>/` (including any `-feedback.md` files). There is no shared file to update; the directory structure is the source of truth.

## Limits

- Max 20 open items (max 20 directories in `work/`).
- Recommended max 20 tasks per item. Items with more should be split into a new item.

## Workflow Rules

### Status flow

```
plan ──→ open ──→ in-progress ──→ done           (requiresHumanReview: false)
                            ──→ review ──→ done (requiresHumanReview: true)

Any active state ──→ block ──→ in-progress or open
```

- `plan → open`: promotion (human or authorized agent only).
- `open → in-progress`: start work (any agent); set `assignee` and optionally `expiresAt`.
- `in-progress → done`: completion (only valid when `requiresHumanReview: false`).
- `in-progress → review → done`: completion with human gate (required when `requiresHumanReview: true`).
- `block`: reachable from `in-progress` or `review`. Returns to `in-progress` (unblocked) or `open` (released).
- A task cannot move to `in-progress` until every referenced task in `dep` (by path `work/.../<tid>-<task-slug>.md`) has `status: done`. Agents wait for each dependency (that work/task) to be fully done before starting.
- No circular dependencies. The set of all tasks and their `dep` paths must form a DAG. Agents adding `dep` entries must verify no cycle is introduced across the backlog.
- Task edits update only the task file. Task feedback updates only the task's `-feedback.md` file when present. Item-level edits (metadata, description, CONTEXT) update the item's `index.md`.
- When all tasks in an item are `done`, archive the item.

## Reconciliation

Task files and directory structure are the source of truth. There is no shared backlog file. `index.md` does not contain a task list; tasks are discovered by listing the item directory for `<tid>-<task-slug>.md` files.

- If a task is `done`, `assignee` MUST be empty in the task file; clear it during any repair.
- No link list to regenerate in `index.md`.

## Conventions

- All paths are relative within `.backlogmd/`.
- Priority numbers are unique per item (lower = higher priority).
- Task files are discovered by listing the item directory for files named `<tid>-<task-slug>.md` (excluding `index.md` and any `-feedback.md` files).
- No YAML frontmatter outside the fenced code block.
- Keep metadata lines ≤ 120 chars.
- Avoid inline HTML outside the three comment markers.

## Versioning

- Semantic Versioning (`MAJOR.MINOR.PATCH`).
- This file is 4.0.4. Prior specs live in `specs/`.
- See `SPEC-CHANGELOG.md` for history and migrations.
- Agents may reject if the spec version in this file is unsupported.
