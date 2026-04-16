---
name: granular-git
description: "Use this skill whenever an agent needs to commit a precise, character-level subset of file changes to a Git repository without disturbing the working directory. Triggers include: staging only part of a file's edits, committing one logical change while leaving other in-progress work untouched on disk, or when patch-based approaches (git apply, git add -p) are failing due to surrounding context drift. Uses Git plumbing commands (hash-object, update-index) to bypass the working directory entirely. Handles tracked files, untracked files that exist on disk, and brand-new files created purely from memory."
---

# Granular Git

Stage and commit exact file content without touching the working directory.

## Core Concept

Standard Git: `Working Directory → Staging Area → Commit`

This skill reverses the flow: the agent constructs the exact desired file state in memory and injects it directly into the Staging Area.

```
[Target State] → hash-object → update-index → commit
```

The physical file on disk is **never modified**.

## Procedure

### 1. Determine Target State

Read current index state if there are existing staged changes to preserve:

```bash
git show :0:path/to/file.txt
```

Apply character-level edits to **that string** (not the disk file).

### 2. Write as Git Object

```bash
TARGET_SHA=$(printf '%s' "$TARGET_CONTENT" | git hash-object -w --stdin)
```

### 3. Map to Index

**For tracked files:**
```bash
git update-index --cacheinfo 100644,"$TARGET_SHA","path/to/file.txt"
```

**For brand-new files:**
```bash
git update-index --add --cacheinfo 100644,"$TARGET_SHA","path/to/file.txt"
```

| Argument | When |
|----------|-------|
| `--add` | **Required** for untracked files |
| `100644` | Regular files |
| `100755` | Executable files |

### 4. Commit

```bash
git commit -m "type(scope): description"
```

## Multi-Commit Pattern

For multiple independent changes in one file:

```
HEAD → Commit 1 (Change A) → Commit 2 (A+B) → Commit 3 (A+B+C)
```

1. Construct target state for Change A only
2. Commit
3. Add Change B to target state, commit
4. Repeat for Change C

## Safety Checklist

- [ ] Is Git initialized? — `git rev-parse --is-inside-work-tree`
- [ ] Existing staged changes? — Read via `git show :0:<file>` and merge
- [ ] Repo in clean state? — No ongoing merge/rebase

## Advantages Over Patches

| Property | Patches | This Skill |
|----------|---------|------------|
| Context sensitivity | Fails if context changed | Immune |
| Precision | Line-level | Character-level |
| Working directory | May modify | Never touches |

## Example

File on disk has Bug fix + Refactor + Docs. You want to commit only the bug fix:

```bash
# 1. Read HEAD version
git show HEAD:src/utils.py

# 2. Create target state (HEAD + bug fix only)
TARGET_SHA=$(printf '%s' "$FIXED_CONTENT" | git hash-object -w --stdin)

# 3. Stage just the bug fix
git update-index --cacheinfo 100644,"$TARGET_SHA","src/utils.py"

# 4. Commit bug fix
git commit -m "fix: correct null pointer check"

# 5. Now repeat for refactor...
```
