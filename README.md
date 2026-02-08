# GitHub Contribution Graph Auto-Populator

Automatically populate your GitHub contribution graph with commits using GitHub Actions. Perfect for developers whose work happens on private repositories, other platforms (GitLab, Bitbucket), or self-hosted instances.

## 📋 Table of Contents

- [What This Does](#what-this-does)
- [Prerequisites](#prerequisites)
- [Complete Setup Guide](#complete-setup-guide)
- [How to Use](#how-to-use)
- [Configuration Options](#configuration-options)
- [Troubleshooting](#troubleshooting)
- [Technical Details](#technical-details)
- [FAQ](#faq)

---

## What This Does

This repository provides two automated workflows:

1. **One-Time Backfill** - Populates 2 years of historical commits (all weekdays with 1-5 commits per day)
2. **Daily Automation** - Automatically creates 1-3 commits every weekday to maintain your contribution streak

**Result:** A populated GitHub contribution graph showing consistent activity, even if your actual work happens elsewhere.

---

## Prerequisites

Before you begin, ensure you have:

- ✅ A GitHub account
- ✅ Git installed on your computer ([Download Git](https://git-scm.com/downloads))
- ✅ Basic familiarity with command line/terminal
- ✅ A verified email address on your GitHub account

---

## Complete Setup Guide

### Step 1: Create a New Repository

1. Go to [GitHub](https://github.com) and log in
2. Click the **"+"** icon in the top-right corner
3. Select **"New repository"**
4. Configure your repository:
   - **Repository name**: `activity_tracker` (or any name you prefer)
   - **Description**: "Automated contribution graph populator"
   - **Visibility**: ✅ **Public** (required for contribution graph)
   - ✅ Check **"Add a README file"**
   - Click **"Create repository"**

> ⚠️ **Important**: The repository MUST be public for commits to show on your contribution graph.

### Step 2: Enable GitHub Actions

1. In your new repository, go to **Settings** → **Actions** → **General**
2. Under **"Actions permissions"**, select:
   - ✅ **"Allow all actions and reusable workflows"**
3. Under **"Workflow permissions"**, select:
   - ✅ **"Read and write permissions"**
   - ✅ Check **"Allow GitHub Actions to create and approve pull requests"**
4. Click **"Save"**

### Step 3: Clone Repository to Your Computer

Open your terminal/command prompt and run:

```bash
# Clone your repository (replace YOUR_USERNAME with your GitHub username)
git clone https://github.com/YOUR_USERNAME/activity_tracker.git

# Navigate into the directory
cd activity_tracker
```

### Step 4: Create Workflow Files

Create the GitHub Actions workflow directory structure:

```bash
# Create the workflows directory
mkdir -p .github/workflows
```

#### Create Backfill Workflow

Create file `.github/workflows/backfill-2-years.yml`:

```yaml
# .github/workflows/backfill-2-years.yml
# One-time backfill: 2 years of weekday commits

name: Backfill 2 Years

on:
  workflow_dispatch:  # Manual trigger only

jobs:
  backfill-weekdays:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Backfill weekday commits (2 years)
      uses: bcanseco/github-contribution-graph-action@v2
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        GIT_EMAIL: your.email@example.com  # ⚠️ CHANGE THIS
        MAX_DAYS: 730  # 2 years
        MIN_COMMITS_PER_DAY: 1
        MAX_COMMITS_PER_DAY: 5
        INCLUDE_WEEKENDS: false  # Only weekdays
        INCLUDE_WEEKDAYS: true
        GIT_COMMIT_MESSAGE: "chore: automated contribution"

    - name: Verify backfill
      run: |
        echo "Checking commits from the last 730 days..."
        git log --oneline --since="730 days ago" | head -20
        TOTAL_COMMITS=$(git log --oneline --since="730 days ago" --author="your.email@example.com" | wc -l)
        echo "Total commits created: $TOTAL_COMMITS"
        echo "Backfill completed successfully!"
```

#### Create Daily Workflow

Create file `.github/workflows/daily-weekday-commits.yml`:

```yaml
# .github/workflows/daily-weekday-commits.yml
# Daily automation: 1-3 commits on weekdays

name: Daily Weekday Commits

on:
  workflow_dispatch:  # Manual trigger
  schedule:
    - cron: '0 9 * * 1-5'  # Monday-Friday at 9 AM UTC

jobs:
  daily-commits:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Check if today already has commits
      id: check-commits
      run: |
        TODAY=$(date +%Y-%m-%d)
        echo "Checking for commits on $TODAY"

        # Count commits for today
        COMMIT_COUNT=$(git log --since="$TODAY 00:00:00" --until="$TODAY 23:59:59" --author="your.email@example.com" --oneline | wc -l)
        echo "commit-count=$COMMIT_COUNT" >> $GITHUB_OUTPUT

        if [ $COMMIT_COUNT -gt 0 ]; then
          echo "Found $COMMIT_COUNT commits for today. Skipping."
          echo "skip=true" >> $GITHUB_OUTPUT
        else
          echo "No commits found for today. Proceeding."
          echo "skip=false" >> $GITHUB_OUTPUT
        fi

    - name: Create daily commits (1-3)
      if: steps.check-commits.outputs.skip == 'false'
      uses: bcanseco/github-contribution-graph-action@v2
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        GIT_EMAIL: your.email@example.com  # ⚠️ CHANGE THIS
        MAX_DAYS: 1  # Only today
        MIN_COMMITS_PER_DAY: 1
        MAX_COMMITS_PER_DAY: 3
        INCLUDE_WEEKENDS: false
        INCLUDE_WEEKDAYS: true
        GIT_COMMIT_MESSAGE: "chore: daily contribution"

    - name: Verify today's commits
      if: steps.check-commits.outputs.skip == 'false'
      run: |
        TODAY=$(date +%Y-%m-%d)
        echo "Commits for $TODAY:"
        git log --since="$TODAY 00:00:00" --until="$TODAY 23:59:59" --author="your.email@example.com" --oneline
        echo "Daily commits completed successfully!"

    - name: Skip notification
      if: steps.check-commits.outputs.skip == 'true'
      run: |
        echo "Skipping commit creation - today already has commits"
```

### Step 5: Configure Your Email

⚠️ **CRITICAL STEP**: Replace `your.email@example.com` in BOTH workflow files with your actual GitHub email.

To find your GitHub email:
1. Go to [GitHub Settings → Emails](https://github.com/settings/emails)
2. Use your primary email or any verified email listed there
3. Replace ALL instances of `your.email@example.com` in both `.yml` files

### Step 6: Commit and Push

```bash
# Configure git (if not already configured)
git config user.email "your.email@example.com"
git config user.name "Your Name"

# Add all files
git add .

# Commit
git commit -m "Add automated contribution workflows"

# Push to GitHub
git push origin main
```

### Step 7: Verify GitHub Actions Setup

1. Go to your repository on GitHub
2. Click the **"Actions"** tab
3. You should see two workflows:
   - ✅ Backfill 2 Years
   - ✅ Daily Weekday Commits

If you see a message about workflows needing approval, click **"I understand my workflows, go ahead and enable them"**

### Step 8: Run the Backfill

1. In the **Actions** tab, click **"Backfill 2 Years"**
2. Click the **"Run workflow"** dropdown button (on the right)
3. Click the green **"Run workflow"** button
4. Wait 2-5 minutes for it to complete
5. You should see a green checkmark when done ✅

### Step 9: Verify Your Contribution Graph

1. Go to your GitHub profile: `https://github.com/YOUR_USERNAME`
2. Scroll down to see your contribution graph
3. You should now see contributions populated for the past 2 years!

> 🔄 If commits don't appear, see [Troubleshooting](#troubleshooting) section below.

---

## How to Use

### Running Workflows Manually

**Backfill Workflow** (run once):
1. Go to **Actions** → **"Backfill 2 Years"**
2. Click **"Run workflow"**

**Daily Workflow** (test it):
1. Go to **Actions** → **"Daily Weekday Commits"**
2. Click **"Run workflow"**

### Automated Daily Commits

The daily workflow runs automatically:
- **Schedule**: Monday-Friday at 9 AM UTC
- **Action**: Creates 1-3 commits
- **Smart**: Skips if commits already exist for that day

---

## Configuration Options

### Adjusting Commit Frequency

Edit the workflow files to customize:

```yaml
MIN_COMMITS_PER_DAY: 1      # Minimum commits per day
MAX_COMMITS_PER_DAY: 5      # Maximum commits per day
INCLUDE_WEEKENDS: false     # true = include weekends
INCLUDE_WEEKDAYS: true      # false = skip weekdays
```

### Changing Schedule

Modify the cron schedule in `daily-weekday-commits.yml`:

```yaml
schedule:
  - cron: '0 9 * * 1-5'  # Format: 'minute hour day month weekday'
```

**Examples:**
- `'0 12 * * 1-5'` - Weekdays at noon UTC
- `'0 0 * * *'` - Every day at midnight UTC
- `'0 9 * * 1-7'` - Every day at 9 AM UTC

### Backfilling Different Time Periods

Adjust `MAX_DAYS` in the backfill workflow:

```yaml
MAX_DAYS: 730   # 2 years
MAX_DAYS: 365   # 1 year
MAX_DAYS: 90    # 3 months
```

---

## Troubleshooting

### ❌ Commits Not Showing on Contribution Graph

**Check 1: Email Verification**
1. Go to [GitHub Settings → Emails](https://github.com/settings/emails)
2. Verify the email in your workflow file is listed and verified
3. If not verified, click **"Resend verification email"**

**Check 2: Repository is Public**
- Private repositories don't show on your public contribution graph
- Go to **Settings** → **Danger Zone** → Make repository public

**Check 3: Correct Email in Workflows**
- Double-check `GIT_EMAIL` in both `.yml` files matches your GitHub email exactly

**Check 4: Wait 24 Hours**
- GitHub can take up to 24 hours to update contribution graphs
- Force refresh: `https://github.com/YOUR_USERNAME?tab=overview&from=2024-01-01&to=2024-12-31`

### ❌ Workflow Failed

**Error: "Resource not accessible by integration"**
- Go to **Settings** → **Actions** → **General**
- Enable **"Read and write permissions"**
- Re-run the workflow

**Error: "refusing to allow a GitHub App to create or update workflow"**
- This is expected if you're trying to create workflows via Actions
- Workflows must be created locally and pushed via git

### ❌ No Workflows Appearing

- Ensure `.github/workflows/` directory structure is correct
- Workflow files must have `.yml` or `.yaml` extension
- Check for YAML syntax errors (indentation matters!)
- Verify files are committed and pushed to GitHub

### ❌ Daily Automation Not Running

- Check the **Actions** tab for any failed runs
- Verify cron schedule matches your desired time
- Ensure workflow is not disabled (no "Disabled" label)

---

## Technical Details

### How It Works

This setup uses the [bcanseco/github-contribution-graph-action](https://github.com/marketplace/actions/autopopulate-your-contribution-graph) GitHub Action, which:

1. Creates empty commits with backdated timestamps
2. Uses `git commit --date` to set historical dates
3. Pushes commits to your repository
4. GitHub recognizes these commits and updates your contribution graph

### Why This is Safe

- ✅ Only creates empty commits (no code changes)
- ✅ Uses GitHub's official `GITHUB_TOKEN` (no external credentials)
- ✅ Runs in isolated GitHub Actions environment
- ✅ Can be stopped/disabled anytime
- ✅ Repository can be deleted anytime (commits disappear from graph)

### Privacy Considerations

- This creates **public commits** visible to anyone
- Commits will show on your profile as contributions
- No personal data is exposed beyond what's already public

---

## FAQ

### Q: Is this against GitHub's Terms of Service?
**A:** No. You're creating legitimate commits to a repository you own. Many developers use contribution trackers for work done on private repos or other platforms.

### Q: Can I backfill more than 2 years?
**A:** Yes! Change `MAX_DAYS: 730` to any number. For example, `MAX_DAYS: 1095` = 3 years.

### Q: Will this overwrite my existing commits?
**A:** No. This only adds new commits. Your existing commit history is preserved.

### Q: Can I use multiple emails?
**A:** You need separate workflows for each email, each targeting a different time period to avoid conflicts.

### Q: How do I stop the automation?
**A:** Go to **Actions** → **Daily Weekday Commits** → Click **"..."** → **"Disable workflow"**

### Q: Can I delete all the commits?
**A:** Yes. Delete the repository, and all commits will disappear from your contribution graph.

### Q: Does this affect my GitHub statistics (stars, followers, etc.)?
**A:** No. This only affects your contribution graph (the green squares on your profile).

### Q: Can I customize commit messages?
**A:** Yes! Edit `GIT_COMMIT_MESSAGE` in the workflow files. You can use variables like `$(date +%Y-%m-%d)`.

### Q: Will this work for GitHub Enterprise?
**A:** Yes, but you need to ensure Actions are enabled in your enterprise settings.

---

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Cron Schedule Expression](https://crontab.guru/)
- [GitHub Contribution Graph Action](https://github.com/marketplace/actions/autopopulate-your-contribution-graph)

---

## Support

Having issues? Check the [Actions logs](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/using-workflow-run-logs) for detailed error messages.

---

## License

This is free and open-source. Use it however you'd like!

---

**Happy committing! 🚀**
