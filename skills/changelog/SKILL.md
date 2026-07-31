---
name: changelog
description: |-
  Write well-formatted changelogs following Keep a Changelog and SemVer, from scratch or from
  existing records (git history, tags, release notes, an existing CHANGELOG.md). Buckets changes
  into Added/Changed/Deprecated/Removed/Fixed/Security, derives the next version with a semver
  bump, and preserves the file's existing conventions when updating. Use proactively when
  generating, regenerating, or maintaining a CHANGELOG.md, or drafting release notes.

  Examples:
  - user: "Write a changelog for this repo" → generate CHANGELOG.md from git log and tags
  - user: "Update the changelog for the next release" → add new entries under [Unreleased] or a new version heading
  - user: "Regenerate CHANGELOG.md from git history" → rebuild the file from commits, tags, and dates
  - user: "What changed between v1.2.0 and v1.3.0?" → extract and format that release window
  - user: "Draft release notes for 2.0.0" → derive version, sections, and notes from recent commits
---

# Changelog Skill

Produce human-readable, release-ready changelogs that follow [Keep a Changelog](https://keepachangelog.com) structure and [Semantic Versioning](https://semver.org) bump rules, whether starting from an empty repo or maintaining an existing file.

## Mode Selection

| Mode | Trigger | Action |
|------|---------|--------|
| **From scratch** | No `CHANGELOG.md` exists | Build full file from git history / records |
| **Update existing** | `CHANGELOG.md` exists | Merge new changes, preserving its conventions |
| **Subset** | User names a version range | Extract only that window |

Confirm the mode and target (whole repo, a package, a file) before writing.

## Gather Records

Prioritize sources in this order:

1. **Existing changelog** — parse headings, section names, date format, link style. Preserve them.
2. **Git history** — scope per mode:
   ```bash
   git tag --sort=-version:refname              # release anchors
   git log <last_tag>..HEAD --oneline --decorate # unreleased changes
   git log --oneline --decorate                 # full history (from scratch)
   git log -1 --format='%ad %h' --date=short <tag>  # release date for a tag
   ```
3. **Release records** — release notes, PR descriptions, issues closed in a milestone. Treat as ground truth when present; use commits to fill gaps.
4. **User-provided lists** — commit messages pasted in chat. Parse them as-is.

Skip merge commits, dependabot noise, and generated files unless they affect users.

## Bucket Into Standard Sections

Keep a Changelog categories, ordered as shown:

| Section | Conventional type mapping | Example entry |
|---------|---------------------------|---------------|
| **Added** | `feat`, `perf` (new) | `- **Added:** `rate_limit` middleware for public endpoints.` |
| **Changed** | `refactor`, `perf`, `style` | `- **Changed:** JSON serialization to use `orjson`.` |
| **Fixed** | `fix` | `- **Fixed:** panic on empty payload in the webhook handler.` |
| **Removed** | `remove`, `revert`, `deprecate` follow-up | `- **Removed:** legacy `v1` auth endpoints.` |
| **Deprecated** | `deprecate` | `- **Deprecated:** `--env` flag; use `--profile` instead.` |
| **Security** | `fix` (CVE), `security` | `- **Security:** patched SSRF in proxy URL validation.` |

Rules:

- Write **user-facing summaries**, not raw commit messages. Imperative or past tense is fine, but stay consistent within one file.
- One bullet per logical change; fold related commits (same feature, multiple parts) into a single entry.
- Reference issue/PR numbers (`[#42](https://.../pull/42)`) only if the project already does so.
- **Omit empty sections**; keep the headings exactly `### Added`, `### Changed`, etc.
- Mark breaking changes inside the relevant section with a `**BREAKING:**` prefix and call them out in the release summary.

## Determine the Next Version

Read `references/version-bumping.md` for the full decision tree.

Quick rules (from the last released tag):

- Breaking change → bump **major** (or minor when `0.x`)
- New user-facing feature → bump **minor** (or patch when `0.x`)
- Fixes/refactors/docs → bump **patch**
- No prior release → start at `0.1.0` (or `1.0.0` if the project is stable by its own convention)

## Write the File

### From scratch — Keep a Changelog skeleton

```markdown
# Changelog

All notable changes to this project will be documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- ...

## [1.0.0] - 2026-07-31

### Fixed

- ...

[Unreleased]: https://github.com/OWNER/REPO/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/OWNER/REPO/compare/v0.1.0...v1.0.0
```

Format rules:

- `## [Unreleased]` always first; released versions newest-first.
- Version headings: `## [X.Y.Z] - YYYY-MM-DD` (ISO date, `-` separators).
- Resolve `OWNER/REPO` from `git remote get-url origin`; omit the diff-link footer if no remote exists.
- First release link points at the tag itself, not a comparison.

### Updating an existing file

- Reuse its exact heading depth, date format, link style, and intro line.
- Never rewrite released sections. Only maintain `[Unreleased]` or append a new version heading.
- Move the previous `[Unreleased]` content into the new release heading on release, then start a fresh `[Unreleased]`.
- Deduplicate: if an entry already appears under a released version, do not copy it forward.

## Verify

- [ ] Version numbers in headings match the footer links exactly (with/without `v` prefix consistent).
- [ ] Dates are `YYYY-MM-DD`.
- [ ] Sections present in canonical order, empty ones omitted.
- [ ] Newest release on top, `[Unreleased]` above all releases.
- [ ] No duplicate entries across sections or versions.
- [ ] Semver bump matches content (breaking → major, feature → minor, fix → patch).
- [ ] Repo-specific conventions preserved (scope, link format, trailing punctuation).

## See Also

- `references/keep-a-changelog.md` — Full format spec and edge cases
- `references/version-bumping.md` — SemVer decision tree with 0.x and prerelease rules
