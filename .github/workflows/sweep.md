---
name: Sweep rollouts
description: >-
  Advance every open rollout: record child PR results, update contracts and memory,
  release the next layer, and close out tracking files. Opens one [sweep] PR when anything changed.
on:
  schedule: every 2h
  workflow_dispatch:
permissions:
  contents: read
  pull-requests: read
  actions: read
engine:
  id: gemini
  model: gemini-2.5-flash   # Pro models have no free-tier quota
  version: "0.43.0"   # gemini-cli >=0.55 fails auth behind the gh-aw API proxy (github/gh-aw#58445)
timeout-minutes: 30
tools:
  edit:
  bash: true
  github:
    toolsets: [repos, pull_requests, actions]
safe-outputs:
  threat-detection:
    engine: false   # gemini cannot run detection; keep the non-AI checks only
  create-pull-request:
    title-prefix: "[sweep] "
    labels: [sweep]
    draft: false
    preserve-branch-name: true
    allowed-branches: ["sweep/*"]
    allowed-files:
      - "runs/**"
      - "contracts/**"
      - "harness/memory/**"
      - "specs/QUEUE.md"
      - "specs/HISTORY.md"
      - "specs/UNRESOLVED.md"
---

# Sweep rollouts

You are the supervisor of the River project. Read `stack.yaml` and `contracts/README.md` first.

## Find work

Open rollouts are `runs/*/rollout.md` files with any row not `done`. For each manifest
`runs/<TASK>/<repo>/manifest.yaml` with status `ready` or `dispatched`, find the pull request in
the manifest's `repo` whose head branch equals the manifest's `branch` (any state).

## Update each manifest

- No PR yet: leave it.
- PR open: set status `dispatched`, put the PR link and its check state in `rollout.md`.
- PR closed without merge: set status `blocked`, note why in `rollout.md` (read the last comments).
- PR merged:
  1. Write `runs/<TASK>/<repo>/result.md` from `harness/templates/pack/result.md`.
  2. Update `contracts/versions.yaml` for every name in the manifest `produces`:
     `river_config_tag` = newest tag of river-common; `openapi` = copy river-query's `openapi.json`
     to `contracts/openapi.json` when the PR reports it changed; others from the PR's `## Outputs` block.
  3. Append each line of the PR's `## Notes for memory` (unless "none") to `harness/memory/<repo>.md`
     as `- [<TASK>] <lesson>`.
  4. Add any `## Open questions` to `specs/UNRESOLVED.md` under Open as `- [<TASK>] <question>`.
  5. Set status `done`.

## Release the next layer

For each `waiting` manifest in the same task: when no affected repo that produces one of its `needs`
is still unfinished, fill `needs` from `contracts/versions.yaml`, copy the updated contract files into
its pack `contracts/` folder, and set status `ready`. Merging the sweep PR dispatches it.

## Close out

When every row of a rollout is `done`: strike the task in `specs/QUEUE.md` (`~~RIVER-N: Title~~`) and
append to `specs/HISTORY.md`:

```
### <date> — RIVER-N: implementation done
<1-2 sentences: what shipped in which repos>
```

## Output

If anything changed, open one pull request with `create_pull_request`, branch `sweep/<date>-<time>`,
summarising each task's transitions. If nothing changed, call `noop`.
