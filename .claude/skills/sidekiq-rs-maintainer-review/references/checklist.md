# Review checklist — redis-performance/sidekiq-rs, honestly graded by evidence strength

6 categories. **Item 1 has real, repeated precedent** (two real PR numbers, the same bug class both times).
**Items 2–5 are documented project policy** (`AGENTS.md`/`CONTRIBUTING.md`) that has never been observed
enforced against an actual PR — this fork has zero recorded review comments, so there is no "a maintainer once
flagged this" to cite honestly. Item 6 is a single-sample process signal from one still-open PR's own
description. See `history.md` for why the evidence base here is this thin.

1. **Pinned versions in `.github/workflows/rust.yml` must track current org/upstream standards, not just "still
   green."** Real, repeated precedent: PR #2 (`actions/checkout@v3` → `@v6`, ahead of the node20 runner
   deprecation) and PR #3 (the `redis:` service-container image `redis:6` → `redis:8.6`, "per org standard").
   Both were self-caught by the PR author, not flagged by review. Any PR touching `rust.yml` — a new action, a
   bumped/unbumped action version, the `redis:` image tag — should be checked against current org standard
   directly; nothing else in CI verifies this for you, and it has gone stale twice already.

2. **New/changed public API surface needs an accompanying discussion, and a new opt-in behavior should default
   to the pre-existing behavior with a test locking that default in.** Verbatim from `AGENTS.md`: "Do not change
   the public API surface (trait signatures, public structs, method names) without an accompanying discussion in
   the PR." The one real example of doing this well is PR #4's `ProcessorConfig::track_stats` /
   `brpop_latency_tx` fields (both default to the pre-existing behavior, one with an explicit
   `..._default_off` test) — but PR #4 is still open and unreviewed as of this mining, so treat that pattern as
   "the bar to hold a similar PR to," not as "a reviewer has required this before."

3. **New dependencies need maintainer sign-off; new behavior needs test coverage; coverage should not decrease.**
   Verbatim from `AGENTS.md`/`CONTRIBUTING.md`: "Do not introduce new dependencies without checking with the
   maintainer"; "All new behaviour must be covered by tests"; "Coverage should not decrease." Documented policy,
   never observed enforced in a real review comment because this fork has no recorded review comments at all —
   `rust.yml`'s CI run proves existing tests pass and `cargo clippy -- -D warnings` is clean, it does not prove
   new code has any test coverage. Check this directly; CI cannot.

4. **No dead code, no commented-out blocks, no reformatting unrelated to the change, comments explain "why" not
   "what."** Verbatim from `AGENTS.md`/`CONTRIBUTING.md`. Same status as item 3: written policy, never seen
   enforced in a real review on this fork. Raise it on its own merits if you see it, not as "you're repeating a
   mistake others made here" — nobody has, on the record.

5. **Async/tokio-specific correctness, since CI cannot see this either.** `cargo test --verbose` and `cargo
   clippy -- -D warnings` run against a live `redis:8.6` service container and are a real backstop for build
   errors, lint violations, and existing-test regressions — but they don't catch a blocking call inside an
   `async fn`, a `std::sync::Mutex` (rather than `tokio::sync::Mutex`) held across an `.await` point, an
   `unwrap()`/`panic!()` newly introduced into a worker's per-job hot path (where Ruby-Sidekiq-compatible error
   handling matters — a panicked worker task is a silent stop, not a job-level failure), or a new Redis command
   sequence that isn't atomic/pipelined where the surrounding code already establishes that pattern. Worth
   checking directly on any PR touching `src/processor.rs`, `src/redis.rs`, or `src/stats.rs` — the three files
   PR #4 touches, and where this class of bug would actually live.

6. **A well-formed engineering PR here states plainly what's verified and what's still outstanding.** Single
   real example: PR #4's `## Test plan` uses checked boxes for what actually ran (unit tests, a local smoke
   test) and an explicit unchecked box for a pending large-scale run, plus a `## Stacking note` disclosing a
   branch-ordering dependency on another unlanded branch. This is one PR's own self-description, not anything an
   independent reviewer has confirmed (the PR is still open) — treat it as a useful example of the bar, not as
   institutional doctrine or a rule anyone has enforced.
