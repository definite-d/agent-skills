# Changelog

Write release-ready `CHANGELOG.md` files that follow [Keep a Changelog](https://keepachangelog.com) and [Semantic Versioning](https://semver.org) — from scratch or from existing records like git history and prior releases.

## When to Use

- User asks to generate, regenerate, or update a changelog.
- User wants release notes for a version or a range of versions.
- User provides commit lists or release records to organize into a changelog.

## Quick Start

```bash
# From scratch: check for tags, then log since the last tag
git tag --sort=-version:refname
git log --oneline --decorate

# Update: only the unreleased window matters
git log <last_tag>..HEAD --oneline --decorate
```

## What It Produces

- `## [Unreleased]` section plus newest-first released versions.
- Canonical sections: **Added, Changed, Deprecated, Removed, Fixed, Security** (empty ones omitted).
- SemVer-derived next version (breaking → major, feature → minor, fix → patch).
- ISO dates and GitHub compare-link footer, matching the repo's tag style.
- Repo conventions preserved when updating an existing file.

## Example

```markdown
## [Unreleased]

### Added

- `rate_limit` middleware for public endpoints.

## [1.0.0] - 2026-07-31

### Fixed

- Panic on empty payload in the webhook handler.

[Unreleased]: https://github.com/OWNER/REPO/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/OWNER/REPO/releases/tag/v1.0.0
```

## See Also

- `references/keep-a-changelog.md` — Full format spec, section definitions, diff-link patterns
- `references/version-bumping.md` — SemVer decision tree, breaking-change classification, 0.x rules
