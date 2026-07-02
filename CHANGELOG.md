# Changelog

All notable changes to this project are documented here. Format loosely follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning follows
[VERSIONING.md](VERSIONING.md).

## [Unreleased]

## [1.1.1] — 2026-07-02

### Added

- CI (`.github/workflows/lint.yml`): every push to `main` and every PR now runs
  schema validation (`action-validator`, against the official action metadata
  schema) and shellcheck across every embedded `run:` script, gated on
  error/warning-level findings only (style-level nits are reported but don't
  block). This repo previously had no CI of its own at all.

### Fixed

- Two shellcheck style findings (SC2129) in the "Auto-generate metadata" step,
  cleaned up while adding the linter above. No behavior change.

## [1.1.0] — 2026-07-02

### Fixed

- `chat.update` (and `chat.postMessage`/`webhooks.execute`) no longer retry 5 times
  with growing backoff on failure — capped to `retries: 2`, so a real Slack-side
  failure now costs seconds instead of up to ~18 minutes of job time.
- `mode: update` now falls back to posting a **new** message if editing the original
  fails, instead of leaving it stuck on its previous status ("Started") forever.
- `status: skipped` (a normal `needs.<job>.result` value) no longer hard-fails the
  step. Any unrecognized status string now degrades to a neutral label instead of
  exiting with an error — this action is meant to be safe to drop into any job with
  `if: always()`, and a config/data edge case shouldn't break that guarantee.
- An unparseable `start_time` no longer produces a nonsensical multi-decade duration
  — it's now omitted with a `::warning::` instead.
- A `start_time` in the future (clock skew) no longer produces a negative duration.
- `mode` is now validated to be exactly `start` or `update` at the top of the
  action. Previously an invalid value (e.g. a typo'd `mode: Start`) silently fell
  through to `update` behavior instead of failing with a clear error.

### Added

- New `ok` output: `'true'`/`'false'` reflecting whether the notification was
  actually delivered (including via the new fallback post). Lets callers react to
  a delivery failure themselves without coupling their build's result to Slack's
  uptime.

## [1.0.0] — 2026-06-16

Initial public release.

### Added

- `mode: start` / `mode: update` threading — post once, edit in place with final
  status, duration, and a link back to the run.
- Bot token (`slack_token`) and incoming webhook (`slack_webhook_url`) auth.
- Auto-detected status/color/duration/commit metadata, PR- and push-aware fields.
- `title`, `message`, `message_on_success`/`_failure`/`_cancel`, `mention`,
  `minimal`, and bot-appearance (`username`, `icon_emoji`, `icon_url`) inputs.
- `timezone` support for all rendered timestamps.
