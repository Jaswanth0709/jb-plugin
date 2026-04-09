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
- Vercel connected: !`gh api repos/$(gh repo view --json owner,name -q '.owner.login + "/" + .name')/deployments --jq 'length' 2>/dev/null || echo "0"`

## Your task

Based on the above context, execute the full deployment pipeline:

### Step 1: Commit
- If there are staged or unstaged changes, stage all relevant files (avoid .env or credential files)
- Create a commit with a concise, descriptive message based on the diff
- If there are no changes to commit, skip to Step 2

### Step 2: Push
- Push to the current branch with `-u` flag: `git push -u origin <branch>`
- If already up to date, continue to Step 3

### Step 3: Monitor Preview Deployment (only if Vercel is configured)
- Check the "Vercel connected" context above — if the deployment count is "0" or the check failed, **skip this step** and go directly to Step 4
- If Vercel IS configured:
  - Spawn a background agent using the `deployment-monitor` agent type
  - Pass it the branch name so it can monitor the Vercel preview deployment
  - Tell the agent: "Monitor the Vercel preview deployment for branch '<branch>'. Poll every 30 seconds using `gh api repos/{owner}/{repo}/deployments` or check Vercel deployment status. If deployment fails, fetch build logs, fix the code, commit, push, and retry (max 3 attempts). If deployment succeeds, extract the Vercel preview URL and share it with the user. Ask them to test the preview to verify their changes work correctly."
  - **STOP HERE and wait** for the background agent to complete and for the user to test the preview
  - Only proceed to Step 4 after user confirms changes look good

### Step 4: Ask if ready for production deployment
- If Vercel was configured, this step runs ONLY after the user has tested the preview and confirmed it looks good
- If Vercel was NOT configured, proceed immediately
- Ask the user: "Ready for production deployment? Let me know and I'll create the PR."
- **Wait for the user's response. Do NOT proceed until they say yes.**

### Step 5: Create PR
- Only proceed here after the user confirmed yes in Step 4
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
- Then ask: "PR created! Do you want to review and merge it yourself, or should I auto-merge it?"
- **Wait for the user's response before proceeding**

### Step 6: Merge (only if user asked for auto-merge)
- Only proceed here if the user explicitly asked you to auto-merge
- Merge the PR:
  ```bash
  gh pr merge <pr-number> --squash --delete-branch
  ```
- If Vercel is configured, spawn a background agent using the `deployment-monitor` agent type to monitor the production deployment on `main`
- Tell the agent: "Monitor the Vercel production deployment for branch 'main' after PR merge. Poll every 30 seconds. If deployment succeeds, open the production URL in the browser and report success. If deployment fails, report the error details to the user."
- If user said they'll review it themselves, just say: "PR is ready for your review." and stop

### Important
- Ask ONE question at a time — never combine multiple questions in one message
- Do NOT create a PR until the user confirms they are ready
- Do NOT auto-merge unless the user explicitly agrees
- Do NOT commit .env files, credentials, or secrets
- If on the `main` branch, warn the user and ask them to switch to a feature branch first — do NOT proceed
