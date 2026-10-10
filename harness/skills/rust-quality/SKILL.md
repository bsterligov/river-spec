---
name: rust-quality
description: Rust quality gates for River crates — formatting, lints, tests, duplication and complexity limits.
user-invocable: false
---

When writing or changing Rust code in a River repo:

1. **Run the gates before finishing.** All must pass:
   - `mise run ci:cargo-fmt-check` (run `mise exec -- cargo fmt` to fix)
   - `mise run ci:cargo-clippy` (warnings are errors)
   - `mise run ci:cargo-test`
2. **Tests live next to code** in `*_tests.rs` files. Follow the spec's Test Approach (TDD or BDD). Use `wiremock` for HTTP dependencies and `temp-env` for env-driven config.
3. **No `unwrap()` / `expect()`** outside tests; propagate errors with `anyhow`.
4. **Duplication under 3%.** If the pack ships `tools/check-duplication`, run `python3 .pack/tools/check-duplication .` and fix what it flags.
5. **Cognitive complexity at most 15 per function.** Extract named helpers instead of nesting `if`/`for`/`match`.
6. **Config is env-only** through the `river-config` crate; every variable is prefixed `RIVER_`.
