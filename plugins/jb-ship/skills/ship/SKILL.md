---
name: ship
description: Commit, push, monitor Vercel preview deployment, and create PR only after user verifies
allowed-tools: [Bash, Agent]
---

# Ship — Full Deployment Pipeline

## Context

- Current git status: !`git status`
- Current git diff (staged and unstaged changes): !`git diff HEAD`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -10`
- Existing PRs for this branch: !`gh pr list --head $(git branch --show-current) --json number,title,url 2>/dev/null || echo "none"`

## Your task

Based on the above context, execute the full deployment pipeline:

### Step 1: Commit
- If there are staged or unstaged changes, stage all relevant files (avoid .env or credential files)
- Create a commit with a concise, descriptive message based on the diff
- If there are no changes to commit, skip to Step 2

### Step 2: Push
- Push to the current branch with `-u` flag: `git push -u origin <branch>`
- If already up to date, continue to Step 3

### Step 3: Monitor Preview Deployment
- Spawn a background agent using the `deployment-monitor` agent type
- Pass it the branch name so it can monitor the Vercel preview deployment
- Tell the agent: "Monitor the Vercel preview deployment for branch '<branch>'. Poll every 30 seconds using `gh api repos/{owner}/{repo}/deployments` or check Vercel deployment status. If deployment fails, fetch build logs, fix the code, commit, push, and retry (max 3 attempts). If deployment succeeds, extract the Vercel preview URL and share it with the user. Ask them to test the preview to verify their changes work correctly."
- **STOP HERE and wait** for the background agent to complete and for the user to test the preview

### Step 4: Create PR (only after user confirms changes look good)
- This step runs ONLY after the user has tested the preview and confirmed it looks good
- Check the "Existing PRs" context above
- If a PR already exists for this branch, skip PR creation and note the existing PR URL
- If no PR exists, create one:
  ```
  gh pr create --title "<concise title from commits>" --body "$(cat <<'EOF'
  ## Summary
  <bullet points summarizing changes>

  ## Test plan
  - [x] Vercel preview deployment passes
  - [x] Manual verification of changes on preview

  Generated with jb-plugin /ship
  EOF
  )"
  ```
- Output the PR URL

### Important
- Do NOT create a PR until the user has tested the preview and confirmed it's good
- Do NOT commit .env files, credentials, or secrets
- If on the `main` branch, warn the user and ask them to switch to a feature branch first — do NOT proceed
