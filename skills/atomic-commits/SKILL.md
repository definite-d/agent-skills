---
name: atomic-commits
description: |-
  Create atomic, logically-grouped git commits from uncommitted changes. Analyzes files,
  buckets by functionality, and commits in dependency order. Use when user has many changes
  to commit, wants to "break down" changes, or asks for commit strategy.

  Examples:
  - user: "commit all these changes" → analyze, group, and commit atomically
  - user: "break these 50 files into commits" → bucket by functionality, stage, commit
---

# Atomic Commits Skill

Transform uncommitted changes into clean, logical commits using iterative analysis.

## Workflow

### 1. Discover
```bash
git status
git diff --stat
git diff --name-only
```

### 2. Bucket by Functionality

Group files into logical buckets:

| Bucket | Use For | Priority |
|--------|---------|----------|
| **Infrastructure** | Settings, constants, base classes | 🔴 High |
| **Feature/Refactor** | New capability, code restructure | 🟡 Medium |
| **Integration** | Routes, wiring, glue code | 🟡 Medium |
| **Tests** | Test files | 🟢 Low |
| **Build** | Generated files, lock files | 🔵 Auto |

### 3. Order by Dependency

1. Infrastructure first (others depend on it)
2. Feature/Refactor (core logic)
3. Integration last (wiring it together)

### 4. Commit Each Bucket

**For cleanly separated files:**
```bash
git add path/to/bucket/files
git commit -m "type(scope): description"
```

**For mixed changes in one file:** Use granular-git skill.

## Commit Messages

### Follow Existing Patterns First

Check `git log --oneline -20` for established conventions:
- Scope names (e.g., `backend`, `frontend`, `auth`)
- Type usage (e.g., `feat`, `fix`, `refactor`)
- Message style

### Fallback: Conventional Commits

```
<type>(<scope>): <description>
```

| Type | When |
|------|------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code restructure |
| `chore` | Build, deps, tooling |
| `test` | Tests only |
| `style` | Formatting only |
| `build` | Generated files |

### Description Rules

1. **Imperative mood**: "add" not "added"
2. **50 chars max**
3. **Lowercase** after scope
4. **No trailing period**

## Decision Checklist

Before each commit:
- [ ] Changes are logically related
- [ ] No unintended files
- [ ] Dependencies satisfied
- [ ] Message follows existing pattern

## Merge Conflicts

When resolving, prefer the implementation that:
1. Has better error handling
2. Includes new features (Task, tracing)
3. Follows updated patterns

## See Also

- `references/bucket-types.md` - Detailed bucket categories
- `references/decision-tree.md` - When to split/merge commits
- Granular-git skill - For precise file-level commits
