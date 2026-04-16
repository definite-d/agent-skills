# Decision Tree: Split or Merge?

## Questions to Ask

### 1. Can each group compile/test independently?

```
YES → Commit separately
NO  → Commit together
```

### 2. Do changes tell a coherent story?

```
"Add X, fix Y, refactor Z" → Multiple commits
"Implement X feature"        → Single commit
```

### 3. Are changes from different people/features?

```
YES → Separate commits (blame, revert)
NO  → Can combine
```

### 4. Does one change enable another?

```
A enables B → Commit A first, then B
Independent  → Can commit in either order
```

## Decision Matrix

| Scenario | Decision | Reason |
|----------|----------|--------|
| 5 new files for same feature | **Merge** | Same feature, one commit |
| New feature + bug fix | **Split** | Different types |
| Config change + feature | **Split** | Revert separately |
| Import reorder + logic | **Split** or **Merge with "reformat"** | Style vs logic |
| 3 files moved to package | **Merge** | Same refactor |
| Test + implementation | **Merge** or **Split** | Depends on test size |

## Common Patterns

### Pattern: Intermingled Changes

One file has multiple logical changes.

**Solution:** Use granular-git for precise staging
1. Commit logical change A (granular-git)
2. Commit logical change B (granular-git or standard)

### Pattern: Test Depends on Implementation

Tests won't pass without implementation.

**Solution:** Commit together OR commit implementation first
```bash
git add src/impl.py src/test_impl.py
git commit -m "feat: add X with tests"
```

### Pattern: Config Changes

`pyproject.toml` has version bump AND deps.

**Solution:** Version bump first, then deps
```bash
git add pyproject.toml
git commit -m "chore: bump version to 2.0.0"

git add requirements.txt
git commit -m "feat: add new dependency"
```

### Pattern: Generated + Hand-edited

SDK regenerated AND types hand-edited.

**Solution:** Generated first, then hand-edits
```bash
git add src/sdk/
git commit -m "build: regenerate SDK"

git add src/sdk/types.ts
git commit -m "feat: add custom type overrides"
```

## Conflict Resolution Decisions

### Prefer Current When:
- Better error handling
- New features (Task metadata, tracing)
- Updated patterns

### Prefer Incoming When:
- Bug fix we missed
- Better test coverage
- Purely additive

### Split If:
- Conflicting changes are independent
- Both sides have valid points
```bash
# Commit current's billing refactor
git add billing/
git commit -m "refactor: restructure billing"

# Then merge normally
git add -A
git commit -m "merge: main into rework"
```

## When in Doubt

- **Split** → Easier to revert, better blame
- **Merge** → Cleaner history, less noise

Default to splitting when uncertain.
