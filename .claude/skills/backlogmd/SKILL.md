---
name: backlogmd
description: Define a new feature, break it into tasks, and add it to the backlog.
argument-hint: <feature description>
allowed-tools: Read, Write, Edit, Glob, Bash(mkdir *)
---

# Backlog Feature Creator

You are an agent that helps the user define a new feature, break it into tasks, and write all the necessary files in the `.backlogmd/` system.

## Step 1: Read the protocol and current backlog

- Read `.backlogmd/PROTOCOL.md` to understand all file formats, naming conventions, and rules.
- Read `.backlogmd/backlog.md` to determine the next available feature priority number (zero-padded to three digits, e.g. `002`).

## Step 2: Propose the feature and tasks

Based on `$ARGUMENTS`, propose:

1. **Feature name** — short, descriptive title
2. **One-line description** — what this feature delivers
3. **Tasks** — break the feature into concrete implementation tasks. For each task propose:
   - Task name
   - Short description (2-3 sentences)
   - Acceptance criteria (as checkbox items)

Present the full proposal to the user in a readable format and **ask for confirmation or edits** before proceeding. Do not write any files yet.

## Step 3: Sprint placement

After the user confirms the feature and tasks:

1. Scan `.backlogmd/sprints/` for existing sprint folders.
2. Read each sprint's `index.md` and check its **Status** field.
3. Collect all sprints with status `open`.

Then:

- **If open sprints exist:** List them and ask the user which sprint to add the tasks to, or whether to create a new sprint.
- **If no open sprints exist:** Ask the user for a new sprint name and a one-line sprint goal.

## Step 4: Write all files

Once the user has confirmed everything:

### 4a. Append feature to `backlog.md`

Add a new feature entry at the end of the `## Features` section following this exact format:

```
### <NNN> - <Feature Name>
- **Status:** todo
- **Sprint:** [<sprint name>](sprints/<sprint-slug>/index.md)
- **Description:** <one-line summary>
```

### 4b. Create sprint folder (if new sprint)

If the user chose to create a new sprint:

1. Create the directory `.backlogmd/sprints/<sprint-slug>/`
2. Create `.backlogmd/sprints/<sprint-slug>/index.md` with this format:

```
# Sprint: <Sprint Name>

- **Status:** open
- **Goal:** <one-line goal>

## Tasks

| # | Task | Status | Owner |
|---|------|--------|-------|
```

### 4c. Create task files

For each task, create `.backlogmd/sprints/<sprint-slug>/<NNN>-<task-slug>.md`:

```
# <Task Name>

- **Status:** todo
- **Priority:** <NNN>
- **Owner:** —
- **Feature:** [<Feature Name>](../../backlog.md#NNN---feature-name-slug)

## Description

<detailed description>

## Acceptance Criteria

- [ ] <criterion>
```

- Task numbers are zero-padded to three digits and sequential within the sprint (check existing tasks to find the next number).
- Task slugs are lowercase kebab-case derived from the task name.
- The Feature link anchor must be lowercase, with spaces replaced by `-` and the pattern `NNN---feature-name`.

### 4d. Update sprint task table

Append a row for each new task to the `## Tasks` table in the sprint's `index.md`:

```
| <NNN> | [<Task name>](<NNN>-<task-slug>.md) | todo | — |
```

## Rules

- Follow the formats in `PROTOCOL.md` exactly — no YAML frontmatter, pure markdown.
- All paths are relative within `.backlogmd/`.
- Never overwrite existing features or tasks — only append.
- Always confirm with the user before writing files.
