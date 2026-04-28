# File Migration Rules (C → D Drive)

## When to Apply

Any time MCP server caches, browser binaries, or tool-related files are moved from C drive to D drive (or any path change), follow this checklist strictly.

## Migration Checklist

### 1. Pre-Migration Verification

Before moving ANY files:

```bash
# a. Verify source directory exists and get size
du -sh /path/to/source/

# b. Verify destination directory exists (create if needed)
mkdir -p /path/to/destination/

# c. Check available disk space on destination
df -h /path/to/destination/
```

### 2. Move Files

```bash
# Move (not copy) to preserve disk space
mv /path/to/source/ /path/to/destination/

# Verify move succeeded — source gone, destination present
ls /path/to/source/ 2>/dev/null && echo "FAIL: source still exists" || echo "OK: source removed"
ls /path/to/destination/ >/dev/null && echo "OK: destination exists" || echo "FAIL: destination missing"
```

### 3. Check Environment Variables

After moving, check ALL relevant environment variables for old paths:

```bash
# Check system/user env vars
env | grep -i -E "cache|puppeteer|playwright|npm|uv"

# Check npm config
npm config get cache

# Check uv config (if any)
uv cache dir 2>/dev/null
```

### 4. Update Configuration Files

For each moved path, update ALL config files that reference it:

| Config File | What to Check |
|-------------|---------------|
| `~/.claude.json` → `mcpServers` | `env` paths (PUPPETEER_EXECUTABLE_PATH, etc.) |
| `~/.claude/settings.json` → `env` | Cache dir env vars (UV_CACHE_DIR, PLAYWRIGHT_BROWSERS_PATH, etc.) |
| `~/.claude/settings.json` → `permissions.additionalDirectories` | Playwright browser paths |
| `~/.npmrc` or `npm config get cache` | npm cache location |
| `~/.env` (project) | Any tool-specific paths |
| Project `docs/mcp-configuration.md` | Documented path examples |

**Update sequence**: Config files FIRST, then env vars, then test.

### 5. Set Environment Variables

Add to `settings.json` → `env` section:

```json
{
  "env": {
    "UV_CACHE_DIR": "D:\\claude_code_mcp\\uv-cache",
    "PUPPETEER_CACHE_DIR": "D:\\claude_code_mcp\\puppeteer-cache",
    "PLAYWRIGHT_BROWSERS_PATH": "D:\\claude_code_mcp\\playwright",
    "NO_PROXY": "api.context7.com,github.com,localhost,127.0.0.1"
  }
}
```

Update npm cache: `npm config set cache D:\claude_code_mcp\npm-cache`

### 6. Post-Migration Testing

After ALL config updates, test EACH affected MCP server:

```bash
# Test MiniMax (stdio MCP — depends on npm/npx cache)
# Just verify it responds in next Claude Code session

# Test Mermaid local (depends on Puppeteer Chrome path)
# Verify PUPPETEER_EXECUTABLE_PATH points to existing chrome.exe
ls "D:\claude_code_mcp\puppeteer-chrome\chrome-win64\chrome.exe"
```

In Claude Code: invoke each MCP tool once to confirm connectivity.

### 7. Cleanup

- Remove old empty directories on C drive (only if move confirmed successful)
- Do NOT remove directories that still contain files (other apps may use them)

### 8. Documentation

Record in work_summary:
- Source path → Destination path
- Files moved and total size
- Config files updated
- Environment variables changed
- Test results (pass/fail for each affected MCP)

## Current D Drive Cache Layout

```
D:\claude_code_mcp\
├── npm-cache\          (1.8G) — npm packages cache
├── playwright\         (31M)  — Playwright browser binaries
│   └── mcp-chrome-e3e3a12/
├── puppeteer-cache\    (909M) — Puppeteer downloaded browsers
├── puppeteer-chrome\   (409M) — Chrome for Mermaid MCP
│   └── chrome-win64\chrome.exe
└── uv-cache\           (1.7G) — uv/uvx Python package cache
```

## Environment Variables Summary

| Variable | Value | Set In |
|----------|-------|--------|
| `UV_CACHE_DIR` | `D:\claude_code_mcp\uv-cache` | settings.json → env |
| `PUPPETEER_CACHE_DIR` | `D:\claude_code_mcp\puppeteer-cache` | settings.json → env |
| `PLAYWRIGHT_BROWSERS_PATH` | `D:\claude_code_mcp\playwright` | settings.json → env |
| `PUPPETEER_EXECUTABLE_PATH` | `D:\claude_code_mcp\puppeteer-chrome\chrome-win64\chrome.exe` | .claude.json → mermaid-local env |
| npm cache | `D:\claude_code_mcp\npm-cache` | npm config |
| `NO_PROXY` | `api.context7.com,github.com,localhost,127.0.0.1` | settings.json → env |

## ChangeLogs

- [2026-04-13 20:56:00] NO_PROXY env var reference now points to proxy-rules.md
- [2026-04-09 — Initial: migration checklist, current cache layout, env var summary](changes/2026-04-09)
