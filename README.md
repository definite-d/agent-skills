# Agent Skills

My personal collection of specialized skills for AI agents, organized as discrete, reusable capabilities.

```bash
npx skills add definite-d/agent-skills
```

## Available Skills

| Skill                                    | Description                                                                                                                         |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| [`granular-git`](./skills/granular-git/) | Character-precise, non-destructive Git commits via Git's plumbing layer. Bypasses the working directory to stage exact file states. |

## Structure

Adapted from Claude's "Anatomy of a Skill". Each skill resides in its own subdirectory:

```
skills/
└── <skill-name>/
    ├── README.md     # Human-readable documentation and examples
    ├── SKILL.md (required)
    │   ├── YAML frontmatter (name, description required)
    │   └── Markdown instructions
    └── Bundled Resources (optional)
        ├── scripts/    - Executable code for deterministic/repetitive tasks
        ├── references/ - Docs loaded into context as needed
        └── assets/     - Files used in output (templates, icons, fonts)
```

- **`SKILL.md`** — Contains the skill's trigger conditions, procedures, and command references. Parsed by agents to determine when and how to use the skill.
- **`README.md`** — Explains the problem the skill solves, the solution approach, and quick-start examples.

## Adding a New Skill

1. Create a subdirectory named after the skill (use kebab-case).
2. Add `SKILL.md` with proper YAML frontmatter (`name` and `description`).
3. Add `README.md` with context and examples.
4. Update the table above in this file in alphabetical order.
