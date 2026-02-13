# Specification

**Version:** 3.0.0

Single source of truth for the `.backlogmd/` system — markdown-based agile/kanban for agentic development. Agents must read this file before interacting with the backlog.

## Directory Structure

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

All paths in this document and throughout the system are relative within `.backlogmd/`.

## IDs and Naming

- **Item IDs** (`item-id`): Zero-padded integers, minimum 3 digits (e.g., `001`, `012`, `999`, `1000`). Unique across the backlog.
- **Task IDs** (`tid`): Zero-padded integers, minimum 3 digits. Unique per item.
- **Slugs**: Lowercase kebab-case. An optional Conventional Commit type may follow the ID as the first slug segment (e.g., `001-chore-project-foundation`).
- **Priority** (`p`): Integer, unique per item. **Lower number = higher priority.**

## Backlog Format (`backlog.md`)

- Bullet list of markdown links to each item's `index.md`, sorted by `item-id` ascending.
- Format: `- [<item-id>-<slug>](work/<item-id>-<slug>/index.md)`
- Completed items are removed from `backlog.md` and their folder is moved to `.archive/`.

### Example

```md
- [001-chore-project-foundation](work/001-chore-project-foundation/index.md)
- [002-ci-initialize-github-actions](work/002-ci-initialize-github-actions/index.md)
```

## Item Format (`work/<item-id>-<slug>/index.md`)

- Bullet list of relative links to task files within the same directory.
- Sorted by task id (`tid`) ascending.
- Format: `- [<tid>-<task-slug>](<tid>-<task-slug>.md)`

### Example

```md
- [001-setup-repo](001-setup-repo.md)
- [002-init-ci](002-init-ci.md)
```

## Task Format (`work/<item-id>-<slug>/<tid>-<task-slug>.md`)

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

### Rules

- Filenames: `<tid>-<task-slug>.md`; `tid` zero-padded, unique per item.
- Status codes:
  - `plan` — groomed/draft, not ready for agents to pick up.
  - `open` — ready to be claimed by an agent.
  - `reserved` — claimed, awaiting start. Requires `a` (non-empty).
  - `ip` — in progress. Requires `a` (non-empty).
  - `review` — awaiting human approval. Set when `h: true` and work is complete. Requires `a`.
  - `block` — blocked by an external dependency. `a` is preserved.
  - `done` — complete. `a` is cleared.
- `dep` values are **quoted strings** matching task IDs (e.g., `["001", "003"]`). This avoids YAML interpreting zero-padded numbers as integers. `dep` must not include the task's own `tid`, and duplicates are invalid.
- Three HTML comment markers only: `<!-- METADATA -->`, `<!-- DESCRIPTION -->`, `<!-- ACCEPTANCE -->`.
- Acceptance criteria use markdown checkboxes.
- No YAML frontmatter outside the fenced block; keep metadata lines ≤ 120 chars.

## Manifest (`manifest.json`)

Machine-readable index. Agents MUST read the manifest before acting and MUST update it alongside every file edit. The file is **strict JSON** (no trailing commas, no comments). Examples in this spec use JSONC for readability only.

### Schema (conceptual)

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
          "expiresAt": null,
        },
        {
          "tid": "002",
          "slug": "init-ci",
          "file": "002-init-ci.md",
          "t": "Initialize CI pipeline",
          "s": "reserved",
          "p": 10,
          "dep": ["001"],
          "a": "alice",
          "h": true,
          "expiresAt": "2026-02-13T15:00:00Z",
        },
      ],
    },
  ],
}
```

### Field notes

- Item `status` (`open | archived`) is distinct from task `s` — they track different lifecycles.
- Each task entry includes `slug` and `file` so agents can resolve paths without reading `index.md`.
- `t` (title) is included for quick lookups and dashboard views.
- `updatedAt` at the root MUST be set to the current ISO 8601 timestamp (UTC, with trailing `Z`) on every manifest write.
- `expiresAt` is `null` (not `""`) when unset, in both JSON and YAML.

## Claim & Human-in-the-Loop Protocol

### Write ordering

File-based systems cannot provide true atomicity. To minimize inconsistency:

- Manifest is the authoritative source for status/assignment; task files are authoritative for content. Write order is manifest → task → index.

1. **Write manifest first**, then the task file, then `index.md` (if affected).
2. If a write fails midway, the manifest is treated as the **authoritative source for status and assignment**. Task file content (description, acceptance criteria) is authoritative for content.
3. Agents encountering a mismatch between manifest and file should trust the manifest for `s`, `a`, `p`, `dep`, `h`, `expiresAt`, and update the task file to match.

### Claiming a task

1. Agent re-reads the manifest. If the task `s` is `open`, the agent sets `s: reserved`, `a: <agent-id>`, and optionally `expiresAt` (ISO 8601, short horizon — e.g., 30 min). Update manifest first, then task file.
2. Only `open` tasks may be claimed. `plan` tasks must first be promoted to `open` by a human or authorized agent.
3. `plan` cannot transition directly to `reserved`; it must go through `open` before claiming.

### Starting work

4. Set `s: ip` (keep `a`). A task cannot move to `ip` if any task ID in its `dep` list has `s` other than `done`.

### Completing work

5. If `h: false`: set `s: done` and clear `a`.
6. If `h: true`: agent MUST set `s: review` and **stop**. Direct `ip → done` is **invalid** when `h: true`. Only a human (or authorized role) may move `review → done`.

### Releasing a claim

7. To unclaim, set `s: open` and clear `a` and `expiresAt`.

### Expiry

8. If `expiresAt` is non-null and in the past, another agent may reclaim by setting `s: reserved`, overwriting `a` with its own ID, and setting a fresh `expiresAt`.
9. `expiresAt` applies only in `reserved` and `ip`; ignore it while a task is in `review`.

### Blocking

8. Set `s: block` when an external dependency prevents progress. `a` remains set. Valid transitions out of `block`: back to `ip` (when unblocked) or to `open` (if releasing the claim).

## Archive

- Cold storage; agents skip `.archive/`.
- An item is archived **only when every task in the item has `s: done`**.
- Archive procedure (execute in order):
  1. Move the item folder to `.archive/<YYYY>/<MM>/<item-id>-<slug>/`.
  2. Remove the item's entry from `backlog.md`.
  3. Set item `status: "archived"` in `manifest.json` (keep the item entry for history; task list may be trimmed).
- Archived item entries remain in the manifest for history; `openItemCount` excludes items with `status: "archived"`. Task lists in archived items may be trimmed as needed. Archive contents are read-only.

## Limits

- Max 50 open items (`openItemCount <= 50`).
- Recommended max 20 tasks per item. Items with more should be split into a new item.
- Archival steps must be executed in the prescribed order (folder move → backlog removal → manifest update).

## Workflow Rules

### Status flow

```
plan ──→ open ──→ reserved ──→ ip ──→ done           (h: false)
                                  ──→ review ──→ done (h: true)

Any active state ──→ block ──→ ip or open
```

- `plan → open`: promotion (human or authorized agent only).
- `open → reserved`: claim (any agent).
- `reserved → ip`: start work (assigned agent).
- `ip → done`: completion (only valid when `h: false`).
- `ip → review → done`: completion with human gate (required when `h: true`).
- `block`: reachable from `reserved`, `ip`, or `review`. Returns to `ip` (unblocked) or `open` (released).
- A task cannot move to `ip` if any `dep` is not `done`.
- No circular dependencies. Agents adding `dep` entries must verify no cycle is introduced; rule of thumb: each `dep` should reference a task with `tid` lower than the current task to keep the DAG by numbering (recommended).
- Task edits must update both the task file and manifest (manifest first).
- When all tasks in an item are `done`, archive the item.

## Reconciliation

If the manifest and task files disagree:

- **Status/assignment fields** (`s`, `a`, `p`, `dep`, `h`, `expiresAt`): manifest wins. Update the task file to match.
- If a task is `done`, `a` MUST be empty in both manifest and task file; clear it during reconciliation.
- **Content fields** (description, acceptance criteria): task file wins. Update the manifest `t` if the title changed in the file.
- **`index.md` link list**: regenerate from the task files present on disk, sorted by `tid`.
- If `index.md` is missing links for existing tasks, regenerate; if it has links to missing files, remove them

Agents should reconcile on read if a mismatch is detected, and log a warning.

## Conventions

- All paths are relative within `.backlogmd/`.
- Priority numbers are unique per item (lower = higher priority).
- `index.md` must stay in sync with task files on disk.
- No YAML frontmatter outside the fenced code block.
- Keep metadata lines ≤ 120 chars.
- Avoid inline HTML outside the three comment markers.

## Versioning

- Semantic Versioning (`MAJOR.MINOR.PATCH`).
- This file is 3.0.0. Prior specs live in `specs/`.
- See `SPEC-CHANGELOG.md` for history and migrations.
- Agents may reject if `specVersion` in `manifest.json` is unsupported.
