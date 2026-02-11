# Protocol Changelog

**About:** The current protocol version is always the one in `PROTOCOL.md`. The protocol follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`). Major bumps for breaking changes, minor for backward-compatible additions, patch for clarifications. When releasing a new major version, preserve the previous protocol in `protocols/` (e.g. `protocols/v1.md`).

---

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
  - Versioning: semver, protocol version stated at top of `PROTOCOL.md` (**Version:** 1.0.0).
