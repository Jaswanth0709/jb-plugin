# jb-plugin

One-command deployment automation for Claude Code. Commit, push, create PR, monitor Vercel deployment, and auto-fix build failures — all from a single `/ship` command.

## Commands

| Command | Description |
|---------|-------------|
| `/ship` | Full pipeline: commit -> push -> PR -> monitor deployment -> auto-fix failures |
| `/deploy-status [pr#]` | Check current deployment status and show errors |

## How it works

```
/ship
  |
  |-- 1. Stage & commit changes
  |-- 2. Push to feature branch
  |-- 3. Create PR to main (if needed)
  +-- 4. Spawn background monitor
          |
          |-- Poll deployment status (every 30s, up to 10 min)
          |
          |-- On success -> "Ready to merge!"
          |
          +-- On failure (up to 3 retries) ->
                |-- Fetch Vercel build logs
                |-- Analyze error
                |-- Fix code
                |-- Commit & push fix
                +-- Resume polling
```

## Setup

### 1. Install the plugin

```bash
# In Claude Code, from any project:
/plugin install /Users/mj/Desktop/jb-plugin
```

### 2. Add Vercel MCP (once per project)

```bash
claude mcp add --transport http vercel https://mcp.vercel.com
```

Then authenticate by typing `/mcp` in Claude Code.

### 3. Prerequisites

- **gh CLI**: Must be installed and authenticated (`gh auth login`)
- **Vercel**: Project must be connected to Vercel via GitHub integration

## Usage

```bash
# Make your code changes, then:
/ship

# Check deployment status anytime:
/deploy-status
/deploy-status 42    # check specific PR
```

## What gets auto-fixed

The deployment monitor can automatically fix:
- TypeScript compilation errors
- ESLint errors
- Import/export issues
- Missing type annotations
- Build configuration issues

It will NOT attempt to fix:
- Environment variable issues
- External service failures
- Vercel platform issues
- Dependency conflicts requiring manual resolution
