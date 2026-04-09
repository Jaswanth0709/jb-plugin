# JB Plugin Marketplace

A custom Claude Code plugin marketplace.

## Plugins

| Plugin | Description |
|--------|-------------|
| **jb-ship** | One-command deployment: commit, push, PR, monitor Vercel, auto-fix build failures |

## Installation

### 1. Add this marketplace (once)

In Claude Code, run:

```
/plugin marketplace add jb-marketplace --source github --repo Jaswanth0709/jb-plugin
```

### 2. Install a plugin

```
/plugin install jb-ship@jb-marketplace
```

### 3. Setup Vercel MCP (once per project)

```bash
claude mcp add --transport http vercel https://mcp.vercel.com
```

Then authenticate by typing `/mcp` in Claude Code.

### Prerequisites

- **gh CLI**: Must be installed and authenticated (`gh auth login`)
- **Vercel**: Project must be connected to Vercel via GitHub integration

## Usage

```
# Full pipeline: commit -> push -> PR -> monitor -> auto-fix
/ship

# Check deployment status anytime
/deploy-status
/deploy-status 42    # check specific PR
```

## Plugin: jb-ship

### How it works

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

### What gets auto-fixed

- TypeScript compilation errors
- ESLint errors
- Import/export issues
- Missing type annotations
- Build configuration issues

### What it will NOT fix

- Environment variable issues
- External service failures
- Vercel platform issues
- Dependency conflicts requiring manual resolution
