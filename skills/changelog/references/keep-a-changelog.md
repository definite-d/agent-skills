# Keep a Changelog — Format Reference

Format 1.1.0. A changelog is a curated, per-release log of **notable** changes for humans.

## File Skeleton

```markdown
# Changelog

(Optional intro paragraph describing the project's changelog policy.)

## [Unreleased]

### Added
### Changed
### Deprecated
### Removed
### Fixed
### Security

## [1.2.0] - 2026-05-01

### Added

- ...

[Unreleased]: https://github.com/OWNER/REPO/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/OWNER/REPO/compare/v1.1.0...v1.2.0
```

## Section Definitions

| Section | Meaning |
|---------|---------|
| `### Added` | New features. |
| `### Changed` | Changes to existing functionality. |
| `### Deprecated` | Soon-to-be-removed features. |
| `### Removed` | Now-removed features. |
| `### Fixed` | Bug fixes. |
| `### Security` | Vulnerabilities, CVEs, hardening. |

Use only these six. Anything else (Tests, Docs, Chore, Style) does not belong in the canonical list — fold docs/test notes into a relevant section or omit them.

## Rules

1. `## [Unreleased]` at the top; released versions below, **newest first**.
2. Version heading format: `## [X.Y.Z] - YYYY-MM-DD`. Dates are ISO, hyphen-separated.
3. Bullets are short, user-facing summaries. One bullet per logical change.
4. Omit empty sections entirely — never ship `### Deprecated` with nothing under it.
5. Hyperlink version numbers and diff links in the footer. `[X.Y.Z]:` labels must match headings byte-for-byte.
6. First release: link to the tag itself (no comparison exists yet).
7. Consistency within a project: imperative vs. past tense, with/without trailing period, scopes — pick one style and keep it.

## Diff-Link Patterns

| Case | Pattern |
|------|---------|
| Latest release → HEAD | `[Unreleased]: .../compare/v1.2.0...HEAD` |
| Between releases | `[1.2.0]: .../compare/v1.1.0...v1.2.0` |
| First release | `[0.1.0]: .../releases/tag/v0.1.0` |

Match the `v` prefix to how the project tags (check `git tag`). GitHub `compare` URLs work with or without the `v`.

## Common Mistakes

- Duplicating entries that were already released.
- Using raw commit messages (`fix: bump orjson dep`) as entries.
- Including every commit instead of notable changes.
- Forgetting to move `[Unreleased]` content into the release heading on release.
- Mismatched footer labels vs. headings (typos break navigation links).
