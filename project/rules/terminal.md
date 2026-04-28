# Terminal Environment Rules

## 三种终端环境

本项目涉及三种不同的终端环境，**必须在文档和命令中明确区分**：

### 1. bash（Git Bash）— Claude Code 使用的环境

Claude Code 本身运行在 **bash**（Git Bash on Windows）中，使用 Unix 语法：

- 路径用正斜杠：`d:/Proj/energy/`（不用反斜杠）
- 环境变量前缀：`NO_PROXY=... command`
- 不支持 PowerShell cmdlet（`Get-Item`、`Where-Object` 等）
- 不支持 CMD 命令（`dir`、`set VAR=value`、`type` 等）

代码块标注：` ```bash `

```bash
# Claude Code 执行命令示例：
NO_PROXY=127.0.0.1,localhost D:/miniconda3/envs/ene/python.exe test_codes/test_xxx.py
conda activate ene && python smart_energy_platform.py
```

### 2. PowerShell — 项目启动脚本

`.ps1` 脚本文件，用于需要管理员权限的服务启动和数据迁移：

- 项目启动：`start_project.ps1`（需要管理员权限）
- InfluxDB 数据迁移：`scripts/migrate_influxdb_data.ps1`（需要管理员权限）
- conda 激活（PowerShell 会话中）：`conda activate ene; python xxx.py`（用分号，不用 `&&`）

代码块标注：` ```powershell `

```powershell
# PowerShell 示例（管理员权限）：
D:\Proj\energy\start_project.ps1
conda activate ene; python smart_energy_platform.py
cd D:\MariaDB_Data
& "C:\Program Files\MariaDB 10.1\bin\mysqld.exe" --defaults-file="D:\MariaDB_Data\my.ini" --console
```

### 3. CMD（命令提示符）— 批处理脚本

`.bat` 文件，用于 InfluxDB 启动（无需管理员权限）：

- `start_influxdb.bat`：双击运行或在任意终端中调用

代码块标注：` ```cmd `（或 ` ```bat `）

```cmd
REM CMD 示例：
D:\Proj\energy\start_influxdb.bat
```

---

## 规则

### 文档写作规则

1. **代码块必须标注正确的 shell 类型**：`bash`、`powershell`、`cmd` 三者不可混用
2. **PowerShell 的 `&&` 替换为 `;`**：PowerShell 不支持 `&&` 链接命令，用 `;` 或 `&&` 仅在 PowerShell 7+ 有效
3. **路径格式与环境匹配**：
   - bash：`d:/Proj/energy/`（正斜杠）
   - PowerShell/CMD：`D:\Proj\energy\`（反斜杠）
4. **每段代码前注明执行环境**，例如：
   > 以下命令在 **PowerShell（管理员）** 中执行：

### Claude Code 执行规则

1. **Claude Code 只使用 bash 语法**——不生成 PowerShell 或 CMD 命令
2. **NO_PROXY 和测试命令**：see [proxy-rules.md](proxy-rules.md)
3. **启动脚本说明**：告知用户哪个脚本在哪种终端运行，不直接执行 `.ps1`
4. **`conda activate`** 在 bash 中使用 `&&`，在 PowerShell 中使用 `;`
5. **服务重启必须使用 restart_services.ps1**：当需要重启后端服务（如数据库变更、代码更新后）时，告知用户在 PowerShell（管理员）中运行 `.\restart_services.ps1`。脚本包含 SQLite 锁检查、WAL Checkpoint、健康验证等防护步骤，比手动停启更安全。加 `-SkipSqliteCheck` 跳过 SQLite 检查阶段（如仅改前端代码时）
6. **Claude Code 自动调用 restart_services.ps1**：经过测试，bash 环境可以通过 `powershell.exe -ExecutionPolicy Bypass -Command "& 'D:\Proj\energy\restart_services.ps1'"` 直接调用。用户确认后可自动执行，无需手动打开 PowerShell。加 `-SkipSqliteCheck` 跳过 SQLite 阶段（仅前端变更时）

---

## 快速参考表

| 任务 | 终端类型 | 权限要求 | 示例 |
|------|----------|----------|------|
| 运行 Claude Code 命令 | bash (Git Bash) | 普通用户 | `NO_PROXY=... python test.py` |
| 启动 Flask 后端 | bash 或 PowerShell | 普通用户 | `python smart_energy_platform.py` |
| **重启后端服务（推荐）** | **PowerShell（管理员）** | **管理员** | `.\restart_services.ps1` |
| 重启（跳过 SQLite 检查） | PowerShell（管理员） | **管理员** | `.\restart_services.ps1 -SkipSqliteCheck` |
| 启动 InfluxDB | 任意（双击 .bat） | 普通用户 | `start_influxdb.bat` |
| 启动 MariaDB | PowerShell（管理员） | **管理员** | `& "...mysqld.exe" --defaults-file=...` |
| 一键启动全服务 | PowerShell（管理员） | **管理员** | `.\start_project.ps1` |
| 数据迁移 | PowerShell（管理员） | **管理员** | `.\scripts\migrate_influxdb_data.ps1` |
| 运行 Python 测试 | bash (Git Bash) | 普通用户 | See [proxy-rules.md](proxy-rules.md) |

## PowerShell 编码规则（mandatory）

PowerShell 脚本（`.ps1`）**必须遵守以下 UTF-8 编码规则**：

### 1. 禁止在 `.ps1` 中使用中文字符

PowerShell 文件编码不可靠（BOM vs 无 BOM、ANSI vs UTF-8），中文在不同终端/系统可能乱码。所有 `.ps1` 文件中的字符串、注释、Write-Host 输出**只使用 ASCII 英文**。

- 用 `[OK]` 不用 `[成功]`
- 用 `[WARN]` 不用 `[警告]`
- 用 `[FAIL]` 不用 `[失败]`
- 用 `Write-Host "Starting Flask..."` 不用 `Write-Host "启动 Flask..."`

### 2. 在 PowerShell 中执行 Python 多行代码的推荐方式

**问题**：PowerShell 传 `-c` 参数时会剥除双引号，导致 Python 语法错误（`r"path"` 变成 `rpath`）。

**推荐方案：临时 .py 文件 + `sys.argv` 传参**

```powershell
# 1. Helper function: write Python to temp file, exec with arg
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

# 2. Python scripts as single-quoted here-strings (@'...'@) at SCRIPT TOP LEVEL
#    Closing '@ MUST be at column 0 (no indentation). Does NOT expand $vars.
$PY_CHECK_INTEGRITY = @'
import sqlite3, sys
db = sys.argv[1]
conn = sqlite3.connect(db, timeout=5)
r = conn.execute('PRAGMA integrity_check').fetchone()[0]
print(f'OK|{r}')
conn.close()
'@

# 3. Call from functions
function Test-SqliteHealth {
    $result = Invoke-SqlitePython $PY_CHECK_INTEGRITY
    if ($LASTEXITCODE -eq 0 -and $result -match "^OK\|") { ... }
}
```

**Why this works**:
- `@'...'@` (single-quoted here-string) doesn't expand PowerShell variables — no `$` conflicts
- Python code is written to temp file as-is, no quoting mangling
- DB path passed via `sys.argv[1]`, avoiding all PowerShell string escaping issues
- Temp file auto-cleaned in `finally` block

**禁止的方案**：

| Approach | Why it fails |
|----------|-------------|
| `& $PYTHON_EXE -c "code with quotes"` | PS strips double quotes from arguments |
| `@"..."@` inside functions | `"@` must be at column 0, indented = parse error |
| `& $PYTHON_EXE -c 'code with r"path"'` | Single quotes prevent PS expansion but r"path" loses quotes |

### 3. Here-String 规则（mandatory）

PowerShell 的 here-string（`@"..."@` 或 `@'...'@`）有一条**绝对规则**：

> **闭合标记 `"@` 或 `'@` 必须在行首（column 0），前面不能有任何空格或缩进。**

```
# WRONG — indented closing "@ inside a function
function Foo {
    $script = @"
        print('hello')
        "@          # <- PS does NOT recognize this as here-string end!
}

# CORRECT — closing '@ at column 0 (script top level)
$SCRIPT = @'
print('hello')
'@              # <- column 0, recognized correctly
```

| Here-string 类型 | 语法 | 变量展开 | 使用场景 |
|-----------------|------|---------|---------|
| 双引号 | `@"..."@` | **展开** `$var` | 需要 PS 变量替换时 |
| 单引号 | `@'...'@` | **不展开** `$var` | 嵌入 Python/SQL 代码（推荐） |

**本项目规则**：
- 嵌入 Python/SQL 代码 → **始终用 `@'...'@`**（单引号），避免 `$` 被 PS 误展开
- Here-string 变量定义在 **script top level**（不在 function 内部），自然满足 column 0 约束
- 如果必须在函数内嵌入多行文本 → 用 .NET `[System.IO.File]::WriteAllText()` 写文件

### 4. PowerShell 引号陷阱速查

PowerShell 对引号的处理与 bash/cmd 完全不同，是 `.ps1` 脚本最大的坑：

| 陷阱 | 症状 | 规避方法 |
|------|------|---------|
| 外部程序参数剥除双引号 | `python -c "r"path""` → Python 收到 `rpath` | 用临时 .py 文件 + `sys.argv` 传参 |
| Here-string 闭合标记缩进 | `"@` 不被识别，Python 代码被 PS 解析 → parse error | 闭合标记放 column 0，变量放 script top level |
| `@'...'@` vs `@"..."@` | `@"..."@` 展开 `$var`，嵌入 Python 代码时 `$` 被误展开 | Python/SQL 代码始终用 `@'...'@` |
| 中文字符编码不一致 | BOM vs 无 BOM、ANSI vs UTF-8 乱码 | `.ps1` 文件只用 ASCII 英文 |
| PowerShell `&&` 仅 7+ 有效 | PS 5.x 报错 `未识别 '&&'` | 用 `;` 分隔命令 |

### 5. 验证命令

每次修改 `.ps1` 文件后，**必须**运行语法验证：

```powershell
# PowerShell syntax validation (no output = OK)
[System.Management.Automation.Language.Parser]::ParseFile('path\to\script.ps1', [ref]$null, [ref]$null)
```

无错误输出 = 语法正确。

### ChangeLogs

- [2026-04-25] 新增 Claude Code 自动调用 restart_services.ps1 规则（bash→PowerShell 桥接验证通过）
- [2026-04-25] 新增 Here-String 规则（Section 3）+ 引号陷阱速查表（Section 4），编号 3→5
- [2026-04-25] 初始：UTF-8 编码规则（禁止中文、临时 .py 文件模式、验证命令）
