# backlog.md

> **[Spec v4.0.4](SPEC.md)** — the full specification for the `.backlogmd/` system.

A simple markdown-based backlog system for agentic development.
Tasks live as plain `.md` files inside your repo. No SaaS. No lock-in. No hidden state.

It also works as a transparency layer for your coding agents. You can clearly track what they are working on and review their changes through Git.

If you do not want the backlog files tracked, just add them to `.gitignore` and keep your repository tidy.

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

```sh
npx @backlogmd/serve --dir your-project-repo/.backlogmd
```

More information in [backlog.md utils](https://github.com/backlogmd/backlogmd-utils) repository.

![Dashboard](docs/dashboard.png)

## Quick start

1. Create a `.backlogmd/` folder in your repo
2. Add items to `backlog.md`
3. Create items and tasks
4. Let your agents work

## License

MIT
