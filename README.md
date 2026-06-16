# slack-actions-notify

[![GitHub marketplace](https://img.shields.io/badge/marketplace-slack--actions--notify-blue?logo=github)](https://github.com/marketplace/actions/slack-actions-notify)
[![GitHub release](https://img.shields.io/github/v/release/Iamrushabhshahh/slack-actions-notify)](https://github.com/Iamrushabhshahh/slack-actions-notify/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A GitHub Action for Slack notifications that does the one thing most others don't — it edits the same message instead of spamming a new one. Post a "workflow started" message, then update it in-place when the workflow finishes with status, duration, and a link back to the run.

Also works as a simple one-shot notifier if you don't need threading.

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
| `status` | auto | Override the workflow status. Auto-detected from `job.status` if omitted. |
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

---

## A few examples

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

## License

MIT
