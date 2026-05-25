# Agent guidelines

Instructions for AI coding agents (Claude Code, Copilot, Cursor, etc.) working in this repo.

## Project overview

`sidekiq-rs` (published as `rusty-sidekiq`) is a Rust reimplementation of the Sidekiq background-job framework, fully compatible with `sidekiq.rb` for both enqueuing and processing jobs. It is built on Tokio for async execution and uses `bb8` for Redis connection pooling. Workers are defined by implementing the `Worker<Args>` trait where `Args` is a strongly-typed, `serde`-deserializable struct. The library also supports server middleware, periodic (cron) jobs, unique-job deduplication, Redis namespacing, and configurable worker concurrency.

## Local setup

```bash
git clone git@github.com:redis-performance/sidekiq-rs.git
cd sidekiq-rs
cargo build
```

The integration tests require a Redis server listening on `redis://127.0.0.1/` (default port 6379). Start one before running tests:

```bash
redis-server &
```

## Branch naming

Same as human contributors: `<type>/<short-description>` (e.g. `fix/off-by-one-in-pipeline`).

## Coding standards

- Match the style already in the file you are editing.
- Prefer clear, minimal changes over large refactors unless explicitly asked.
- Do not add comments that describe *what* the code does — only add comments when the *why* is non-obvious.
- Do not introduce new dependencies without checking with the maintainer.

## Running tests

Run the full test suite:

```bash
cargo test --verbose
```

Run Clippy before declaring a task complete:

```bash
cargo clippy -- -D warnings
```

Always run tests before declaring a task complete.

## How to submit changes

1. Create a branch: `git checkout -b <type>/<description>`.
2. Commit with a clear message focused on *why*, not *what*.
3. Open a pull request against `master`.
4. Do **not** push directly to `master`.

## What to avoid

- Do not reformat files unrelated to your change.
- Do not remove error handling or tests.
- Do not commit secrets, credentials, or large binary files.
- Do not amend published commits.
- Do not add new crate dependencies to `Cargo.toml` without explicit maintainer approval.
- Do not change the public API surface (trait signatures, public structs, method names) without an accompanying discussion in the PR.
