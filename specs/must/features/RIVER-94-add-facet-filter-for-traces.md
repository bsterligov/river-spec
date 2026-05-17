# RIVER-94: Add facet filter for traces
Priority: Must
Test Approach: BDD
Why: Operators filtering traces must type raw filter expressions by hand — the facet panel that exists for logs is missing from the traces page.
<!-- STOP -->

## Problem

The traces page has a search bar but no facet panel. Operators investigating a trace have no way to discover what values exist for `service_name` or `operation_name` without already knowing them. The logs page solves this with a facet panel; traces has none. The existing `FacetPanel` widget is also hardwired to `LogsController` and `getLogsFacets`, so it cannot be dropped into the traces page without refactoring.

## Goal

An operator opens the traces page and sees the same facet panel they know from logs: a collapsible left panel with `service_name` and `operation_name` groups. Tapping a value narrows the trace table and appends the token to the search bar.

**Scenarios**

Given the traces page is open and `/v1/traces/facets` returns facets,
When the panel finishes loading,
Then `service_name` and `operation_name` `ExpansionTile` groups are visible with value/count rows.

Given a facet value row is visible,
When the operator taps it,
Then the search bar appends `field:value` (joined with ` AND ` if a filter was already present) and the trace table re-fetches.

Given the `/v1/traces/facets` request is in flight,
When the panel is rendered,
Then a grey shimmer placeholder is shown.

Given the `/v1/traces/facets` request fails,
When the error is received,
Then the facet panel shows nothing.

Given facets have loaded,
When the operator changes the time range or edits the search bar,
Then the facet panel re-fetches to reflect the new context.

## Scope

**In**
- `GET /v1/traces/facets` — new API endpoint; queries `service_name` and `operation_name` from the `traces` ClickHouse table; same shape as `GET /v1/logs/facets` (`Vec<FacetField>`); LIMIT 20 per field; silent partial response on error
- Regenerate Dart API client to include `getTracesFacets`
- Extract a `FacetController` interface (or equivalent abstraction) from `FacetPanel` so it can work with either `LogsController` or `TracesController` — no logic duplication
- Wire `FacetPanel` into `TracesPage` in the same collapsible left-panel position as in `LogsPage`
- `TracesController.setFilter` and `reload()` already exist — no changes needed there

**Out**
- Multi-select within a facet group
- Facet search within a group
- Additional facet fields beyond `service_name` and `operation_name`
- Any change to the logs facet panel behavior

## Decisions

- Mirror the logs facets endpoint exactly (`Vec<FacetField>`, LIMIT 20, silent partial failure) — consistency over customisation.
- Abstract `FacetPanel` over a minimal interface rather than duplicating the widget — the two controllers are structurally identical for facet purposes (`filter`, `from`, `to`, `rangeVersion`, `setFilter`, `reload`, `apiClient`).
