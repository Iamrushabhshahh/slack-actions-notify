# Slack Status Notifier

[![GitHub marketplace](https://img.shields.io/badge/marketplace-slack--actions--notify-blue?logo=github)](https://github.com/marketplace/actions/slack-actions-notify)
[![GitHub release](https://img.shields.io/github/v/release/Iamrushabhshahh/slack-actions-notify)](https://github.com/Iamrushabhshahh/slack-actions-notify/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Send rich Slack notifications for GitHub Actions workflow status updates with timestamps, duration tracking, and detailed metadata.

## ✨ Features

- 🚀 **Rich notifications** with workflow details, commit info, and execution metadata
- ⏱️ **Duration tracking** between workflow start and completion
- 🌍 **Timezone support** for displaying timestamps in your preferred timezone
- 🔄 **Thread updates** - start with initial notification, then update the same message with final status
- 📊 **Comprehensive metadata** including commit message, branch, triggered by, and more
- 🎨 **Status-based styling** with appropriate colors and emojis for success, failure, and cancelled workflows
- 🔗 **Direct links** to GitHub workflow runs, commits, branches, and user profiles

## 🚀 Quick Start

### Basic Usage

```yaml
name: CI/CD Pipeline
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Your build steps here
      - name: Build and test
        run: |
          npm install
          npm test
          npm run build
      
      # Notify on completion
      - name: Notify Slack
        if: always()
        uses: Iamrushabhshahh/slack-actions-notify@v1
        with:
          slack_channel: ${{ secrets.SLACK_CHANNEL_ID }}
          slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
```

### Advanced Usage with Thread Updates

```yaml
name: Advanced CI/CD with Thread Updates
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Send initial notification
      - name: Notify Slack - Start
        id: slack-start
        uses: Iamrushabhshahh/slack-actions-notify@v1
        with:
          slack_channel: ${{ secrets.SLACK_CHANNEL_ID }}
          slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
          mode: start
          timezone: 'America/New_York'
      
      # Your build steps
      - name: Build and test
        run: |
          npm install
          npm test
          npm run build
      
      # Update the same message with final status
      - name: Notify Slack - Complete
        if: always()
        uses: Iamrushabhshahh/slack-actions-notify@v1
        with:
          slack_channel: ${{ secrets.SLACK_CHANNEL_ID }}
          slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
          mode: update
          slack_ts: ${{ steps.slack-start.outputs.ts }}
          start_time: ${{ steps.slack-start.outputs.start_timestamp }}
          timezone: 'America/New_York'
```

## 📖 Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `slack_channel` | Slack channel ID where notifications will be sent | ✅ Yes | |
| `slack_token` | Slack bot token with `chat:write` permissions | ✅ Yes | |
| `mode` | Notification mode: `start` (initial) or `update` (final) | ❌ No | `update` |
| `status` | Workflow status (auto-detected if not provided) | ❌ No | Auto-detected |
| `slack_ts` | Slack thread timestamp (required for updates) | ❌ No | |
| `start_time` | Workflow start timestamp (for duration calculation) | ❌ No | |
| `timezone` | Timezone for timestamp display (e.g., 'Asia/Kolkata', 'America/New_York') | ❌ No | `UTC` |

## 📤 Outputs

| Output | Description |
|--------|-------------|
| `ts` | Slack message timestamp for thread updates |
| `start_timestamp` | Workflow start timestamp for duration calculation |

## 🔧 Setup

### 1. Create a Slack App

1. Go to [Slack API](https://api.slack.com/apps) and create a new app
2. Add the following OAuth scopes in **OAuth & Permissions**:
   - `chat:write`
   - `chat:write.public` (if posting to public channels)
3. Install the app to your workspace
4. Copy the **Bot User OAuth Token** (starts with `xoxb-`)

### 2. Get Channel ID

1. Open Slack in your browser
2. Navigate to the desired channel
3. Copy the channel ID from the URL (e.g., `C1234567890`)

### 3. Configure GitHub Secrets

Add these secrets to your repository (**Settings** → **Secrets and variables** → **Actions**):

- `SLACK_BOT_TOKEN`: Your Slack bot token
- `SLACK_CHANNEL_ID`: Your Slack channel ID

## 🌍 Timezone Support

The action supports any valid timezone identifier. Examples:

- `UTC` (default)
- `America/New_York`
- `Europe/London`
- `Asia/Kolkata`
- `Pacific/Auckland`

Invalid timezones will automatically fall back to UTC with a warning.

## 📱 Notification Preview

The action sends rich notifications that include:

**Initial notification (mode: start):**
- 🟡 Yellow indicator for "in progress"
- Workflow start timestamp
- Basic metadata (branch, commit, triggered by)

**Final notification (mode: update):**
- 🟢 Green for success / 🔴 Red for failure / ⚪ Gray for cancelled
- Execution duration
- Complete metadata including commit message
- Links to GitHub resources

## 🔄 Workflow Modes

### Start Mode
Use `mode: start` to send an initial notification when your workflow begins:
```yaml
- uses: yourusername/slack-actions-notify@v1
  with:
    mode: start
    # ... other inputs
```

### Update Mode (Default)
Use `mode: update` (or omit the mode) for final status notifications:
```yaml
- uses: yourusername/slack-actions-notify@v1
  with:
    mode: update
    slack_ts: ${{ steps.slack-start.outputs.ts }}
    start_time: ${{ steps.slack-start.outputs.start_timestamp }}
    # ... other inputs
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

If you encounter any issues or have questions:

1. Check the [Issues](https://github.com/Iamrushabhshahh/slack-actions-notify/issues) page
2. Create a new issue with detailed information
3. Include your workflow configuration and any error messages

## 🏷️ Examples by Use Case

### Simple Success/Failure Notifications
```yaml
- name: Notify on completion
  if: always()  # Runs regardless of workflow outcome
  uses: yourusername/slack-actions-notify@v1
  with:
    slack_channel: ${{ secrets.SLACK_CHANNEL_ID }}
    slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
```

### Conditional Notifications

#### Only Notify on Failures
```yaml
- name: Alert on Failure
  if: failure()
  uses: yourusername/slack-actions-notify@v1
  with:
    slack_channel: ${{ secrets.SLACK_ALERT_CHANNEL }}
    slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
```

#### Production Branch Only
```yaml
- name: Production Deploy Notification
  if: github.ref == 'refs/heads/main'
  uses: yourusername/slack-actions-notify@v1
  with:
    slack_channel: ${{ secrets.SLACK_PROD_CHANNEL }}
    slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
```

#### Critical Failures (Main + Release Branches)
```yaml
- name: Critical Failure Alert
  if: failure() && (github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/heads/release/'))
  uses: yourusername/slack-actions-notify@v1
  with:
    slack_channel: ${{ secrets.SLACK_CRITICAL_CHANNEL }}
    slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
```

### Deployment Notifications with Custom Timezone
```yaml
- name: Deployment Complete
  uses: yourusername/slack-actions-notify@v1
  with:
    slack_channel: ${{ secrets.SLACK_DEPLOY_CHANNEL }}
    slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
    timezone: 'Europe/London'
```

### Multi-Job Workflow with Thread Updates
```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      slack_ts: ${{ steps.notify.outputs.ts }}
      start_time: ${{ steps.notify.outputs.start_timestamp }}
    steps:
      - name: Start Notification
        id: notify
        uses: Iamrushabhshahh/slack-actions-notify@v1
        with:
          slack_channel: ${{ secrets.SLACK_CHANNEL_ID }}
          slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
          mode: start

  build:
    needs: setup
    runs-on: ubuntu-latest
    steps:
      # ... build steps ...
      
  deploy:
    needs: [setup, build]
    runs-on: ubuntu-latest
    steps:
      # ... deploy steps ...
      - name: Final Notification
        if: always()
        uses: Iamrushabhshahh/slack-actions-notify@v1
        with:
          slack_channel: ${{ secrets.SLACK_CHANNEL_ID }}
          slack_token: ${{ secrets.SLACK_BOT_TOKEN }}
          mode: update
          slack_ts: ${{ needs.setup.outputs.slack_ts }}
          start_time: ${{ needs.setup.outputs.start_time }}
```