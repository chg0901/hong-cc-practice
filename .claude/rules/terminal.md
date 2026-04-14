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

---

## 快速参考表

| 任务 | 终端类型 | 权限要求 | 示例 |
|------|----------|----------|------|
| 运行 Claude Code 命令 | bash (Git Bash) | 普通用户 | `NO_PROXY=... python test.py` |
| 启动 Flask 后端 | bash 或 PowerShell | 普通用户 | `python smart_energy_platform.py` |
| 启动 InfluxDB | 任意（双击 .bat） | 普通用户 | `start_influxdb.bat` |
| 启动 MariaDB | PowerShell（管理员） | **管理员** | `& "...mysqld.exe" --defaults-file=...` |
| 一键启动全服务 | PowerShell（管理员） | **管理员** | `.\start_project.ps1` |
| 数据迁移 | PowerShell（管理员） | **管理员** | `.\scripts\migrate_influxdb_data.ps1` |
| 运行 Python 测试 | bash (Git Bash) | 普通用户 | See [proxy-rules.md](proxy-rules.md) |
