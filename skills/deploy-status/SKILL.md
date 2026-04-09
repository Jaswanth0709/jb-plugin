---
name: deploy-status
description: Check Vercel deployment status and fetch build logs on failure
argument-hint: [pr-number]
allowed-tools: [Bash]
---

# Deploy Status — Check Deployment

## Context

- Current branch: !`git branch --show-current`

## Arguments

The user provided: $ARGUMENTS

## Your task

Check the current Vercel deployment status:

### Step 1: Identify the target
- If the user provided a PR number as argument, use that
- Otherwise, use the current branch name

### Step 2: Check status
Run:
```bash
gh pr checks <branch-or-pr-number>
```

### Step 3: Report results

**If all checks pass:**
- Report "All checks passed" with a green checkmark
- Show the PR URL

**If checks are pending:**
- Report which checks are still running
- Suggest the user wait or run `/deploy-status` again shortly

**If any check failed:**
- Show which check(s) failed
- Run `gh pr checks <branch> --json name,state,description,targetUrl` to get the Vercel deployment URL
- Display the Vercel dashboard URL so the user can inspect build logs
- If Vercel MCP is available, try to fetch deployment logs using Vercel MCP tools and show the relevant error
- Suggest what might have gone wrong based on the error details

### Output format
```
Deployment Status: <branch> (PR #<number>)
─────────────────────────────────
  Check Name          Status     
─────────────────────────────────
  Vercel Preview       pass/fail/pending
  Backup Workflow      pass/fail/pending
  ...
─────────────────────────────────

[If failed] Error details: ...
[If failed] Vercel logs: <dashboard-url>
```
