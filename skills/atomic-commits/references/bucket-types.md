# Bucket Types Reference

## Infrastructure (🔴 High Priority)

Files that others depend on. Commit first.

| File Types | Examples |
|------------|----------|
| Settings | `settings.py`, `.env.example`, `config.py` |
| Constants | `constants.py`, `enums.py` |
| Base classes | `base.py`, `models.py`, `schemas.py` |
| Dependencies | `pyproject.toml`, `package.json` |

## Feature (🟡 Medium Priority)

New capabilities or significant changes.

| File Types | Examples |
|------------|----------|
| New modules | `tasks/*.py`, `routes/*.py`, `services/*.py` |
| New components | `components/*.svelte`, `pages/*.tsx` |
| New endpoints | `api/v1/*.py` |

## Refactor (🟡 Medium Priority)

Code restructure without behavior change.

| Patterns | Examples |
|----------|----------|
| Move to subpackage | `billing.py` → `billing/__init__.py`, `billing/client.py` |
| Rename | `UserService` → `UserManager` |
| Extract | Pull common logic into utilities |

## Integration (🟡 Medium Priority)

Wiring code together.

| File Types | Examples |
|------------|----------|
| Routes | `__init__.py` with imports |
| Glue | Factory functions, dependency injection |
| Wiring | `app.py`, `main.py` |

## Tests (🟢 Low Priority)

Test files and fixtures.

| File Types | Examples |
|------------|----------|
| Test files | `tests/test_*.py`, `*.test.ts` |
| Fixtures | `tests/conftest.py`, `__fixtures__/` |
| Mocks | `tests/mocks/` |

## Build (🔵 Auto)

Generated files. Commit separately.

| File Types | Examples |
|------------|----------|
| SDK | `*.gen.ts`, `client.gen.ts` |
| Locks | `package-lock.json`, `bun.lock` |
| Type definitions | `types.gen.ts` |

## Style (🟢 Low Priority)

Formatting only. Commit last or combine with related change.

| Patterns | Examples |
|----------|----------|
| Import reorder | `isort`, alphabetize |
| Whitespace | Trailing commas, line breaks |
| Class order | Methods reordered |

## Cross-Cutting Changes

Some files span multiple buckets:

### Example: Billing Restructure

```
backend/src/infra/billing.py        → Refactor (move code)
backend/src/infra/billing/__init__.py → Infrastructure (new package)
backend/src/infra/billing/client.py  → Feature (new module)
backend/src/infra/billing/models.py  → Feature (new models)
tests/test_infra_billing.py         → Tests (update)
```

**Order:** Infrastructure → Feature → Tests

### Example: New Endpoint

```
backend/src/api/v1/webhook.py        → Feature (new route)
backend/src/api/services/webhook.py   → Feature (new service)
backend/src/api/v1/__init__.py      → Integration (register route)
tests/test_webhook.py                → Tests
```

**Order:** Feature → Integration → Tests
