# GitHub Contribution Graph Backfill

Automatically populate your GitHub contribution graph with commits.

## Setup

This repository uses GitHub Actions to:
1. **Backfill** 2 years of weekday commits (Feb 8, 2024 to Feb 8, 2026)
2. **Daily automation** to create 1-3 commits every weekday

## How to Use

### 1. One-Time Backfill (2 Years)

To backfill all weekdays from Feb 8, 2024 to Feb 8, 2026 with 1-5 commits per day:

1. Go to your repository on GitHub
2. Click on **Actions** tab
3. Select **Backfill 2 Years (2024-2026)** workflow
4. Click **Run workflow**
5. Wait for it to complete (may take a few minutes)

This will create random commits (1-5 per day) for all weekdays in the past 2 years.

### 2. Daily Automation

The **Daily Weekday Commits** workflow runs automatically:
- **Schedule**: Monday-Friday at 9 AM UTC
- **Commits**: 1-3 random commits per day
- **Smart**: Skips if commits already exist for that day

You can also trigger it manually:
1. Go to **Actions** tab
2. Select **Daily Weekday Commits** workflow
3. Click **Run workflow**

## Configuration

Both workflows use your email: `emmanuelcuyugan@gmail.com`

To modify settings, edit the workflow files:
- `.github/workflows/backfill-2-years.yml` - One-time backfill
- `.github/workflows/daily-weekday-commits.yml` - Daily automation

### Commit Settings

You can adjust:
- `MIN_COMMITS_PER_DAY` / `MAX_COMMITS_PER_DAY`: Number of commits per day
- `INCLUDE_WEEKENDS`: true/false for weekend commits
- `GIT_COMMIT_MESSAGE`: Custom commit messages

## Troubleshooting

**Commits not showing on GitHub?**
- Verify email `emmanuelcuyugan@gmail.com` is added to your GitHub account
- Check that the email is set as a verified email
- Go to Settings → Emails on GitHub

**Action failed?**
- Check that Actions have write permissions in repository settings
- Verify `GITHUB_TOKEN` has proper permissions

## How It Works

Uses [bcanseco/github-contribution-graph-action](https://github.com/marketplace/actions/autopopulate-your-contribution-graph) to create backdated commits with proper timestamps, populating your contribution graph.
