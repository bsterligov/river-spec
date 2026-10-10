# Contracts

Interfaces between child repos. A child never reads another child; it only sees the contracts the supervisor copies into its pack.

| Contract | Owner (produces) | Consumers (needs) | Where |
|----------|------------------|-------------------|-------|
| `river_config_tag` | river-common | river-ingest, river-query | git tag on river-common, used as `river-config = { git = ..., tag = ... }` |
| `clickhouse_schema` | river-ingest | river-query, river-infra | latest migration version in `river-ingestion/migrations/clickhouse` |
| `s3_key_schema` | river-ingest | (sidecar and ingestion share it internally) | `{signal}/{service}/{timestamp}-{uuid}.pb` |
| `openapi` | river-query | river-ui | `openapi.json` at the river-query repo root, mirrored to `contracts/openapi.json` |

Current values: `versions.yaml`. Only `sweep.md` changes them after a child PR merges: `river_config_tag` from the newest river-common tag, `openapi` by copying river-query's `openapi.json`, the rest from the PR's `## Outputs` block.

Cross-cutting rules that are not a single value (env var naming, ports, filter DSL) stay in `specs/SPEC.md` under Decisions; `pack.md` copies the relevant lines into each pack's `context.md`.
