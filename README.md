# slack-actions-notify

[![GitHub marketplace](https://img.shields.io/badge/marketplace-slack--actions--notify-blue?logo=github)](https://github.com/marketplace/actions/slack-actions-notify)
[![GitHub release](https://img.shields.io/github/v/release/Iamrushabhshahh/slack-actions-notify)](https://github.com/Iamrushabhshahh/slack-actions-notify/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A GitHub Action for Slack notifications that does the one thing most others don't — it edits the same message instead of spamming a new one. Post a "workflow started" message, then update it in-place when the workflow finishes with status, duration, and a link back to the run.

Also works as a simple one-shot notifier if you don't need threading.

See [CHANGELOG.md](CHANGELOG.md) for release history and [VERSIONING.md](VERSIONING.md) for the versioning policy.

## Contents

- [Why this instead of the alternatives?](#why-this-instead-of-the-alternatives)
- [How the threading works](#how-the-threading-works)
- [Setup](#setup)
- [Quickstart](#quickstart)
- [Inputs](#inputs)
- [Outputs](#outputs)
- [How failures are handled](#how-failures-are-handled)
- [Troubleshooting](#troubleshooting)
- [More examples](#more-examples)
- [Contributing](#contributing)
- [License](#license)

---

## Why this instead of the alternatives?

| | slack-actions-notify | Raw `slackapi/slack-github-action` | Typical one-shot notifiers (`rtCamp/action-slack-notify`, `8398a7/action-slack`) |
|---|---|---|---|
| Edits the start message in place | ✅ | You wire this yourself | ❌ — always posts a new message |
| Auto duration / timestamps / branch / commit fields | ✅ | You build the payload yourself | Partial |
| Never fails your build on a Slack-side failure | ✅ (opt-in visibility via the `ok` output) | Depends how you configure `errors` | Varies by action |
| If the in-place edit can't land, still gets a status into the channel | ✅ (fallback post) | You wire this yourself | N/A (always posts fresh anyway) |

If you don't need threading and don't mind a message per event, a lighter one-shot notifier is a totally reasonable choice. This action exists for the "keep the channel readable during a busy CI/CD day" case.

---

## How the threading works

Most Slack notify actions post two separate messages (start + result). This one posts one message and edits it. Your Slack channel stays clean.

```
[🟡 Push Workflow Started]      ← posted by mode: start
        ↓ workflow runs...
[✅ Push Workflow Succeeded]    ← same message, updated by mode: update
   Duration: 2m 14s
   Started: 10:32 16-06-2026
```

---

## Setup

**Slack side:**

1. Go to [api.slack.com/apps](https://api.slack.com/apps) → Create New App → From Scratch
2. Under *OAuth & Permissions*, add `chat:write` (and `chat:write.public` if you want to post to public channels without the bot being a member)
3. Install to workspace, copy the Bot User OAuth Token (`xoxb-…`)
4. Get your channel ID: open Slack in a browser, go to the channel, it's the last part of the URL

**GitHub side:**

Add to your repo secrets (*Settings → Secrets and variables → Actions*):
- `SLACK_BOT_TOKEN`
- `SLACK_CHANNEL_ID`

Prefer a webhook instead of a bot token? Skip straight to [Quickstart](#quickstart) — webhooks need nothing beyond the URL itself, just note they don't support threading (see [Inputs](#inputs)).

---

## Quickstart

**Just want a notification when your workflow finishes:**

```yaml
- name: Notify Slack
  if: always()
  uses: Iamrushabhshahh/slack-actions-notify@v1
  with:
    slack_channel: ${{ secrets.SLACK_CHANNEL_ID }}
    slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
```

**Start → update pattern (recommended for CI/CD):**

```yaml
jobs:
  notify-start:
    runs-on: ubuntu-latest
    outputs:
      ts: ${{ steps.notify.outputs.ts }}
      start: ${{ steps.notify.outputs.start_timestamp }}
    steps:
      - id: notify
        uses: Iamrushabhshahh/slack-actions-notify@v1
        with:
          mode: start
          slack_channel: ${{ secrets.SLACK_CHANNEL_ID }}
          slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
          timezone: Asia/Kolkata

  build:
    needs: notify-start
    runs-on: ubuntu-latest
    steps:
      - run: npm ci && npm test

  notify-done:
    needs: [notify-start, build]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - uses: Iamrushabhshahh/slack-actions-notify@v1
        with:
          mode: update
          slack_ts: ${{ needs.notify-start.outputs.ts }}
          start_time: ${{ needs.notify-start.outputs.start }}
          status: ${{ needs.build.result }}
          slack_channel: ${{ secrets.SLACK_CHANNEL_ID }}
          slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
          timezone: Asia/Kolkata
```

**Using a webhook instead of a bot token:**

```yaml
- uses: Iamrushabhshahh/slack-actions-notify@v1
  with:
    slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
    message: "Deployed to production"
```

Note: webhook mode doesn't support start→update threading. You'll need a bot token for that.

---

## Inputs

### Auth

| Input | Description |
|-------|-------------|
| `slack_token` | Bot token (`xoxb-…`). Needed for threading. Requires `slack_channel`. |
| `slack_webhook_url` | Incoming webhook URL. Simpler, but no threading support. |

You need one or the other. If you pass both, token takes precedence.

### Core

| Input | Default | Description |
|-------|---------|-------------|
| `slack_channel` | | Channel ID (e.g. `C1234567890`). Required with `slack_token`. |
| `mode` | `update` | `start` to post a new message, `update` to edit the previous one. |
| `slack_ts` | | The `ts` output from a `start` step. Required for `mode: update`. |
| `start_time` | | The `start_timestamp` output from a `start` step. Used to compute duration. |
| `status` | auto | Override the workflow status — typically `${{ needs.<job>.result }}` (`success`/`failure`/`cancelled`/`skipped`, all handled). Auto-detected from `job.status` if omitted. |
| `timezone` | `UTC` | Any valid tz name — `Asia/Kolkata`, `America/New_York`, etc. Falls back to UTC. |

### Content

| Input | Description |
|-------|-------------|
| `title` | Custom notification title. Defaults to `repo/workflow — status`. |
| `message` | Extra text shown below the fields. |
| `message_on_success` | Overrides `message` when the workflow succeeds. |
| `message_on_failure` | Overrides `message` when the workflow fails. |
| `message_on_cancel` | Overrides `message` when the workflow is cancelled. |
| `mention` | Prepend a Slack mention. Pass `here`, `channel`, a user ID (`U123456`), or `<@U123456>`. |
| `minimal` | Set to `true` to strip the metadata fields and show only branch, commit, and duration. |

### Appearance

These only apply when posting a new message (`mode: start`). Editing an existing message can't change the bot's appearance.

| Input | Description |
|-------|-------------|
| `username` | Custom bot display name. |
| `icon_emoji` | Bot icon emoji, e.g. `:rocket:`. Overrides `icon_url`. |
| `icon_url` | Bot icon URL. |

---

## Outputs

| Output | Description |
|--------|-------------|
| `ts` | Slack message timestamp. Pass as `slack_ts` to the update step. |
| `start_timestamp` | Workflow start time. Pass as `start_time` to the update step. |
| `ok` | `'true'` if the notification was actually delivered (directly or via the fallback post described below), `'false'` if every attempt failed. The action itself never fails on delivery failure — check this if you want to react to one yourself. |

---

## How failures are handled

This action is designed to be safe to drop into any job with `if: always()` — a broken Slack integration should never be the reason your CI/CD goes red. That has a few concrete implications:

- **It retries, then gives up quickly.** Each Slack API call gets 2 retries (a few seconds of backoff total), not GitHub's default of 5 — so a real Slack-side outage costs you seconds, not minutes.
- **It never fails the step.** A failed Slack call is logged as a `::warning::` in the run log, not a step failure. Your build result reflects your code, not Slack's availability.
- **`mode: update` has a fallback.** If editing the original "started" message fails (expired token, deleted message, transient API error), it posts a **new** message with the final status instead of leaving the original stuck showing "Started" forever.
- **Unusual `status` values degrade gracefully.** If you pass `status: ${{ needs.some_job.result }}` and that job was skipped, you get a clean "Skipped" notification, not a broken workflow — same for any other status string GitHub might hand you.
- **You can still check delivery.** Use the `ok` output if you want visibility (e.g. log a warning, page a human a different way) without coupling your build's success to Slack's uptime.

---

## Troubleshooting

**Message never updates / stays stuck on "Started":**
Almost always one of: the bot's token expired or was regenerated between the `start` and `update` steps, the message was deleted, or `slack_ts`/`start_time` weren't threaded through from the `start` step's outputs correctly (check `needs.<start-job>.outputs.ts` is wired up). Check the `ok` output on the `update` step — if it's `'false'`, a fallback message should still have landed in the channel.

**`not_in_channel` / message never appears at all:**
The bot needs to be invited to the channel (`/invite @your-bot-name`), or add the `chat:write.public` OAuth scope to post to public channels without an invite.

**`missing_scope`:**
Your bot token needs the `chat:write` scope at minimum. Reinstall the app to your workspace after adding scopes — token doesn't update until you do.

**Duration missing from an update:**
`start_time` wasn't passed, or couldn't be parsed. Check for a `::warning::` in the `start` job's logs.

---

## More examples

**Alert on failure with @here mention:**

```yaml
- name: Alert on failure
  if: failure()
  uses: Iamrushabhshahh/slack-actions-notify@v1
  with:
    slack_channel: ${{ secrets.SLACK_ALERTS_CHANNEL }}
    slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
    mention: here
    message_on_failure: "Build broke on ${{ github.ref_name }}. Someone take a look."
```

**Minimal notification for noisy repos:**

```yaml
- uses: Iamrushabhshahh/slack-actions-notify@v1
  if: always()
  with:
    slack_channel: ${{ secrets.SLACK_CHANNEL_ID }}
    slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
    minimal: "true"
```

**Custom bot branding:**

```yaml
- uses: Iamrushabhshahh/slack-actions-notify@v1
  with:
    slack_channel: ${{ secrets.SLACK_CHANNEL_ID }}
    slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
    mode: start
    username: deploy-bot
    icon_emoji: ":ship:"
    title: "Deploying to production..."
```

**Per-status messages:**

```yaml
- uses: Iamrushabhshahh/slack-actions-notify@v1
  with:
    slack_channel: ${{ secrets.SLACK_CHANNEL_ID }}
    slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
    mode: update
    slack_ts: ${{ steps.start.outputs.ts }}
    start_time: ${{ steps.start.outputs.start_timestamp }}
    message_on_success: "Shipped. All good."
    message_on_failure: "Something went wrong — check the run."
    message_on_cancel: "Cancelled."
```

---

## Contributing

Issues and PRs welcome. A few things that make review faster:

- This is a single composite `action.yaml` — no build step, no `dist/` to keep in sync. Edit the file directly.
- Test against a real Slack workspace before opening a PR; there's no automated test suite (see [Known limitations](#known-limitations)).
- Follow [VERSIONING.md](VERSIONING.md) for how a change should be versioned, and add an entry to [CHANGELOG.md](CHANGELOG.md).

### Known limitations

- No automated tests — changes are verified by running them against a real workflow. If you're comfortable setting up a CI job that exercises this against a Slack sandbox workspace, that'd be a welcome contribution.
- Uses Slack's legacy `attachments` format rather than Block Kit — simpler to generate from bash/jq, at the cost of some newer Slack formatting features.
- `errors: false` + capped retries is an opinionated default (see [How failures are handled](#how-failures-are-handled)); there's currently no input to opt back into "fail my build if Slack is down" for teams that want that instead.

---

## License

MIT
