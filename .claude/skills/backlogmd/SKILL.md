---
name: backlogmd
description: Use when planning work (to create items and tasks), when starting implementation (to start and work tasks), when completing work (to mark tasks done), or to check backlog status. Manages .backlogmd/ (work/ and z-archive/); no shared manifest or backlog file. Item index.md has metadata, description, and CONTEXT for agents.
argument-hint: init/create/start/done/edit/show/archive/check <item or task>
allowed-tools: Read, Write, Edit, Glob, Bash(mkdir *), Bash(mv *)
---

# Backlog Manager

You are an agent that manages the `.backlogmd/` backlog system. You can create items (features, bugfixes, refactors, chores) and tasks, start and release tasks, update statuses, edit content, and archive completed work. There is no shared manifest or backlog file: open items are directories under `work/`; tasks are discovered by listing each item directory for `<tid>-<task-slug>.md` files; each item's `index.md` holds metadata, description, and `<!-- CONTEXT -->` for agents. Write only the task file (or only `index.md` for item-level edits); never regenerate a shared index.

## Workflow (MANDATORY)

> **RULE**: For new features, bugfixes, refactors, or chores — create or update backlog items BEFORE writing code. The backlog is the source of truth for planned work. For small iterations on an existing task (tweaks, adjustments, follow-ups), you may skip backlog updates and just work.

1. **Before planning**: List `.backlogmd/work/`, read each item's `index.md` (metadata, CONTEXT) and list task files to see existing items and tasks.
2. **When planning**: Create items and tasks in the backlog FIRST, before any implementation. Don't just describe plans in conversation — record them. New tasks start as `open` (ready for agents) or `plan` (draft, needs human promotion). Item `status` in `index.md`: `plan` | `open` | `claimed` | `in-progress` | `done`; optional `assignee` at work level.
3. **Wait for approval**: After planning, present the plan to the user and **STOP**. Do NOT start implementing until the user explicitly approves.
4. **When implementing**: Follow this loop for EACH task, one at a time:
   - **Start** the task: set `status: in-progress`, `assignee: <agent-id>` (and optionally `expiresAt`) in the task file; set the item's `index.md` to `status: claimed` or `in-progress` and `assignee: <agent-id>`. First verify every `dep` path resolves to a task file with `status: done`. Read the item's `index.md` (especially `<!-- CONTEXT -->`) and any `<tid>-<task-slug>-feedback.md` if present.
   - **Implement** the task.
   - **Complete** the task: if `requiresHumanReview: false`, set `status: done` and clear `assignee` in the task file; if all tasks in the item are done, set item `status: done` and clear item `assignee`. If `requiresHumanReview: true`, set `status: review` and **stop** — only a human may move `review → done`.
   - **Only then** move to the next task.
   - **Writes**: Task edits → task file only. Item-level edits → `index.md` only. When blocking or releasing (stopping without completing), create/append to the task's `-feedback.md` file.
5. **When all tasks are done**: Inform the user and ask if they want to archive the item.

---

## Spec v4.0.4 (embedded)

Single source of truth for `.backlogmd/` is `SPEC.md` in the repo; this section embeds the key rules so the skill is self-contained. When in doubt, prefer `SPEC.md`.

### Directory Structure

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

All paths are relative within `.backlogmd/`.

### Open items

- **Open items** are the directories under `work/`. Agents discover work by listing `work/`, then for each item directory reading `index.md` (metadata, including item `status`, and `<!-- CONTEXT -->`) and listing task files (filenames matching `<tid>-<task-slug>.md`). Items with `status: plan` are not ready for agents; items with `status: open` or `status: claimed` may have tasks ready to start (claimed = an agent has taken the item).
- **Archived items** are under `z-archive/`; agents skip them for active work.
- When every task in an item has `status: done`, archive the item by moving its folder to `z-archive/<YYYY>/<MM>/<item-id>-<slug>/`.

### IDs and Naming

- **Item IDs** (`item-id`): Zero-padded integers, minimum 3 digits (e.g., `001`, `012`, `999`, `1000`). Unique across the backlog.
- **Task IDs** (`tid`): Zero-padded integers, minimum 3 digits. Unique per item.
- **Slugs**: Lowercase kebab-case. An optional Conventional Commit type may follow the ID as the first slug segment (e.g., `001-chore-project-foundation`).
- **Priority** (`priority`): Integer, unique per item. **Lower number = higher priority.**

### Item Format (`work/<item-id>-<slug>/index.md`)

- Item-level metadata, description, and a **CONTEXT** section for agents. It does **not** list tasks. Agents discover tasks by listing the item directory for files matching `<tid>-<task-slug>.md` (excluding `index.md`).
- Three HTML comment markers: `<!-- METADATA -->`, `<!-- DESCRIPTION -->`, `<!-- CONTEXT -->`. The **CONTEXT** block is read and used by agents when working on any task in this item.

**Structure:**

````md
<!-- METADATA -->

```yaml
work: Add login flow # work item title
status: open # plan | open | claimed | in-progress | done
assignee: "" # agent id when work item is claimed (required when status: claimed); empty when open or done
```

<!-- DESCRIPTION -->

<optional item description>

<!-- CONTEXT -->

<context for agents: conventions, links, env notes, etc.>
````

**Item status:** `plan` (grooming, not ready) | `open` (ready for agents; assignee empty) | `claimed` (agent has claimed this work item; **assignee required**, non-empty) | `in-progress` (at least one task in progress; assignee may be set) | `done` (all tasks done, ready to archive; assignee empty). **Work-level assignee:** required when status is `claimed`; set when claiming; clear when releasing or when item is done.

### Task Format (`work/<item-id>-<slug>/<tid>-<task-slug>.md`)

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

- Filenames: `<tid>-<task-slug>.md`; `tid` zero-padded, unique per item. Optional sibling: `<tid>-<task-slug>-feedback.md` (feedback file). **Feedback file:** path `work/<item-id>-<slug>/<tid>-<task-slug>-feedback.md`; each entry starts with `## YYYY-MM-DD` or `## YYYY-MM-DDTHH:mm:ssZ` (UTC), then freeform text; append (don’t overwrite) so history is preserved. Write when blocking (MUST: what was tried, why blocked, what would unblock) or when releasing while stuck (SHOULD: what was tried, why stuck). Not used for task discovery. Agents SHOULD read it when starting a task if present.
- Status codes: `plan` | `open` | `in-progress` | `review` | `block` | `done`. `in-progress` and `review` require non-empty `assignee`; `done` clears `assignee`.
- **dep**: Paths relative to `.backlogmd/`: `work/<item-id>-<slug>/<tid>-<task-slug>.md`. Cross-item allowed. No self-reference, no duplicates, no cycles (DAG). Task cannot move to `in-progress` until every `dep` task has `status: done`.

### Human-in-the-Loop Protocol

- **Write ordering:** Task edits → task file only. Item edits → `index.md` only. Feedback → `-feedback.md` only. No shared file to update.
- **Starting work:** List `work/`, list task files per item, find tasks with `status: open` (items with `status: open` or `claimed`). Read item `index.md` (CONTEXT) and task `-feedback.md` if present. In the **task file**: set `status: in-progress`, `assignee: <agent-id>`, optionally `expiresAt`. In the **item** `index.md`: set item `status: claimed` (or `in-progress` once any task is in progress) and `assignee: <agent-id>`. Require every `dep` task file to have `status: done` before starting.
- **Completing:** If `requiresHumanReview: false` → `status: done`, clear `assignee` in task file. If all tasks in the item are done, set item `status: done` and clear item `assignee`. If `requiresHumanReview: true` → set `status: review` and **stop**.
- **Releasing:** In task file: set `status: open`, clear `assignee` and `expiresAt`. If no other task is in progress on that item, set item `status: open` and clear item `assignee`. If releasing because stuck, append to task's `-feedback.md` first.
- **Blocking:** Set `status: block`; MUST create/append to task's `-feedback.md` (what was tried, why blocked, what would unblock).
- **Expiry:** If `expiresAt` in the past, another agent may take over (set `status: in-progress`, new `assignee`, fresh `expiresAt`).

### Status flow

```
plan ──→ open ──→ claimed ──→ in-progress ──→ done   (requiresHumanReview: false)
                            ──→ review ──→ done (requiresHumanReview: true)

Any active state ──→ block ──→ in-progress or open
```

### Archive

- Archive **only when every task in the item has `status: done`**. Move the item folder to `z-archive/<YYYY>/<MM>/<item-id>-<slug>/` (including any `-feedback.md` files). No shared file to update.

### Limits

- Max 20 open items (max 20 directories in `work/`). Recommended max 20 tasks per item.

### Reconciliation

- Task files and directory structure are the source of truth. No task list in `index.md`. If a task is `done`, `assignee` MUST be empty in the task file.

---

## Checking the current spec

Before acting on the backlog, ensure your behavior matches the **current spec**. The single source of truth is `SPEC.md` in the repo (see **Version** in that file). This skill embeds a summary (Spec v4.0.4 above); when in doubt, prefer `SPEC.md`. In particular:

- **Work-level metadata:** Item `index.md` has `work`, `status` (plan | open | claimed | in-progress | done), and `assignee`. When status is `claimed`, assignee is required (non-empty). When an agent claims a work item, set item `status: claimed` and `assignee: <agent-id>` in that item's `index.md`; when releasing or when the item is done, clear `assignee` and set status to `open` or `done` as appropriate.
- **Task-level metadata:** Task files use `task`, `status`, `priority`, `dep`, `assignee`, `requiresHumanReview`, `expiresAt`. Discovery is by listing `work/` and task files only; no shared backlog or manifest file.
- If the repo's `SPEC.md` version is newer than the embedded spec, read `SPEC.md` and follow it.

---

## Step 1: Read current state

- Check if `.backlogmd/` exists. If not, run **Step 1b: Bootstrap** before continuing.
- List `.backlogmd/work/` to get open item directories.
- For each item directory, read `index.md` (metadata: `work`, item `status`, optional `assignee`, CONTEXT) and list task files (`<tid>-<task-slug>.md`) and read their metadata to understand current items and tasks. Ignore `-feedback.md` for discovery; read them when working on a specific task.

### Step 1b: Bootstrap (first-time setup)

If `.backlogmd/` does not exist, create the initial structure:

1. Create `.backlogmd/` directory.
2. Create `.backlogmd/work/` directory.
3. Create `.backlogmd/z-archive/` directory (optional; can be created when first archiving).

Inform the user that the backlog has been initialized, then continue to Step 2.

## Step 2: Determine intent

Based on `$ARGUMENTS`, determine which operation the user wants:

| Intent            | Trigger examples                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------ |
| **Init backlog**  | "init backlog", "set up backlogmd", "initialize" (also happens automatically if `.backlogmd/` doesn't exist) |
| **Create item**   | "add a feature for...", "new bugfix: ...", "refactor the...", "chore: ...", a work item description          |
| **Add tasks**     | "add tasks to...", "new task for..."                                                                         |
| **Update status** | "mark task X as done", "start working on...", "task X is blocked", "release task...", "stop working on..."   |
| **Edit**          | "edit task...", "update description of...", "rename item..."                                                 |
| **Archive**       | "archive item...", "clean up done items"                                                                     |
| **Show status**   | "what's the current state?", "show backlog", "what's in progress?"                                           |
| **Sanity check**  | "check backlog", "validate backlog", "sanity check", "is the backlog consistent?"                            |

If the intent is ambiguous, ask the user to clarify before proceeding.

### Inferring the Type (optional)

When creating an item, you may infer a Conventional Commits type from context to include in the slug. This is optional — slugs without a type are valid.

- Words like "add", "implement", "build", "create" → `feat`
- Words like "fix", "bug", "broken", "crash", "error" → `fix`
- Words like "refactor", "clean up", "simplify", "restructure" → `refactor`
- Words like "update deps", "migrate", "maintenance", "chore" → `chore`

If the type is unclear, omit it.

---

## Operation A: Create a new item

### A1. Propose the item and tasks

Based on `$ARGUMENTS`, propose:

1. **Item name** — short, descriptive title
2. **Type** (optional) — Conventional Commits type to include in the slug (e.g. `feat`, `fix`, `refactor`, `chore`)
3. **Tasks** — break the item into concrete implementation tasks. For each task propose:
   - Task name (`task`)
   - Short description (2–3 sentences)
   - Acceptance criteria (as checkbox items)
   - Whether human review is required (`requiresHumanReview`)

Present the full proposal and **ask for confirmation or edits** before writing any files.

### A2. Item placement

After user confirms:

1. List `.backlogmd/work/` to get existing item directory names and infer existing item IDs.
2. Determine the next available `item-id` (e.g. next number after highest existing).

Then:

- **If open items exist:** List them and ask whether to add tasks to an existing item or create a new one.
- **If no open items exist:** Proceed with creating a new item folder.
- **If 20 directories already in work/:** Archive a completed item first (all tasks `done`), then create. If none can be archived, inform the user.

### A3. Write all files (no shared file)

#### 1. Create item directory and `index.md`

Create `.backlogmd/work/<item-id>-<slug>/` and `.backlogmd/work/<item-id>-<slug>/index.md` with:

- `<!-- METADATA -->` YAML: `work: <work item title>`, `status: open` (or `plan` if not ready for agents), optional `assignee: ""` (set when work item is claimed).
- `<!-- DESCRIPTION -->` (optional).
- `<!-- CONTEXT -->` (optional; add context for agents if any).

#### 2. Create task files

For each task, create `.backlogmd/work/<item-id>-<slug>/<tid>-<task-slug>.md` using the YAML task format.

- Task IDs are zero-padded to three digits and sequential within the item.
- Task slugs are lowercase kebab-case derived from the task name.
- Set initial status to `open` (or `plan` if the task needs grooming).
- Set `assignee: ""`, `expiresAt: null`.

---

## Operation B: Update task status

### B1. Identify the task

- List `.backlogmd/work/` and scan item directories for task files (`<tid>-<task-slug>.md`); read metadata to locate the task (by name, id, or slug).
- If ambiguous, list matching tasks and ask the user to pick one.

### B2. Validate the transition

Valid status flow:

```
plan ──→ open ──→ claimed ──→ in-progress ──→ done   (requiresHumanReview: false)
                            ──→ review ──→ done (requiresHumanReview: true)
Any active state ──→ block ──→ in-progress or open
```

- `plan → open`: promotion (human or authorized agent only).
- `open → in-progress`: start work — in task file set `assignee` (and optionally `expiresAt`); in item `index.md` set `status: claimed` or `in-progress` and `assignee: <agent-id>`. Verify every `dep` path resolves to a task with `status: done`.
- `in-progress → done`: only valid when `requiresHumanReview: false`. Clear task `assignee`; if all tasks in item are done, set item `status: done` and clear item `assignee`.
- `in-progress → review`: required when `requiresHumanReview: true`. Keep `assignee`. Agent must **stop**.
- `review → done`: human only. Clear `assignee`.
- Any active → `block`: set when externally blocked. `assignee` preserved.
- `block → in-progress`: when unblocked. `block → open`: when releasing claim — clear task `assignee`; if no other task in progress in that item, set item `status: open` and clear item `assignee`.
- Reject invalid transitions and explain why.

### B3. Write changes (task file only)

1. Update the task file's YAML metadata block to match the new status (and `assignee` if starting or releasing). If starting or releasing affects the work item (claim or release), update the item's `index.md` metadata: set or clear `assignee`, set `status` to `claimed`, `in-progress`, `open`, or `done` as appropriate.
2. If moving to `done`, check all acceptance criteria boxes (`- [ ]` → `- [x]`).
3. When setting `status: block`, create or append to the task's `-feedback.md` file (what was tried, why blocked, what would unblock). When releasing because stuck, append to `-feedback.md` before setting status to open.

### B4. Handle item completion

If all tasks in the item now have `status: done`:

1. Inform the user that all tasks in the item are complete.
2. Ask if they want to archive the item.

---

## Operation C: Edit an item or task

### C1. Identify the target

List `work/` and item directories; read `index.md` or task files to locate the item or task.

### C2. Present current content

Show the current content and ask the user what they want to change.

### C3. Apply edits (single file only)

1. **Task edits:** Edit only the task file (metadata, description, acceptance criteria). If renaming task/slug, the task filename can be updated (move file) and any `dep` references in other task files that point to it must be updated.
2. **Item edits:** Edit only that item's `index.md` (metadata `work`/`status`, description, CONTEXT). No task list to sync.

### C4. Confirm changes

Show the user a summary of what was changed.

---

## Operation D: Archive an item

### D1. Validate

- List task files in the item directory and read each task file's metadata. Verify **all tasks in the item have `status: done`**.
- If any tasks are not `done`, inform the user and refuse to archive.

### D2. Execute archive (single step)

1. **Move** the item folder from `.backlogmd/work/<item-id>-<slug>/` to `.backlogmd/z-archive/<YYYY>/<MM>/<item-id>-<slug>/` (create year/month directories if needed). This includes `index.md`, all task files, and any `-feedback.md` files.

### D3. Confirm

Report to the user that the item has been archived and how many open item directories remain in `work/` (out of 20).

---

## Operation E: Show backlog status

### E1. Read state from directory and files

List `.backlogmd/work/`; for each item directory read `index.md` (item title, item `status`) and list task files, then read each task file's metadata for task title and `status`.

### E2. Present a summary

Show:

- Total open items (directories in `work/`) and item `status` where relevant
- For each item: task breakdown by status (e.g. "3/5 tasks done, 1 in-progress, 1 open")
- Any tasks currently `in-progress` (and their assignees)
- Any tasks in `review` awaiting human approval
- Items ready to archive (all tasks `done`)
- Open item count (directories in `work/`) and limit (20)

---

## Operation F: Sanity check

Validate that the entire `.backlogmd/` system is consistent. Read all files and check every rule below. Report issues grouped by severity.

### F1. Read all state

- List `.backlogmd/work/` and for each item directory read `index.md` and list task files (`<tid>-<task-slug>.md`), then read each task file.
- If `z-archive/` exists, list it (read-only check).

### F2. Validate structure

- [ ] `work/` exists. No requirement for `backlog.md` or `manifest.json`.
- [ ] Every item folder under `work/` has an `index.md`.
- [ ] Task files are named `<tid>-<task-slug>.md` (zero-padded tid, kebab-case slug).
- [ ] No more than 20 directories in `work/`.

### F3. Validate formats

- [ ] Every `index.md` has `<!-- METADATA -->`, `<!-- DESCRIPTION -->`, `<!-- CONTEXT -->` and YAML with at least `work` and `status`; optional `assignee`. Item `status` is one of `plan`, `open`, `claimed`, `in-progress`, `done`. When `claimed`, `assignee` must be non-empty; when `open` or `done`, `assignee` must be empty; when `in-progress`, `assignee` may be set.
- [ ] Every task file has `<!-- METADATA -->`, `<!-- DESCRIPTION -->`, `<!-- ACCEPTANCE -->` and YAML with required fields: `task`, `status`, `priority`, `dep`, `assignee`, `requiresHumanReview`, `expiresAt`.
- [ ] Task statuses are valid (`plan`, `open`, `in-progress`, `review`, `block`, `done`).
- [ ] `dep` values are paths relative to `.backlogmd/`: `work/<item-id>-<slug>/<tid>-<task-slug>.md`; no self-references, no duplicates, no cycles (DAG).
- [ ] Item IDs and task IDs are zero-padded (min 3 digits) and unique in their scope.
- [ ] `in-progress` and `review` tasks have non-empty `assignee`. `done` tasks have empty `assignee`. Items with status `claimed` have non-empty item `assignee`.

### F4. Validate dependencies and workflow

- [ ] No circular dependencies (all `dep` paths form a DAG).
- [ ] No task is `in-progress` while any of its `dep` (resolved by path to that task file) is not `done`.
- [ ] No task with `requiresHumanReview: true` is `done` without having gone through `review`.

### F5. Validate archive

- [ ] No item folder in `z-archive/` is also present in `work/`.

### F6. Report

Present results as:

- **Errors** — spec violations (missing markers, invalid statuses, broken `dep` paths, cycles).
- **Warnings** — items with all tasks done but not archived, stale `expiresAt`.
- **OK** — checks that passed.

If errors are found, offer to fix them (with user confirmation). Task file is source of truth for task state; no manifest to reconcile.

---

## Rules

- Follow the spec formats exactly — YAML metadata in fenced code blocks, no YAML frontmatter.
- All paths are relative within `.backlogmd/`.
- **Task edits:** write only the task file. **Item edits:** write only that item's `index.md`. **Feedback:** write only the task's `-feedback.md`. Never update a shared manifest or backlog file.
- Never overwrite existing items or tasks — only create new item dirs/task files or edit in place.
- Always confirm with the user before writing or modifying files.
- Max 20 open items (directories in `work/`). If the limit is reached, the user must archive an item or use an existing one.
- The `z-archive/` directory is cold storage. After moving items into it, never modify them again.
- A completed task cannot be reopened. If the work needs revisiting, create a new task instead.
- When `requiresHumanReview: true`, agents MUST NOT move a task directly from `in-progress` to `done` — it must go through `review`.
