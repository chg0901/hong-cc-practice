---
name: Windows environment gotchas
description: Three terminals, NO_PROXY, startup lessons, .env + load_dotenv 陷阱
type: feedback
originSessionId: 85b62cf5-e098-4ce2-b282-fb41489b8bb7
---
## Three Terminal Types

| Terminal | Used for | Code block label |
|----------|----------|-----------------|
| **bash (Git Bash)** | Claude Code commands | ` ```bash ` |
| **PowerShell** | Admin scripts (.ps1) | ` ```powershell ` |
| **CMD** | .bat files (InfluxDB) | ` ```cmd ` |

Rules: `.claude/rules/terminal.md` — full syntax differences and Claude Code-specific rules.

## NO_PROXY (mandatory on this machine)

```bash
NO_PROXY=127.0.0.1,localhost python test_codes/test_xxx.py   # Tests
NO_PROXY=gitee.com git push origin main                        # Git
```

Rules: `.claude/rules/testing.md` (tests), `.claude/rules/workflow.md` (git).

## start_project.ps1 Gotchas (2026-04-06)

1. **Do NOT auto-elevate** — `Start-Process -Verb RunAs` triggers UAC popup; use soft warning instead
2. **conda activate fails in spawned CMD** — use `D:\miniconda3\envs\ene\python.exe` directly
3. **localhost resolves to ::1 (IPv6)** on Windows 11 — always use `127.0.0.1` in TcpClient
4. **Use `cmd /k`** for spawned Python processes to keep window open on crash

## .env + load_dotenv 陷阱 (2026-04-16)

**问题**：`os.environ.get("FLASK_PORT")` 返回 `None`，即使 `.env` 文件中已配置 `FLASK_PORT=5000`。

**根因**：`os.environ.get()` 只读**系统环境变量**，不读 `.env` 文件。必须在所有 `os.getenv()` 调用之前执行 `from dotenv import load_dotenv; load_dotenv()`。

**影响**：如果 `load_dotenv()` 放在文件中间或函数内部，它之前的 `os.getenv()` 调用都会返回 `None`。

**解决方案**：在 `smart_energy_platform.py` 顶层、所有 `os.getenv()` 之前添加：
```python
from dotenv import load_dotenv
load_dotenv()
```

**同理适用于 PowerShell**：`.ps1` 脚本中不自动加载 `.env`，需自行解析：
```powershell
function Get-EnvVar($key) {
    $line = Get-Content .env | Select-String "^$key="
    if ($line) { return $line.ToString().Split('=', 2)[1] }
}
```
