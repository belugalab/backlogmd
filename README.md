# backlogmd

A markdown-based backlog system designed for agentic development. Define features, sprints, and tasks in plain `.md` files — no databases, no SaaS lock-in, just your repo.

## Why

AI coding agents need a structured way to understand what to build next. backlogmd gives them (and you) a simple, human-readable backlog that lives in your codebase.

## How it works

Drop a `.backlogmd/` folder into any repo:

```
.backlogmd/
├── PROTOCOL.md              # Rules and formats
├── backlog.md               # Feature roadmap
└── sprints/
    └── <sprint-slug>/
        ├── index.md          # Sprint overview + task table
        ├── 001-task-slug.md
        └── ...
```

- **Features** live in `backlog.md`, ordered by priority
- **Sprints** group tasks into deliverable chunks
- **Tasks** are individual markdown files with status, owner, and acceptance criteria

Agents read `PROTOCOL.md` to understand the format, then pick up tasks and move them through: `todo` → `in-progress` → `ready-to-review` → `ready-to-test` → `done`.

## Visualizer

Browse and explore your backlog at [backlogmd.com](https://www.backlogmd.com).

## Quick start

1. Create a `.backlogmd/` folder in your repo
2. Add a `PROTOCOL.md` (see [the spec](.backlogmd/PROTOCOL.md))
3. Add features to `backlog.md`
4. Create sprints and tasks
5. Point your agents at `PROTOCOL.md` and let them work

## License

MIT
