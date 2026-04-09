---
name: ship
description: Commit, push, create PR, and monitor Vercel deployment with auto-fix on failure
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

### Step 3: Create PR (if needed)
- Check the "Existing PRs" context above
- If a PR already exists for this branch, skip PR creation and note the existing PR URL
- If no PR exists, create one:
  ```
  gh pr create --title "<concise title from commits>" --body "$(cat <<'EOF'
  ## Summary
  <bullet points summarizing changes>

  ## Test plan
  - [ ] Vercel preview deployment passes
  - [ ] Manual verification of changes

  Generated with jb-plugin /ship
  EOF
  )"
  ```
- Output the PR URL

### Step 4: Monitor Deployment
- Spawn a background agent using the `deployment-monitor` agent type
- Pass it the branch name and PR URL so it can monitor the Vercel deployment
- Tell the agent: "Monitor the Vercel deployment for branch '<branch>' (PR: <url>). Poll `gh pr checks` every 30 seconds. If deployment fails, fetch Vercel build logs, fix the code, commit, push, and retry (max 3 attempts). If all checks pass, notify that the PR is ready to merge."

### Important
- You MUST do Steps 1-3 in a single message using parallel tool calls where possible
- Do NOT commit .env files, credentials, or secrets
- If on the `main` branch, warn the user and ask them to switch to a feature branch first — do NOT proceed
