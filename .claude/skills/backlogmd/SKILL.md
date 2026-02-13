---
name: backlogmd
description: Use when planning work (to create items and tasks), when starting implementation (to claim and work tasks), when completing work (to mark tasks done), or to check backlog status. Manages .backlogmd/ with manifest.json for features, bugfixes, refactors, and chores.
argument-hint: init/create/start/done/edit/show/archive/check <item or task>
allowed-tools: Read, Write, Edit, Glob, Bash(mkdir *), Bash(mv *)
---

# Backlog Manager

You are an agent that manages the `.backlogmd/` backlog system. You can create items (features, bugfixes, refactors, chores) and tasks, claim and release tasks, update statuses, edit content, and archive completed work. The manifest (`manifest.json`) is the machine-readable index — read it before acting and update it alongside every file edit.

## Workflow (MANDATORY)

> **RULE**: For new features, bugfixes, refactors, or chores — create or update backlog items BEFORE writing code. The backlog is the source of truth for planned work. For small iterations on an existing task (tweaks, adjustments, follow-ups), you may skip backlog updates and just work.

1. **Before planning**: Read `manifest.json` and check the backlog for existing items and tasks.
2. **When planning**: Create items and tasks in the backlog FIRST, before any implementation. Don't just describe plans in conversation — record them. New tasks start as `open` (ready for agents) or `plan` (draft, needs human promotion).
3. **Wait for approval**: After planning, present the plan to the user and **STOP**. Do NOT start implementing until the user explicitly approves.
4. **When implementing**: Follow this loop for EACH task, one at a time:
   - **Claim** the task: set `s: reserved`, `a: <agent-id>`.
   - **Start** the task: set `s: ip` (verify deps are `done` first).
   - **Implement** the task.
   - **Complete** the task: if `h: false`, set `s: done` and clear `a`. If `h: true`, set `s: review` and **stop** — only a human may move `review → done`.
   - **Only then** move to the next task.
   - **Always** write manifest first, then task file, then `index.md`.
5. **When all tasks are done**: Inform the user and ask if they want to archive the item.

---

## Spec v3.0.0 (embedded)

### Directory Structure

```
.backlogmd/
├── backlog.md
├── manifest.json
├── work/
│   ├── <item-id>-<slug>/
│   │   ├── index.md
│   │   ├── 001-task-slug.md
│   │   ├── 002-task-slug.md
│   │   └── ...
│   └── ...
└── .archive/
    └── <YYYY>/<MM>/<item-id>-<slug>/
```

All paths are relative within `.backlogmd/`.

### IDs and Naming

- **Item IDs** (`item-id`): Zero-padded integers, minimum 3 digits (e.g., `001`, `012`, `999`, `1000`). Unique across the backlog.
- **Task IDs** (`tid`): Zero-padded integers, minimum 3 digits. Unique per item.
- **Slugs**: Lowercase kebab-case. An optional Conventional Commit type may follow the ID as the first slug segment (e.g., `001-chore-project-foundation`).
- **Priority** (`p`): Integer, unique per item. **Lower number = higher priority.**

### Backlog Format (`backlog.md`)

```md
- [001-chore-project-foundation](work/001-chore-project-foundation/index.md)
- [002-ci-initialize-github-actions](work/002-ci-initialize-github-actions/index.md)
```

- Bullet list of markdown links to each item's `index.md`, sorted by `item-id` ascending.
- Format: `- [<item-id>-<slug>](work/<item-id>-<slug>/index.md)`
- Completed items are removed from `backlog.md` and their folder is moved to `.archive/`.

### Item Format (`work/<item-id>-<slug>/index.md`)

```md
- [001-setup-repo](001-setup-repo.md)
- [002-init-ci](002-init-ci.md)
```

- Bullet list of relative links to task files within the same directory.
- Sorted by task id (`tid`) ascending.
- Format: `- [<tid>-<task-slug>](<tid>-<task-slug>.md)`

### Task Format (`work/<item-id>-<slug>/<tid>-<task-slug>.md`)

````md
<!-- METADATA -->

```yaml
t: Add login form
s: plan # plan | open | reserved | ip | review | block | done
p: 10 # priority within item (lower = higher priority)
dep: ["002"] # optional list of task id strings within the same item
a: "" # assignee/agent id; empty string if unassigned
h: false # true if human review required before done
expiresAt: null # ISO 8601 timestamp for reservation expiry, or null
```

<!-- DESCRIPTION -->

## Description

<detailed description>

<!-- ACCEPTANCE -->

## Acceptance criteria

- [ ] <criterion>
````

- Filenames: `<tid>-<task-slug>.md`; `tid` zero-padded, unique per item.
- Status codes:
  - `plan` — groomed/draft, not ready for agents to pick up.
  - `open` — ready to be claimed by an agent.
  - `reserved` — claimed, awaiting start. Requires `a` (non-empty).
  - `ip` — in progress. Requires `a` (non-empty).
  - `review` — awaiting human approval. Set when `h: true` and work is complete. Requires `a`.
  - `block` — blocked by an external dependency. `a` is preserved.
  - `done` — complete. `a` is cleared.
- `dep` values are **quoted strings** matching task IDs (e.g., `["001", "003"]`). `dep` must not include the task's own `tid`, and duplicates are invalid.
- Three HTML comment markers only: `<!-- METADATA -->`, `<!-- DESCRIPTION -->`, `<!-- ACCEPTANCE -->`.
- Acceptance criteria use markdown checkboxes.
- No YAML frontmatter outside the fenced block; keep metadata lines ≤ 120 chars.

### Manifest (`manifest.json`)

Machine-readable index. Agents MUST read the manifest before acting and MUST update it alongside every file edit. The file is **strict JSON** (no trailing commas, no comments).

```jsonc
{
  "specVersion": "3.0.0",
  "updatedAt": "2026-02-13T12:00:00Z", // set on every manifest write
  "openItemCount": 1,
  "items": [
    {
      "id": "001",
      "slug": "chore-project-foundation",
      "path": "work/001-chore-project-foundation",
      "status": "open", // open | archived
      "updated": "2026-02-13T11:59:00Z",
      "tasks": [
        {
          "tid": "001",
          "slug": "setup-repo",
          "file": "001-setup-repo.md",
          "t": "Set up repository structure",
          "s": "done",
          "p": 5,
          "dep": [],
          "a": "",
          "h": false,
          "expiresAt": null
        }
      ]
    }
  ]
}
```

- Item `status` (`open | archived`) is distinct from task `s`.
- Each task entry includes `slug` and `file` so agents can resolve paths without reading `index.md`.
- `updatedAt` at the root MUST be set to current ISO 8601 timestamp (UTC, trailing `Z`) on every write.
- `expiresAt` is `null` (not `""`) when unset, in both JSON and YAML.

### Claim & Human-in-the-Loop Protocol

**Write ordering:** manifest → task file → `index.md` (if affected).

- Manifest is authoritative for status/assignment; task files are authoritative for content.
- If a write fails midway, trust the manifest for `s`, `a`, `p`, `dep`, `h`, `expiresAt`.

**Claiming:** Re-read manifest. If `s` is `open`, set `s: reserved`, `a: <agent-id>`, optionally `expiresAt`. Only `open` tasks may be claimed (`plan` must be promoted to `open` first).

**Starting work:** Set `s: ip` (keep `a`). A task cannot move to `ip` if any `dep` is not `done`.

**Completing:** If `h: false`: set `s: done`, clear `a`. If `h: true`: set `s: review` and **stop** — direct `ip → done` is invalid when `h: true`.

**Releasing:** Set `s: open`, clear `a` and `expiresAt`.

**Expiry:** If `expiresAt` is past, another agent may reclaim. Applies only in `reserved` and `ip`.

**Blocking:** Set `s: block` when externally blocked. `a` remains. Transitions out: `ip` (unblocked) or `open` (released).

### Status Flow

```
plan ──→ open ──→ reserved ──→ ip ──→ done           (h: false)
                                  ──→ review ──→ done (h: true)

Any active state ──→ block ──→ ip or open
```

### Archive

- Cold storage; agents skip `.archive/`.
- Archive **only when every task in the item has `s: done`**.
- Procedure (execute in order):
  1. Move the item folder to `.archive/<YYYY>/<MM>/<item-id>-<slug>/`.
  2. Remove the item's entry from `backlog.md`.
  3. Set item `status: "archived"` in `manifest.json` (keep the entry for history).
- `openItemCount` excludes archived items. Archive contents are read-only.

### Reconciliation

- **Status/assignment fields** (`s`, `a`, `p`, `dep`, `h`, `expiresAt`): manifest wins. Update task file to match.
- **Content fields** (description, acceptance criteria): task file wins. Update manifest `t` if title changed.
- **`index.md`**: regenerate from task files on disk, sorted by `tid`.

### Limits

- Max 50 open items (`openItemCount <= 50`).
- Recommended max 20 tasks per item. Items with more should be split.

---

## Step 1: Read current state

- Check if `.backlogmd/` exists. If not, run **Step 1b: Bootstrap** before continuing.
- Read `.backlogmd/manifest.json` for the full item/task index.
- Read `.backlogmd/backlog.md` to understand current items.
- If manifest is missing or stale, scan `.backlogmd/work/` and reconcile.

### Step 1b: Bootstrap (first-time setup)

If `.backlogmd/` does not exist, create the initial structure:

1. Create `.backlogmd/` directory.
2. Create `.backlogmd/backlog.md` (empty file).
3. Create `.backlogmd/work/` directory.
4. Create `.backlogmd/manifest.json`:

```json
{
  "specVersion": "3.0.0",
  "updatedAt": "<current ISO 8601 UTC>",
  "openItemCount": 0,
  "items": []
}
```

Inform the user that the backlog has been initialized, then continue to Step 2.

## Step 2: Determine intent

Based on `$ARGUMENTS`, determine which operation the user wants:

| Intent            | Trigger examples                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------ |
| **Init backlog**  | "init backlog", "set up backlogmd", "initialize" (also happens automatically if `.backlogmd/` doesn't exist) |
| **Create item**   | "add a feature for...", "new bugfix: ...", "refactor the...", "chore: ...", a work item description          |
| **Add tasks**     | "add tasks to...", "new task for..."                                                                         |
| **Update status** | "mark task X as done", "start working on...", "task X is blocked", "claim task...", "release task..."        |
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
   - Task name (`t`)
   - Short description (2–3 sentences)
   - Acceptance criteria (as checkbox items)
   - Whether human review is required (`h`)

Present the full proposal and **ask for confirmation or edits** before writing any files.

### A2. Item placement

After user confirms:

1. Read `manifest.json` to get current `openItemCount` and existing item IDs.
2. Determine the next available `item-id`.

Then:

- **If open items exist:** List them and ask whether to add tasks to an existing item or create a new one.
- **If no open items exist:** Proceed with creating a new item folder.
- **If 50 items already exist:** Archive a completed item first (all tasks `done`), then create. If none can be archived, inform the user.

### A3. Write all files (manifest first)

#### 1. Update `manifest.json`

Add the new item entry with all task entries. Set `updatedAt` to now.

#### 2. Create task files

For each task, create `.backlogmd/work/<item-id>-<slug>/<tid>-<task-slug>.md` using the YAML task format.

- Task IDs are zero-padded to three digits and sequential within the item.
- Task slugs are lowercase kebab-case derived from the task name.
- Set initial status to `open` (or `plan` if the task needs grooming).
- Set `a: ""`, `expiresAt: null`.

#### 3. Create item `index.md`

Create `.backlogmd/work/<item-id>-<slug>/index.md` with bullet list of task links sorted by `tid`.

#### 4. Append item to `backlog.md`

Add a new entry sorted by `item-id` ascending:

```
- [<item-id>-<slug>](work/<item-id>-<slug>/index.md)
```

---

## Operation B: Update task status

### B1. Identify the task

- Read `manifest.json` to locate the task.
- If ambiguous, list matching tasks and ask the user to pick one.

### B2. Validate the transition

Valid status flow:

```
plan ──→ open ──→ reserved ──→ ip ──→ done       (h: false)
                                  ──→ review ──→ done (h: true)
Any active state ──→ block ──→ ip or open
```

- `plan → open`: promotion (human or authorized agent only).
- `open → reserved`: claim — set `a` to agent ID, optionally set `expiresAt`.
- `reserved → ip`: start work — verify all `dep` tasks are `done`.
- `ip → done`: only valid when `h: false`. Clear `a`.
- `ip → review`: required when `h: true`. Keep `a`. Agent must **stop**.
- `review → done`: human only. Clear `a`.
- Any active → `block`: set when externally blocked. `a` preserved.
- `block → ip`: when unblocked. `block → open`: when releasing claim (clear `a`).
- Reject invalid transitions and explain why.

### B3. Write changes (manifest first)

1. Update the task entry in `manifest.json`. Set `updatedAt` to now.
2. Update the task file's YAML metadata block to match.
3. If moving to `done`, check all acceptance criteria boxes (`- [ ]` → `- [x]`).

### B4. Handle item completion

If all tasks in the item now have `s: done`:

1. Inform the user that all tasks in the item are complete.
2. Ask if they want to archive the item.

---

## Operation C: Edit an item or task

### C1. Identify the target

Read `manifest.json` to locate the item or task.

### C2. Present current content

Show the current content and ask the user what they want to change.

### C3. Apply edits (manifest first)

1. Update `manifest.json` (e.g. `t` if title changed, `p`, `dep`, `h`). Set `updatedAt`.
2. Edit the task file with the requested changes.
3. If editing a task's name/slug, update the item's `index.md` link to keep it in sync.

### C4. Confirm changes

Show the user a summary of what was changed.

---

## Operation D: Archive an item

### D1. Validate

- Read `manifest.json` and verify **all tasks in the item have `s: done`**.
- If any tasks are not `done`, inform the user and refuse to archive.

### D2. Execute archive (strict order)

1. **Move** the item folder from `.backlogmd/work/<item-id>-<slug>/` to `.backlogmd/.archive/<YYYY>/<MM>/<item-id>-<slug>/` (create year/month directories if needed).
2. **Remove** the item entry from `.backlogmd/backlog.md`.
3. **Update** `manifest.json`: set item `status: "archived"`, decrement `openItemCount`, set `updatedAt`. Keep the item entry for history.

### D3. Confirm

Report to the user that the item has been archived and how many open item slots remain.

---

## Operation E: Show backlog status

### E1. Read manifest

Read `manifest.json` for the full index. Cross-check with `backlog.md` if needed.

### E2. Present a summary

Show:

- Total open items and their types
- For each item: task breakdown by status (e.g. "3/5 tasks done, 1 ip, 1 open")
- Any tasks currently `ip` or `reserved` (and their assignees)
- Any tasks in `review` awaiting human approval
- Items ready to archive (all tasks `done`)
- Open item slots remaining (out of 50)

---

## Operation F: Sanity check

Validate that the entire `.backlogmd/` system is consistent. Read all files and check every rule below. Report issues grouped by severity.

### F1. Read all state

- Read `manifest.json`.
- Read `backlog.md`.
- Scan `work/` and read every `index.md` and task file.
- If `.archive/` exists, scan it too (read-only check).

### F2. Validate structure

- [ ] `backlog.md` exists.
- [ ] `manifest.json` exists and is valid JSON.
- [ ] `manifest.json` `specVersion` is `"3.0.0"`.
- [ ] Every item in `backlog.md` has a corresponding folder in `work/`.
- [ ] Every item folder has an `index.md`.
- [ ] Every task file referenced in an item's `index.md` exists.
- [ ] No orphan task files (files in an item folder that aren't listed in `index.md`).
- [ ] No more than 50 open items (`openItemCount <= 50`).

### F3. Validate formats

- [ ] Every `backlog.md` entry follows `- [<item-id>-<slug>](work/<item-id>-<slug>/index.md)`, sorted by `item-id` ascending.
- [ ] Every `index.md` is a bullet list of task links sorted by `tid` ascending.
- [ ] Every task file has `<!-- METADATA -->`, `<!-- DESCRIPTION -->`, and `<!-- ACCEPTANCE -->` markers.
- [ ] Every task has YAML metadata with required fields: `t`, `s`, `p`, `dep`, `a`, `h`, `expiresAt`.
- [ ] Task statuses are valid (`plan`, `open`, `reserved`, `ip`, `review`, `block`, `done`).
- [ ] `dep` values are quoted strings matching task IDs; no self-references or duplicates.
- [ ] Item IDs and task IDs are zero-padded (min 3 digits) and unique in their scope.
- [ ] Slugs are lowercase kebab-case.
- [ ] `reserved`, `ip`, `review` tasks have non-empty `a`. `done` tasks have empty `a`.
- [ ] No YAML frontmatter outside the fenced code block.

### F4. Validate manifest consistency

- [ ] Every item in `manifest.json` with `status: "open"` has a folder in `work/`.
- [ ] Every task in manifest matches its task file for `s`, `a`, `p`, `dep`, `h`, `expiresAt`.
- [ ] `openItemCount` matches the actual count of non-archived items.
- [ ] Manifest `t` matches the task file's `t` field.

### F5. Validate dependencies and workflow

- [ ] No circular dependencies within an item.
- [ ] No task is `ip` while any of its `dep` tasks is not `done`.
- [ ] No task with `h: true` has transitioned directly from `ip` to `done` (should go through `review`).

### F6. Validate archive

- [ ] Archived item folders in `.archive/` are not also present in `work/`.
- [ ] `backlog.md` does not reference archived items.
- [ ] Archived items in manifest have `status: "archived"`.

### F7. Report

Present results as:

- **Errors** — spec violations that must be fixed (missing files, broken links, invalid statuses, manifest mismatches).
- **Warnings** — potential issues (items with all tasks done but not archived, stale `expiresAt`).
- **OK** — checks that passed.

If errors are found, offer to fix them automatically (with user confirmation before writing). For manifest/file mismatches, follow reconciliation rules (manifest wins for status/assignment, task file wins for content).

---

## Rules

- Follow the spec formats exactly — YAML metadata in fenced code blocks, no YAML frontmatter.
- All paths are relative within `.backlogmd/`.
- **Always update `manifest.json` first**, then task files, then `index.md`.
- Never overwrite existing items or tasks — only append (for creation) or edit in place (for updates).
- Always confirm with the user before writing or modifying files.
- Max 50 open items in `work/`. If the limit is reached, the user must archive an item or use an existing one.
- The `.archive/` directory is cold storage. After moving items into it, never modify them again.
- A completed task cannot be reopened. If the work needs revisiting, create a new task instead.
- When `h: true`, agents MUST NOT move a task directly from `ip` to `done` — it must go through `review`.
