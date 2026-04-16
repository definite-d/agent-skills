# granular-git

A skill for performing character-precise, non-destructive Git commits via Git's plumbing layer. Designed for use by AI agents or automated pipelines that need to commit exact subsets of file changes without touching the working directory.

---

## The Problem

Standard Git tooling (`git add`, `git add -p`, `git apply`) is built for human workflows. For agents, this creates three failure modes:

- **Context fragility** — patch application breaks if surrounding lines have changed.
- **Line-level granularity ceiling** — interactive staging tops out at hunks, not characters.
- **Working directory side effects** — staging operations can silently alter the files a developer is actively editing.

## The Solution

`granular-git` uses the **Index-Overwrite Method**: the agent constructs the exact desired file state as a string, injects it into Git's object database with `git hash-object`, maps it to a file path with `git update-index`, and commits — all without reading or writing the physical file on disk.

## Contents

| File       | Description                                       |
| ---------- | ------------------------------------------------- |
| `SKILL.md` | Step-by-step procedure, safety checklist, examples |

## Quick Reference

```bash
# 1. (Optional) Read the current staged state before overwriting
git show :0:path/to/file.txt

# 2. Write the target content into Git's object store
TARGET_SHA=$(printf '%s' "$TARGET_CONTENT" | git hash-object -w --stdin)

# 3. Point the index at the new blob
git update-index --cacheinfo "100644,$TARGET_SHA,path/to/file.txt"

# 4. Commit
git commit -m "Precise, scoped change message"
```

## When to Use

- Committing one logical change out of many interleaved edits in a file.
- Patch-based methods are failing due to context drift.
- The working directory must remain untouched throughout the commit process.

## When _Not_ to Use

- Simple, whole-file commits — `git add` is fine and less overhead.
- Repositories not initialized with Git.
- Situations requiring interactive human review before committing — use `git add -p` instead.
