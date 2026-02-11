# backlogmd

A markdown-based backlog system designed for agentic development. Define work items and tasks in plain `.md` files — no databases, no SaaS lock-in, just your repo.

## Why

AI coding agents need a structured way to understand what to build next. backlogmd gives them (and you) a simple, human-readable backlog that lives in your codebase.

## How it works

Drop a `.backlogmd/` folder into any repo:

```
.backlogmd/
├── PROTOCOL.md              # Rules and formats
├── backlog.md               # Item roadmap
└── items/
    └── <item-slug>/
        ├── index.md          # Item overview + task table
        ├── 001-task-slug.md
        └── ...
```

- **Items** live in `backlog.md`, ordered by priority. Each item has a type: `feature`, `bugfix`, `refactor`, or `chore`.
- **Item folders** group tasks into deliverable chunks
- **Tasks** are individual markdown files with status, owner, and acceptance criteria

Agents read `PROTOCOL.md` to understand the format, then pick up tasks and move them through: `todo` → `in-progress` → `ready-to-review` → `ready-to-test` → `done`.

## Visualizer

Browse and explore your backlog at [backlogmd.com](https://www.backlogmd.com).

## Quick start

1. Create a `.backlogmd/` folder in your repo
2. Add a `PROTOCOL.md` (see [the spec](.backlogmd/PROTOCOL.md))
3. Add items to `backlog.md`
4. Create items and tasks
5. Point your agents at `PROTOCOL.md` and let them work

## License

MIT
