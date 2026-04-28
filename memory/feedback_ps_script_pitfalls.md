---
name: PS script pitfalls — here-string + quote stripping + encoding
description: Three critical PowerShell script writing pitfalls discovered in restart_services.ps1 development — here-string indentation, quote stripping, and UTF-8 encoding
type: feedback
originSessionId: 3b3201af-9f26-4b79-80df-584185f81759
---
Three pitfalls when writing `.ps1` files for this project:

1. **Here-string closing `"@` or `'@` must be at column 0 (no indentation)**
   - **Why**: PowerShell does NOT recognize an indented closing tag as the end of a here-string. The entire block gets parsed as PS code → parse errors on Python/SQL syntax.
   - **How to apply**: Define here-string variables at script top level (not inside functions) where column 0 is natural. For `@'...'@` (single-quoted), this also prevents `$var` expansion — perfect for embedding Python code.

2. **PowerShell strips double quotes from external program arguments**
   - **Why**: `& python.exe -c "code with r"path""` — PS removes the inner `"` around `path`, so Python receives `rpath` instead of `r"path"` → SyntaxError.
   - **How to apply**: NEVER use `python -c "..."` in PS scripts. ALWAYS use the temp `.py` file pattern: `Invoke-SqlitePython` helper writes code to temp file, passes paths via `sys.argv[1]`, cleans up in `finally`.

3. **PowerShell file encoding is unreliable for non-ASCII characters**
   - **Why**: PS 5.x encoding varies (BOM vs no-BOM, ANSI vs UTF-8) across terminals and systems. Chinese characters may garble.
   - **How to apply**: `.ps1` files use ASCII English only. `[OK]` not `[成功]`, `Write-Host "Starting..."` not `Write-Host "启动..."`.

Full rules: `.claude/rules/terminal.md` Section 3-5. Skill: `/ps-script-dev`.
