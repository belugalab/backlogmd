# Spec Changelog

**About:** The current spec version is always the one in `SPEC.md`. The spec follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`). Major bumps for breaking changes, minor for backward-compatible additions, patch for clarifications. When releasing a new major version, preserve the previous spec in `specs/` (e.g. `specs/v1.md`).

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
