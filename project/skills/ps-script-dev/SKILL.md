---
name: ps-script-dev
description: PowerShell script development workflow — create, edit, validate .ps1 files with pitfall prevention
user-invocable: true
---

# PowerShell Script Development Skill

Create or edit `.ps1` scripts for this project, following all pitfall prevention rules.

## Trigger

- User asks to create or modify a `.ps1` file
- User mentions "PowerShell script" or ".ps1"
- `/ps-script-dev`

## Pre-flight Checklist

Before writing any `.ps1` code, verify these constraints:

### 1. ASCII English Only
- ALL strings, comments, `Write-Host` output in `.ps1` files use **ASCII English**
- No Chinese/Japanese/Korean characters anywhere in the file
- Use `[OK]` not `[成功]`, `[WARN]` not `[警告]`, `[FAIL]` not `[失败]`

### 2. Here-String Rules
- Closing `"@` or `'@` **MUST be at column 0** (no indentation)
- Embed Python/SQL code → use `@'...'@` (single-quoted, no `$var` expansion)
- Define here-string variables at **script top level** (not inside functions)

### 3. Python Execution Pattern
- **NEVER** use `python -c "multi-line code"` — PS strips double quotes
- **ALWAYS** use temp `.py` file + `sys.argv` pattern:
  1. Define Python as `$PY_SCRIPT = @'...'@` at script top level
  2. Write to temp file via `Invoke-SqlitePython` helper
  3. Pass paths via `sys.argv[1]`, not string interpolation
  4. Clean up temp file in `finally` block

### 4. Command Chaining
- Use `;` not `&&` for PS 5.x compatibility
- `conda activate ene; python xxx.py` (not `conda activate ene && python xxx.py`)

## Script Template

```powershell
<#
.SYNOPSIS
    Brief description in English
.DESCRIPTION
    Detailed description in English
.NOTES
    Run:  .\script_name.ps1
          .\script_name.ps1 -OptionalFlag
#>

param(
    [switch]$OptionalFlag
)

$ErrorActionPreference = "Continue"
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$OutputEncoding = [System.Text.Encoding]::UTF8

# --- Config ---
$PROJECT_DIR = "D:\Proj\energy"
$PYTHON_EXE  = "D:\miniconda3\envs\ene\python.exe"

# --- Helper functions ---
function Invoke-SqlitePython($pyScript) {
    $tmpFile = Join-Path $env:TEMP "energy_sqlite_$(Get-Random).py"
    try {
        [System.IO.File]::WriteAllText($tmpFile, $pyScript, (New-Object System.Text.UTF8Encoding $false))
        $output = & $PYTHON_EXE $tmpFile $DB_PATH 2>&1
        return "$output"
    } finally {
        Remove-Item $tmpFile -Force -ErrorAction SilentlyContinue
    }
}

# --- Python scripts (single-quoted here-strings at TOP LEVEL) ---
# Closing '@ MUST be at column 0

$PY_EXAMPLE = @'
import sqlite3, sys
db = sys.argv[1]
conn = sqlite3.connect(db, timeout=5)
r = conn.execute('PRAGMA integrity_check').fetchone()[0]
print(f'OK|{r}')
conn.close()
'@

# --- Main logic ---
Write-Host "[OK] Script started" -ForegroundColor Green
```

## Post-edit Validation

After any `.ps1` file modification, run these checks:

### Step 1: Syntax Validation (mandatory)

```powershell
# No output = syntax OK
[System.Management.Automation.Language.Parser]::ParseFile('path\to\script.ps1', [ref]$null, [ref]$null)
```

### Step 2: Content Scan (mandatory)

```bash
# Check for Chinese characters (should return nothing)
grep -Pn '[\x{4e00}-\x{9fff}]' path/to/script.ps1

# Check for && (should return nothing — use ; instead)
grep -n '&&' path/to/script.ps1

# Check for python -c (should return nothing — use temp file pattern)
grep -n '\-c\s*["\x27]' path/to/script.ps1
```

### Step 3: Here-string Audit (mandatory)

```bash
# Verify all closing "@ or '@ are at column 0
grep -Pn '^\s+["\x27]@' path/to/script.ps1
# Should return NOTHING — if matches found, fix indentation
```

## Common Patterns

### Read .env in PowerShell

```powershell
function Get-EnvVar($key, $default) {
    $envFile = Join-Path $PROJECT_DIR ".env"
    if (Test-Path $envFile) {
        $line = Get-Content $envFile |
            Where-Object { $_ -match "^\s*$key\s*=\s*(.+)" } |
            Select-Object -First 1
        if ($line -match "=\s*(.+)") {
            return $Matches[1].Split('#')[0].Trim().Trim('"').Trim("'")
        }
    }
    return $default
}
```

### Port Check Functions

```powershell
function Test-Port($port) {
    try {
        $tcp = New-Object System.Net.Sockets.TcpClient("127.0.0.1", $port)
        $tcp.Close()
        return $true
    } catch { return $false }
}

function Wait-Port($port, $name, [int]$timeoutSec = 30) {
    for ($i = 0; $i -lt $timeoutSec; $i++) {
        if (Test-Port $port) { return $true }
        Start-Sleep -Seconds 1
    }
    return $false
}
```

### Stop Process by Command Line

```powershell
function Stop-ProcessByCmdline($pattern) {
    $procs = Get-CimInstance Win32_Process -Filter "Name = 'python.exe'" |
        Where-Object { $_.CommandLine -like "*$pattern*" }
    if (-not $procs) { return 0 }
    foreach ($p in $procs) {
        Stop-Process -Id $p.ProcessId -Force -ErrorAction SilentlyContinue
    }
    return ($procs | Measure-Object).Count
}
```

## Project .ps1 Files Reference

| File | Purpose | Admin Required |
|------|---------|---------------|
| `restart_services.ps1` | Restart Flask + Docs with SQLite lock protection | No |
| `start_project.ps1` | Start all services (Flask + InfluxDB + MariaDB) | Yes |
| `scripts/migrate_influxdb_data.ps1` | InfluxDB data migration | Yes |

## Related Rules

- [terminal.md](.claude/rules/terminal.md) — Full terminal environment rules and PS pitfall reference
