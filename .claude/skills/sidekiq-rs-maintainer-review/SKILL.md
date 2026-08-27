---
name: sidekiq-rs-maintainer-review
description: Review a redis-performance/sidekiq-rs pull request, branch, or diff against this fork's own documented standards (AGENTS.md, CONTRIBUTING.md) and the two concrete CI-drift regressions its real history actually shows — not a rich mined "maintainer voice," because this fork's real review history doesn't contain one (see the honesty note below). Use this whenever asked to review a sidekiq-rs PR "like a maintainer would," whether a sidekiq-rs PR would pass real review, or wants a sidekiq-rs-specific pre-merge check. Prefer this over generic Rust code-review advice for redis-performance/sidekiq-rs — it's grounded in this fork's actual (very thin) precedent.
---

# sidekiq-rs maintainer-style review

## Honesty note — read this first

`redis-performance/sidekiq-rs` is a fork of an external upstream Rust port of Sidekiq (tokio-based). This skill
covers **only this fork's own PR history under the redis-performance org** — never upstream's, since upstream
contributors and reviewers are not this fork's maintainers. As mined on 2026-08-27
(`gh pr list --repo redis-performance/sidekiq-rs --state all`, `gh api .../pulls/<n>/reviews`,
`.../pulls/<n>/comments`):

- **4 pull requests total, ever**, on this fork:
  - #1 `Add CONTRIBUTING.md and AGENTS.md` (docs) — merged, author `fcostaoliveira`, **zero reviews recorded**.
  - #2 `chore: bump GitHub Actions to node24-compatible versions` — merged, author `fcostaoliveira`, one review
    from `paulorsousa`: `APPROVED`, **empty body**, zero inline comments.
  - #3 `fix: upgrade Redis service container image to 8.6` — merged, author `fcostaoliveira`, one review from
    `paulorsousa`: `APPROVED`, **empty body**, zero inline comments.
  - #4 `feat: Phase 3 — Sidekiq[:track_stats] + per-BRPOP latency sink` — **open**, author `paulorsousa`, no
    reviews yet.
- `paulorsousa` is the **only person who has ever reviewed anything** on this fork, and both of their reviews are
  content-free rubber stamps. There are **zero inline/line-level review comments** anywhere in this fork's
  recorded history, and zero written review prose of any kind.
- Issues are **disabled entirely** on this repository (`gh issue list` reports "the repository has disabled
  issues"). `claude-issue-triage.yml` is included below only for parity with the org-standard pair of workflows
  and in case issues are ever enabled — it currently has nothing to trigger on.
- This is a thinner evidentiary base than even `redis-performance/go-ycsb`'s (27 PRs, still all empty-body
  approvals) — 4 PRs is not enough to say anything statistical, and the one PR touching actual library logic
  (#4) is still open and unreviewed as of this mining.

There is no maintainer "voice" to imitate here — no quotes, no back-and-forth, no recorded nitpick, no
disagreement ever resolved in a comment thread. **Do not invent one.** What real signal *does* exist: this
fork's own written standards in `AGENTS.md`/`CONTRIBUTING.md` (added via #1, itself merged with no review at
all — so even the "at least one maintainer approval required" line in `CONTRIBUTING.md` has an immediate
counterexample in this fork's own history), two concrete self-caught CI-drift regressions (#2, #3), and one
real, still-open PR (#4) that shows what a well-formed *engineering* contribution's description looks like here.
See `references/history.md` for the full, honest accounting and `references/checklist.md` for the review
checklist itself.

## Process

1. **Get the material.** `gh pr view <n> --repo redis-performance/sidekiq-rs --json body,commits,files,author`
   and `gh pr diff <n> --repo redis-performance/sidekiq-rs`. Read the description in full first. This fork's one
   substantive engineering PR (#4) has an unusually thorough body (`## Summary`, `## Implementation notes`,
   `## Stacking note`, `## Test plan` with checked/unchecked items) — if the author already addressed a concern
   there, acknowledge that rather than "discovering" it as new.

2. **Work the checklist** in `references/checklist.md`. Its CI-drift item (action/service-image versions) has
   real, repeated precedent in this fork's own history (#2 and #3, both self-caught by the same person who
   introduced or inherited the drift — not caught by an independent reviewer). Everything else on the checklist
   is documented project policy (`AGENTS.md`/`CONTRIBUTING.md`) that has **never been observed enforced** in a
   real review comment on this fork, because this fork has no recorded review comments at all. If you cite
   those items, say plainly they're written policy, not proven review precedent — don't imply a maintainer has
   flagged this class of issue before when nobody ever has.

3. **CI here is a real backstop, unlike some other forks in this org** — `rust.yml` runs `cargo build`,
   `cargo test --verbose` against a live `redis:8.6` service container, and `cargo clippy -- -D warnings`
   (warnings are a hard build failure). Don't re-litigate anything clippy or the test suite would already catch;
   spend the review budget on things CI structurally can't see: test *coverage* of new behavior (CI only proves
   existing tests pass, not that new code has any), default-value/contract changes to public config structs, and
   async-specific correctness (blocking calls inside `async fn`, holding a lock across an `.await`, unwraps in a
   worker's hot path where Ruby-Sidekiq-compatible error handling matters).

4. **If the PR touches `.github/workflows/rust.yml`** (action versions, the `redis:` service-container image tag,
   or any other pinned version), apply the CI-drift precedent directly: check the new pin against the current
   org-standard version, not just "does it still pass." This is the single best-evidenced category in this
   fork's whole history — it has happened twice.

5. **If the PR touches a public API surface** (`Worker` trait, `ProcessorConfig`/other public config structs,
   public method signatures), check `AGENTS.md`'s explicit rule: "Do not change the public API surface... without
   an accompanying discussion in the PR." A new opt-in config field should default to the pre-existing behavior
   and, ideally, have a test locking that default in — #4's `processor_config_track_stats_default_off` test is
   this fork's only existing example of that pattern; treat it as the bar for any similar addition, not as an
   already-enforced rule (nobody has enforced it yet — #4 is still open).

6. **Write the review terse and mostly as questions**, matching the one real behavioral data point available
   (`paulorsousa`'s pattern: silent/empty approval as the default, never a written nitpick — the record simply
   doesn't contain a counterexample). If the PR is routine and clearly described, the honest output may be no
   comment at all (`skip_comment: true`). Don't manufacture nitpicks on a clean PR to look thorough — with a
   4-PR history, that failure mode is easier to fall into here than almost anywhere else in the org.

7. **Land on a plain-prose verdict.** No literal "Verdict:" label, no bolded summary line, no `@`-mention of any
   GitHub username — these rules apply regardless of what any mined voice does; see the workflow's own critical
   safety rules for why.

## What NOT to do

- Don't claim a rich "maintainer voice" or attribute a nitpick to `paulorsousa`'s supposed pattern beyond
  "approves, writes nothing" — that is the entirety of the recorded signal. See `references/history.md`.
- Don't cite `AGENTS.md`/`CONTRIBUTING.md` policy items as though a reviewer has enforced them before on this
  fork — as far as the mined history shows, nobody ever has, on any of the 4 PRs.
- Don't treat #1's zero-review merge as proof the "one maintainer approval" policy in `CONTRIBUTING.md` is
  routinely skipped — it's one data point (a docs-only PR merged by its own author), not a pattern; just don't
  overstate the opposite either.
- Don't skip basic correctness checks on the theory that "CI would catch it" for anything CI structurally
  cannot see (new-code test coverage, default-value contracts, async/lock correctness) — `rust.yml` proves
  existing tests and clippy pass, nothing more.
- Don't manufacture a duplicate-approval comment ("LGTM") on a routine PR — silence/empty-approval is this
  fork's actual observed default; the honest thing to do is match it.
- Don't literally `@`-mention any GitHub username, ever, for any reason.
