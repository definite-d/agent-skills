# Atomic Commits

Transform uncommitted changes into clean, logical commits using iterative analysis and dependency-aware ordering.

## When to Use

- User has 5+ modified files to commit
- Changes span multiple logical features/fixes
- User asks to "break down" or "organize" commits
- Merge conflicts need analysis before resolution

## Quick Start

```bash
# 1. Discover all changes
git status
git diff --stat

# 2. Analyze and bucket files by functionality
# 3. Order by dependency (infra first, integration last)
# 4. Commit each bucket with proper message format
```

## Commit Message Format

```
<type>(<scope>): <description>
```

**Follow existing project patterns first**, then fallback to Conventional Commits:

```bash
git log --oneline -20  # Check existing patterns
```

## Bucket Types

| Priority | Bucket | Files |
|----------|--------|-------|
| 🔴 High | Infrastructure | Settings, constants, base classes |
| 🟡 Medium | Feature/Refactor | New modules, code restructure |
| 🟡 Medium | Integration | Routes, wiring, glue code |
| 🟢 Low | Tests | Test files, fixtures |
| 🔵 Auto | Build | Generated files, lock files |

## Dependencies

- **Granular-git skill** — For precise, character-level commits when one file has multiple logical changes

## See Also

- `references/bucket-types.md` — Detailed bucket categories with examples
- `references/decision-tree.md` — When to split vs merge commits
