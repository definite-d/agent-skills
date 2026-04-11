---
name: granular-git
description: "Use this skill whenever an agent needs to commit a precise, character-level subset of file changes to a Git repository without disturbing the working directory. Triggers include: staging only part of a file's edits, committing one logical change while leaving other in-progress work untouched on disk, or when patch-based approaches (git apply, git add -p) are failing due to surrounding context drift. Uses Git plumbing commands (hash-object, update-index) to bypass the working directory entirely. Handles tracked files, untracked files that exist on disk, and brand-new files created purely from memory."
---

# Skill: `granular-git`

## When to Trigger

- The task requires staging a **specific version of a file** that differs from both HEAD and the current disk state.
- The agent needs **character- or token-precise commits** (e.g., changing only one word, one line, or one expression).
- Multiple **distinct semantic changes** exist within a single file and deserve separate commits (e.g., a bug fix + a refactor + a docs update).
- Patch-based approaches (`git add -p`, `git apply`) are too fragile due to surrounding context having changed.
- The commit must be **non-destructive** to any unsaved or in-progress work in the working directory.

---

## Core Concept: The Index-Overwrite Method

Standard Git assumes: `Working Directory → Staging Area → Commit`

This skill reverses the flow: the agent **constructs the exact desired file state in memory** and injects it directly into the Staging Area (Index), bypassing the working directory entirely.

```
[Target State String] → hash-object → update-index → commit
```

The physical file on disk is **never modified**.

---

## Step-by-Step Procedure

### Step 1: Determine the Target State

Before touching Git, the agent must produce a string representing the **exact desired content** of the file for this commit.

If there are already staged changes to preserve, read the current index state first:

```bash
git show :0:path/to/file.txt
```

Apply character-level edits to **that string** (not the disk file). This ensures no previously staged changes are lost.

**Example:**
| Version | Content |
| ------- | ------- |
| HEAD (last commit) | `The quick brown fox jumps.` |
| Disk (working copy) | `The fast orange cat sleeps.` |
| Target (this commit) | `The fast brown fox jumps.` ← only "fast" is being staged |

---

### Step 2: Write the Target State as a Git Object

Use `git hash-object` to write the target string into Git's object database. This produces a Blob ID (SHA hash).

```bash
TARGET_SHA=$(printf '%s' "$TARGET_CONTENT" | git hash-object -w --stdin)
```

- `-w` writes the object to `.git/objects` and returns its SHA-1 hash.
- `printf '%s'` is used for binary safety and exact preservation of newlines.

---

### Step 3: Map the Blob to a Filename in the Index

Tell Git to associate the new Blob with the target file path in the Staging Area:

```bash
git update-index --add --cacheinfo 100644 "$TARGET_SHA" "path/to/file.txt"
```

| Argument             | Meaning                                                                                  |
| -------------------- | ---------------------------------------------------------------------------------------- |
| `--add`              | Required for brand-new files that have never existed on disk (harmless on tracked files) |
| `100644`             | Standard permission mode for a regular (non-executable) file                             |
| `$TARGET_SHA`        | The Blob hash from Step 2                                                                |
| `"path/to/file.txt"` | The repo-relative path of the file                                                       |

For executable files, use `100755`. For symlinks, use `120000`.

---

### Step 4: Commit

The Index now holds the precise target state. Commit it normally, respecting the existing commit conventions:

```bash
git commit -m "Your precise commit message"
```

The working directory file is unchanged. The developer's in-progress work is untouched.

---

### Step 5: Multi-Commit Semantic Grouping (Optional)

When a file contains **multiple independent logical changes**, repeat Steps 1-4 for each distinct change, building progressively:

**Pattern:**

```
HEAD → Commit 1 (Change A) → Commit 2 (Change A+B) → Commit 3 (Change A+B+C)
```

**Procedure:**

1. Identify the distinct semantic changes (e.g., bug fix, refactor, docs update)
2. For each change, construct the **cumulative target state** including all prior changes
3. Execute Steps 2-4 (hash-object → update-index → commit) for each cumulative state
4. Each commit should have a focused message describing that specific change

**Example:**
| Commit | Target Content | Message |
|--------|----------------|---------|
| 1 | Original + Bug fix only | `fix: correct null pointer check` |
| 2 | Commit 1 + Refactor | `refactor: extract validation logic` |
| 3 | Commit 2 + Docs | `docs: add usage examples` |

This produces a clean, reviewable history where each commit represents one logical unit of work, even when all changes coexist in a single file on disk.

---

## Full Reference Pipeline

| Stage          | Command                              | Purpose                                           |
| -------------- | ------------------------------------ | ------------------------------------------------- |
| **Read index** | `git show :0:path/to/file.txt`       | Retrieve current staged state before overwriting  |
| **Transform**  | String manipulation in agent logic   | Produce the character-perfect target file content |
| **Objectify**  | `git hash-object -w`                 | Persist target content as a Git Blob, get its SHA |
| **Index**      | `git update-index --add --cacheinfo` | Map the Blob to a file path in the Staging Area   |
| **Record**     | `git commit -m "..."`                | Persist the staged state into permanent history   |

---

## Key Advantages Over Patch-Based Methods

| Property                 | Patch / `git apply`                | Index-Overwrite (this skill)          |
| ------------------------ | ---------------------------------- | ------------------------------------- |
| Context sensitivity      | Fails if surrounding lines changed | Immune — only cares about final state |
| Precision                | Line-level at best                 | Character-level                       |
| Working directory safety | May modify disk files              | Never touches disk files              |
| Agent reliability        | Fragile                            | Robust                                |

---

## Safety Checklist

Before executing, the agent should verify:

1. **Is Git initialized?** — `git rev-parse --is-inside-work-tree`
2. **Are there existing staged changes?** — `git diff --cached --name-only`
   - If yes, read and incorporate them via `git show :0:<file>` before overwriting.
3. **Is the file tracked?** — `git ls-files --error-unmatch path/to/file.txt` (no longer required for new files; `--add` handles them automatically)
4. **Is the repo in a clean, non-rebasing state?** — `git status --porcelain`

---

## Example: Agent Pseudocode

```python
import subprocess

def granular_commit(repo_path, file_path, target_content, commit_message):
    # Step 1: Check for existing staged state (optional merge logic)
    result = subprocess.run(
        ["git", "show", f":0:{file_path}"],
        cwd=repo_path, capture_output=True, text=True
    )
    # (Merge with target_content if needed)

    # Step 2: Write target content as a Git Blob
    hash_result = subprocess.run(
        ["git", "hash-object", "-w", "--stdin"],
        input=target_content, cwd=repo_path,
        capture_output=True, text=True
    )
    blob_sha = hash_result.stdout.strip()

    # Step 3: Update the index (now fool-proof for new files)
    subprocess.run(
        ["git", "update-index", "--add", "--cacheinfo", f"100644,{blob_sha},{file_path}"],
        cwd=repo_path, check=True
    )

    # Step 4: Commit
    subprocess.run(
        ["git", "commit", "-m", commit_message],
        cwd=repo_path, check=True
    )
```

> **Note:** `--cacheinfo` accepts either `mode SHA path` (three args) or `mode,SHA,path` (one comma-separated arg). Prefer the comma form to avoid shell splitting issues in scripts. The `--add` flag is now always included for universal reliability.
