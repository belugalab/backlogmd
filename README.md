# backlog.md

A markdown-based backlog system designed for agentic development. Define work items and tasks in plain `.md` files — no databases, no SaaS lock-in, just your repo.

## Why

AI coding agents need a structured way to understand what to build next. backlogmd gives them (and you) a simple, human-readable backlog that lives in your codebase.

## How it works

Drop a `.backlogmd/` folder into any repo:

```
.backlogmd/
├── backlog.md               # Item list (one slug per line)
└── work/
    └── <item-slug>/
        ├── index.md          # Bullet list of task links
        ├── 001-task-slug.md
        └── ...
```

- **Items** are listed in `backlog.md` as linked slugs ordered by priority (e.g. `001-project-foundation`). An optional [Conventional Commits](https://www.conventionalcommits.org/) type can be included (e.g. `001-chore-project-foundation`).
- **Item folders** group tasks into deliverable chunks under `work/`
- **Tasks** are individual markdown files with metadata in HTML comment sections and status tracking

Agents pick up tasks and move them through: `open` → `in-progress` → `done`.

## Visualizer

Browse and explore your backlog at [backlogmd.com](https://www.backlogmd.com).

## Quick start

1. Create a `.backlogmd/` folder in your repo
2. Add items to `backlog.md`
3. Create items and tasks
4. Let your agents work

## License

MIT
