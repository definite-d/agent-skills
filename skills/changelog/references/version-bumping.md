# SemVer Version Bumping — Decision Tree

Semver 2.0.0: `MAJOR.MINOR.PATCH`. Read the highest-impact change in the release, then bump.

## Decision Tree

```
Any breaking change?
├─ Yes, project >= 1.0.0 ──► bump MAJOR (reset MINOR, PATCH to 0)
├─ Yes, project is 0.x ────► bump MINOR (PATCH resets to 0)
└─ No
   ├─ Any new user-facing feature? ──► bump MINOR (PATCH to 0)
   └─ Only fixes / refactors / docs / chore
      └─ Any previous release? ──► bump PATCH
         └─ No prior release ──► 0.1.0 (or 1.0.0 if project is stable by convention)
```

## What Counts as Breaking

Treat as breaking when:

- Removed or renamed a public API, flag, env var, config key, or route.
- Changed default behavior that silently alters existing output.
- Dropped a supported runtime/dependency version.
- Changed a response schema or error contract.
- Behavior fix that breaks previously "working" (even if buggy) usage.

Reverts of a change that was itself released count as breaking for that release window.

## Special Cases

| Case | Rule |
|------|------|
| No prior release | `0.1.0` baseline; consider `1.0.0` only if project already declares stability |
| Multiple categories | Use the highest-impact bump (one rule wins, not summed) |
| Pre-releases (`1.0.0-rc.1`) | Precedence per semver: `1.0.0-alpha < 1.0.0-alpha.1 < 1.0.0-beta < 1.0.0`; bump pre-release identifier, not MAJOR/MINOR/PATCH, for iteration on the same target version |
| `0.x` series | MINOR effectively acts as MAJOR for compatibility claims; only PATCH is backwards-compatible by default |
| Unreleased-only edits | When only updating `[Unreleased]`, do **not** bump anything — the version is decided at release time |

## Ask the User When

- The repo has no tags and no declared version policy (0.1.0 vs 1.0.0).
- A change is ambiguously breaking (e.g., renaming an internal-only function).
- Monorepo: which package(s) the version applies to.
