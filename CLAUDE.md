# CLAUDE.md

Instructions for Claude Code when working in this repository.

This is the supervisor repo (V2, see `docs/v2.md`). It contains no product code. Code lives in the child repos listed in `stack.yaml`; they receive context only through packs in `runs/`.

## Key files

| File | Purpose |
|------|---------|
| `specs/SPEC.md` | Project context — why, architecture, tech stack, decisions. Read first. Read-only during spec authoring. |
| `specs/QUEUE.md` | Work queue — one spec in progress at a time. |
| `specs/HISTORY.md` | Changelog — what changed across specs and when. Append at bottom. |
| `specs/UNRESOLVED.md` | Open questions — unresolved issues across all specs. |
| `.claude/commands/spec.md` | Skill — write and update specs. Opens a PR with spec only; implementation follows after merge. |
| `.claude/commands/create-issue.md` | Skill — create a GitHub issue with category and priority labels, then check out the GHA-created spec branch. |
| `.claude/commands/create-feature.md` | Skill — plan an ambitious feature: decompose into phases, document risks, save to `specs/plans/`. |
| `.claude/commands/plan-spec.md` | Skill — execute a saved plan: spawn one parallel subagent per phase to create issues and write specs. |
| `stack.yaml` | Child repos, layers, skills, checks, needs/produces. The only file that knows every repo. |
| `contracts/` | Interfaces between child repos (`versions.yaml`, `openapi.json`). Updated only by `sweep.md`. |
| `harness/` | Skills, per-repo memory, pack templates, tools shipped in packs. `.claude/skills` links here. |
| `runs/` | Context packs per task and repo, plus `rollout.md` status. Written by `pack.md` and `sweep.md`. |
| `.github/workflows/spec-from-issue.yml` | GHA — fires on issue open (spec draft branch + PR). |
| `.github/workflows/pack.md` | gh-aw — on spec PR merge, compiles packs and opens a `[pack]` PR. |
| `.github/workflows/dispatch.yml` | GHA — when a manifest becomes `ready` on main, starts the child's `implement` workflow. |
| `.github/workflows/sweep.md` | gh-aw — every 2h, records child PR results, updates contracts/memory/queue, releases the next layer. |

## Choosing the right command

When a user describes something they want to build or fix, proactively suggest the right starting point before asking for further detail.

| Situation | Command to propose |
|-----------|--------------------|
| Small, well-scoped task (bug, doc, single endpoint, config change) | `/create-issue` — create an issue and jump straight to the spec branch |
| Feature with multiple moving parts or unclear scope | `/create-feature` — plan phases first, then `/plan-spec` to kick off specs in parallel |
| Spec branch already exists and is checked out | `/spec` — write the spec now |

Implementation no longer happens in this repo: merging a spec PR triggers `pack.md`. To re-pack a task manually, run the `Compile context packs` workflow with its task id.

If the user describes a task but does not specify a command, make a recommendation in one sentence ("This sounds like a `/create-issue` — want me to kick that off?") and wait for confirmation. Do not silently start a workflow.

Use `/create-feature` when:
- The task spans more than one component or codebase area
- The user uses words like "system", "pipeline", "full", "end-to-end", "redesign"
- The scope is unclear and decomposing it first would surface risks or open questions

Use `/create-issue` when:
- The task is contained in a single component
- Acceptance criteria can be stated in a few lines
- No design decisions need to be resolved before implementation begins

## Running Commands

Always prefix with `mise exec --` so the correct toolchain is used (`mise exec -- cargo build -p river-index`). Never call `cargo` directly.

After editing any `.github/workflows/*.md`, recompile with `gh aw compile` and commit the `.lock.yml` with it.

## Style

- No emojis in any output, files, or code.
