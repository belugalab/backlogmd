# Spec Changelog

**About:** The current spec version is always the one in `SPEC.md`. The spec follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`). Major bumps for breaking changes, minor for backward-compatible additions, patch for clarifications. When releasing a new major version, preserve the previous spec in `specs/` (e.g. `specs/v1.md`).

---

## 4.0.4

- **Summary:** Renamed `.archive/` to `z-archive/` so that in alphabetical listings (e.g. `ls`) `work/` appears first and archive second. The `z-` prefix ensures the archive folder sorts after `work/`.
- **Breaking changes:**
  - **Archive directory** — Path is now `z-archive/<YYYY>/<MM>/<item-id>-<slug>/` (was `.archive/...`). Agents skip `z-archive/` for active work.
- **Migration (4.0.3 → 4.0.4):** Rename `.backlogmd/.archive/` to `.backlogmd/z-archive/` (move existing archived item folders into the new path if any).

---

## 4.0.3

- **Summary:** Renamed item-level metadata key from `task` to `work` in `index.md` so it is not confused with task files (which use `task` for the task title). The work item title is now `work:`.
- **Breaking changes:**
  - **Item Format** — In `work/<item-id>-<slug>/index.md`, the METADATA YAML must use `work: <title>` (not `task:`) for the work item title.
- **Migration (4.0.2 → 4.0.3):** In every item `index.md`, rename the key `task:` to `work:` in the METADATA block (value unchanged).

---

## 4.0.2

- **Summary:** Removed the claim step. Tasks go directly from `open` to `in-progress` when an agent starts work; the `reserved` status is removed.
- **Breaking changes:**
  - **Task status `reserved` removed** — No longer a valid status. Flow is now `plan → open → in-progress → done` (or `→ review → done`).
- **Changed:**
  - **Starting work:** Agent sets `status: in-progress`, `assignee`, and optionally `expiresAt` in one step (no separate "claim" with `reserved`).
  - **Expiry:** Another agent may "take over" by setting `status: in-progress` and a new `assignee` (no `reserved`).
  - **Protocol section** renamed from "Claim & Human-in-the-Loop Protocol" to "Human-in-the-Loop Protocol". Subsections "Claiming a task" and "Starting work" merged into "Starting work"; "Releasing a claim" renamed to "Releasing".
  - **Status flow diagram:** `open → in-progress` (no `reserved` in between). `block` reachable from `in-progress` or `review` only.
- **Migration (4.0.1 → 4.0.2):** Any task with `status: reserved` should be treated as `in-progress` (or move to `open` if the assignee released). Update task files to use only the remaining statuses.

---

## 4.0.1

- **Summary:** Item `index.md` no longer contains a task list. It now has metadata (YAML), an optional description, and a `<!-- CONTEXT -->` section for agents. Tasks are discovered by listing the item directory for `<tid>-<task-slug>.md` files. Metadata keys use full names (no shorthand).
- **Breaking changes:**
  - **Item Format** — `work/<item-id>-<slug>/index.md` is no longer a bullet list of task links. It has three sections: `<!-- METADATA -->` (YAML, e.g. item title `task`), `<!-- DESCRIPTION -->`, and `<!-- CONTEXT -->`. The CONTEXT block is read and used by agents when working on any task in that item.
  - **Metadata keys** — Task and item METADATA use full key names: `task` (was `t`), `status` (was `s` for tasks), `priority` (was `p`), `assignee` (was `a`), `requiresHumanReview` (was `h`).
- **Changed:**
  - **Task discovery** — Agents discover tasks by listing each item directory for files matching `<tid>-<task-slug>.md` (excluding `index.md`).
  - **Write ordering** — Task edits touch only the task file. Item-level edits (metadata, description, CONTEXT) touch only `index.md`. No link list to keep in sync.
  - **Claiming** — Agent SHOULD read the item's `index.md` (especially `<!-- CONTEXT -->`) when working on a task in that item.
  - **Reconciliation** — No task list in `index.md`; nothing to regenerate.
  - **Conventions** — Replaced "index.md must stay in sync with task files" with "task files are discovered by listing the item directory for `<tid>-<task-slug>.md`".
- **Migration (4.0.0 → 4.0.1):** Convert each existing `index.md` from a task-link list to the new structure (METADATA with at least `task: <item title>`, optional DESCRIPTION, CONTEXT). In task files and index METADATA, rename keys: `t` → `task`, `s` → `status`, `p` → `priority`, `a` → `assignee`, `h` → `requiresHumanReview`. Populate CONTEXT as needed for agents. Task discovery will be by directory listing from 4.0.1 onward.

---

## 4.0.0

- **Summary:** Removed `manifest.json` and `backlog.md` from the spec. Shared files that multiple agents write to cause merge conflicts; the system uses only directory structure and per-item task files. Open items are directories under `work/`; agents discover work by listing `work/` and reading task files.
- **Breaking changes:**
  - **`manifest.json` removed** — No longer part of directory structure. Agents must not read or write a manifest. All status, assignment, and content are read from and written to task files only.
  - **`backlog.md` removed** — No longer part of directory structure. Agents must not read or write a backlog file. Open items are the directories under `work/`.
  - **Write ordering** — Was manifest → task → index. Now: task file first, then `index.md` if affected.
  - **Claiming / starting work** — Agents list `work/` and read task files to find tasks with `s: open`. Dependency resolution is done by reading the task file at each `dep` path (no manifest lookup).
  - **Archive** — Procedure is a single step: move the item folder to `.archive/<YYYY>/<MM>/<item-id>-<slug>/`. No shared file to update.
  - **Reconciliation** — Source of truth is task files and directory structure only. Only `index.md` may need repair: regenerate from task files on disk if links are missing or stale.
  - **Versioning** — Spec version is only in `SPEC.md`. Removed "Agents may reject if `specVersion` in `manifest.json` is unsupported"; replaced with "Agents may reject if the spec version in this file is unsupported."
- **Removed:**
  - Entire **Manifest (`manifest.json`)** section (schema, field notes).
  - Entire **Backlog Format (`backlog.md`)** section (format, example).
  - All references to manifest and `backlog.md` in Claim protocol, Archive, Limits, Workflow rules, and Reconciliation.
- **Added:**
  - **Open items** — Short section stating that open items = directories under `work/`, archived = under `.archive/`, and discovery is by listing `work/` and reading task files.
- **Changed:**
  - **Limits:** "Max 50 open items" = max 50 directories in `work/`.
  - **Reconciliation:** Describes only `index.md` repair from task files on disk; no shared backlog file.
- **Migration (3.x → 4.0.0):** Remove `manifest.json` and `backlog.md` from `.backlogmd/` if present. Ensure all task metadata is correct in task files; agents use only directory structure and per-item files from 4.0.0 onward.

---

## 3.0.1

- **Summary:** Clarified and standardized task dependency format. Dependencies are now explicit work/task paths so agents know exactly which item and task must be done before starting.
- **Breaking changes:**
  - **`dep` format** — Previously `dep` was a list of task ID strings within the same item (e.g. `["001", "003"]`). It is now a list of **paths** relative to `.backlogmd/`: `work/<item-id>-<slug>/<tid>-<task-slug>.md` (e.g. `["work/002-ci-initialize-github-actions/001-ci-cd-setup.md"]`). Same format in both task file YAML and `manifest.json`.
- **Added:**
  - **Dependencies** subsection under Task Format: defines path format, allows cross-item dependencies, and states that agents must wait for each referenced task to be `s: done` before moving a task to `ip`.
  - Manifest field note: task `dep` in the manifest uses the same path format; cross-item dependencies are allowed.
- **Changed:**
  - Claim protocol "Starting work" (rule 4): a task cannot move to `ip` until every `dep` entry is resolved to a task in the manifest and that task has `s === "done"`.
  - Workflow rules: dependency rule and cycle check now apply across the whole backlog (DAG over all tasks and their `dep` paths), not only within an item.
- **Migration (3.0.0 → 3.0.1):** For each task that has `dep` with the old format (e.g. `["001", "002"]`), rewrite each ID to the full path for the same item: `work/<this-item-path>/<tid>-<task-slug>.md`. Example: in item `001-chore-project-foundation`, `dep: ["002"]` becomes `dep: ["work/001-chore-project-foundation/002-docker-setup.md"]` (use the actual task file name from that item). Cross-item deps were not expressible in 3.0.0; add them using the new path format.

---

## 3.0.0

- **Summary:** Introduced machine-readable manifest, YAML task metadata with short keys, agent claim/reservation protocol, human-in-the-loop gating, reconciliation rules, and expanded status lifecycle.
- **Breaking changes:**
  - Task metadata format changed from plain key-value fenced code block (`Task:`, `Status:`, `Priority:`, `DependsOn:`) to YAML fenced code block with short keys (`t`, `s`, `p`, `dep`, `a`, `h`, `expiresAt`).
  - `DependsOn` (markdown link to file) replaced by `dep` (list of quoted task ID strings, e.g., `["001", "003"]`).
  - HTML comment markers simplified from paired open/close tags (`<!-- METADATA -->` / `<!-- /METADATA -->`) to opening tags only (`<!-- METADATA -->`, `<!-- DESCRIPTION -->`, `<!-- ACCEPTANCE -->`).
  - `<!-- ACCEPTANCE CRITERIA -->` renamed to `<!-- ACCEPTANCE -->`.
  - Status `in-progress` shortened to `ip`; new required statuses added (`plan`, `reserved`, `review`).
  - `backlog.md` ordering changed from priority-ordered to sorted by `item-id` ascending.
  - `manifest.json` is now required — agents MUST read and update it alongside every file edit.
- **Added:**
  - `manifest.json` — machine-readable index that agents MUST read before acting and update alongside every file edit. Includes `specVersion`, `updatedAt`, `openItemCount`, and full item/task entries with resolved paths.
  - **IDs and Naming** section formalizing `item-id` (zero-padded, min 3 digits, unique across backlog), `tid` (zero-padded, min 3 digits, unique per item), slugs, and priority rules.
  - New task statuses: `plan` (groomed/draft, not claimable), `reserved` (claimed, awaiting start), `ip` (replaces `in-progress`), `review` (human approval gate).
  - New task metadata fields: `a` (assignee/agent id), `h` (human review required flag), `expiresAt` (ISO 8601 reservation expiry or `null`).
  - **Claim & Human-in-the-Loop Protocol** — write ordering (manifest → task → index), claim/release/expiry semantics, `h: true` enforces `ip → review → done` (direct `ip → done` invalid).
  - **Reconciliation** rules — manifest is authoritative for status/assignment fields; task file is authoritative for content; `index.md` regenerated from disk.
  - Recommended max 20 tasks per item (items with more should be split).
  - Archival steps must be executed in prescribed order (folder move → backlog removal → manifest update).
  - Archived item entries remain in manifest with `status: "archived"`; `openItemCount` excludes them.
  - Convention: keep metadata lines ≤ 120 chars; avoid inline HTML outside the three comment markers.
- **Changed:**
  - Item `index.md` task list explicitly sorted by `tid` ascending.
  - Item directory naming explicitly documented as `<item-id>-<slug>`.
  - Archive procedure formalized as a strict 3-step process including manifest update.
  - Workflow rules expanded with formal status flow diagram, `plan → open` promotion gate, DAG enforcement for dependencies, and requirement to update both task file and manifest on every edit.

## 2.0.0

- **Summary:** Simplified protocol. Renamed `items/` to `work/`, simplified backlog to a plain slug list, replaced item task tables with bullet lists, introduced HTML comment sections and code block metadata in task files, reduced task statuses to four, removed Owner/Blocks/Goal/Item-backlink fields.
- **Breaking changes:**
  - Directory renamed: `items/` → `work/`.
  - `backlog.md` format changed from structured markdown (headers, Type, Status, Item link, Description) to a bullet list of markdown links to each item's `index.md`.
  - Item `index.md` changed from metadata + task table to a bullet list of task links.
  - Task file format changed from plain markdown fields to HTML comment sections (`<!-- METADATA -->`, `<!-- DESCRIPTION -->`, `<!-- ACCEPTANCE CRITERIA -->`) with a fenced code block for metadata.
  - Type is now embedded in the item slug (e.g. `001-chore-project-foundation`) instead of a separate field.
  - Task statuses reduced: `todo → in-progress → ready-to-review → ready-to-test → done` replaced by `open → in-progress → done` (plus `block`).
  - Removed fields: Owner, Blocks, Item backlink, Goal.
  - Removed derived status logic from `backlog.md` — backlog is now a plain list with no status.
  - Archive simplified: `.archive/backlog.md` and `.archive/items/` replaced by `.archive/` containing item folders directly.

## 1.0.0

- **Summary:** Initial version. Item folders under `items/`, roadmap in `backlog.md`, task files with status and acceptance criteria, archive for completed work. Items support typed work: feature, bugfix, refactor, chore.
- **Changes:**
  - Directory layout: `backlog.md`, `items/<item-slug>/` with `index.md` and task files, `.archive/` for completed items and archived item folders.
  - Roadmap format: items with Type, Status, Item (link to folder), Description.
  - Item format: `items/<item-slug>/index.md` with Type, Status, Goal, and Tasks table.
  - Task format: `items/<item-slug>/<NNN>-<task-slug>.md` with Status, Priority, Owner, Item link, Description, Acceptance Criteria; optional Depends on / Blocks.
  - Type field: `feature`, `bugfix`, `refactor`, `chore` — extensible per project.
  - Derived status logic: all done → done; any in-progress/review/test → in-progress; mix of done and todo → in-progress; all todo → todo.
  - Archive: completed items appended to `.archive/backlog.md`; archived item folders moved to `.archive/items/<slug>/`.
  - Limits: max 10 open items; only `open` items accept new tasks.
  - Versioning: semver, protocol version stated at top of `SPEC.md` (**Version:** 1.0.0).
