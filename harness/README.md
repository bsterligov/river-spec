# Harness

Everything an agent needs that is not code. Child repos contain only code; `pack.md` copies the relevant parts of this folder into a task's context pack.

| Path | What |
|------|------|
| `skills/<name>/SKILL.md` | Coding rules, selected per repo by `stack.yaml` `skills:` |
| `memory/<repo>.md` | Durable learnings per child repo (pitfalls, conventions, past decisions). Appended by `sweep.md` |
| `templates/pack/` | Shape of a context pack (`manifest.yaml`, `task.md`, `context.md`, `result.md`) |
| `tools/` | Scripts shipped inside packs (e.g. `check-duplication`) |
| `legacy-v1/commands/` | v1 monorepo implementation commands (`/spec-dev`, `/plan-dev`, `/pr-merge`), kept for reference |

`.claude/skills` is a symlink to `skills/`, so interactive Claude Code sessions in this repo use the same rules.
