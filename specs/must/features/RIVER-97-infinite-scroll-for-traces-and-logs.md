# RIVER-97: Infinite Scroll for Traces and Logs
Priority: Must
Test Approach: BDD
Why: Users can only see the first 200 records on the logs and traces pages, making it impossible to investigate older or lower-ranked signals.
<!-- STOP -->

## Problem

Both the logs page and the traces page hard-cap results at 200 rows fetched in a single API call. Users with high-volume services hit this ceiling quickly and cannot reach records outside the first page. Applying a filter or changing sort order fetches a fresh 200-row batch but does not otherwise change this behaviour. There is no way to load more.

## Goal

**Given** the user is on the logs or traces page with at least one page of results loaded,
**When** they scroll to the bottom of the list,
**Then** the next batch of records appends below the existing rows without replacing them, and the scroll position is preserved.

**Given** the user changes a filter or sort column,
**When** the list reloads,
**Then** the accumulated rows are discarded and loading starts from the first page again.

**Given** the API returns fewer rows than the page size,
**When** the user reaches the bottom,
**Then** no further fetch is triggered (end-of-data state).

Success: a user can scroll continuously through all matching records with no artificial ceiling.

## Scope

**In**
- Offset-based pagination on `GET /v1/logs` and `GET /v1/traces` (add `offset` and `limit` query params; default `limit` = 100)
- `LogsController` and `TracesController` accumulate pages; reset on filter/sort/range change
- Scroll listener in `LogsTable` and `TracesTable` triggers next-page fetch when within ~200px of the bottom
- Loading indicator row appended at bottom while a page fetch is in progress
- End-of-data detection: stop fetching when response row count < page size
- Both controllers expose `hasMore` and `isLoadingMore` booleans for the table widgets

**Out**
- Cursor-based or keyset pagination (offset is sufficient given expected data volumes)
- Virtualised rendering / row recycling (Flutter `ListView` handles this natively)
- Pull-to-refresh
- Metrics page (uses a different data model)

## Decisions

- Page size is 100 rows (constant `_kPageSize = 100`); no user control over this.
- The `offset` param is appended to the existing ClickHouse SQL query via `OFFSET ?`; no schema change required.
- Reset is triggered by any change to filter, sort column, sort direction, or time range — same events that currently call `reload()`.
- Client-side sort (already applied via `_sortedRows()`) continues to operate on the accumulated list; server-side sort order must match so pages are consistent.
