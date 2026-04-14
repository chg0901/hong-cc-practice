# Secrets 管理规范

## 原则

所有敏感凭据（数据库密码、API Key、tokens）必须遵循以下规范：

### 1. 存储位置

- **唯一真实来源**：`.env` 文件（已在 `.gitignore` 中，不会提交）
- **模板供参考**：`.env.example`（包含占位符，提交到仓库）
- **禁止位置**：Python 源代码、文档、配置文件、HTML 页面

### 2. 文档中的密码显示

#### 变量赋值（`.env` 配置）

使用**脱敏格式**：前两位字符 + `(还有N位中文数字)`

**格式定义：**
- 数字 → 中文：4→四，5→五，6→六，8→八，10→十，16→十六，30→三十，32→三十二，46→四十六，86→八十六
- 例：`254760`（6位）→ `25(还有四位)`

**应用范围：**
```env
# ✓ 正确 - 文档中展示脱敏值
MARIA_PASS=25(还有四位)
INFLUX_TOKEN=Yc(还有八十六位)
QWEATHER_API_KEY=a7(还有三十位)

# ✗ 错误 - 不要显示完整值
MARIA_PASS=254760
```

#### CLI 命令中的密码

使用**变量替换**，不允许内联明文密码。

**Bash 命令（Git Bash）：**
```bash
# 前置：加载 .env 环境变量
set -a && source .env && set +a

# 使用变量传递密码
mysql -h 127.0.0.1 -P 3307 -u root -p"$MARIA_PASS" -e "SELECT 1"
mysqldump -h 127.0.0.1 -P 3307 -u root -p"$MARIA_PASS" smart_energy > backup.sql
```

**PowerShell 命令：**
```powershell
# 方式1：从 .env 文件读取（推荐）
$mariaPass = (Get-Content .env | Select-String "^MARIA_PASS=(.+)").Matches.Groups[1].Value

& "C:\Program Files\MariaDB 10.1\bin\mysqldump.exe" `
    -h 127.0.0.1 -P 3307 -u root "-p$mariaPass" smart_energy > backup.sql

# 方式2：使用环境变量（需已加载）
& "C:\Program Files\MariaDB 10.1\bin\mysqldump.exe" `
    -h 127.0.0.1 -P 3307 -u root "-p$($env:MARIA_PASS)" smart_energy > backup.sql
```

### 3. Python 代码中的凭据

**规则：**
- 使用 `os.getenv("VAR")` **无默认值**
- 缺失的环境变量应在应用启动时报错（不要 silent fallback）
- 禁止在源代码中硬编码密码或 fallback 值

**✗ 错误示例（不要这样做）：**
```python
# 硬编码默认值会在 git 历史中永久保留
INFLUX_TOKEN = os.getenv("INFLUX_TOKEN", "YcVbYL...M8hg==")
MARIA_PASS = os.getenv("MARIA_PASS", "254760")
```

**✓ 正确示例：**
```python
# 无默认值，缺失时应用启动时报错
INFLUX_TOKEN = os.getenv("INFLUX_TOKEN")
if not INFLUX_TOKEN:
    raise EnvironmentError("INFLUX_TOKEN 未在 .env 中设置")

# 或使用 python-dotenv
from dotenv import load_dotenv
load_dotenv()
INFLUX_TOKEN = os.getenv("INFLUX_TOKEN")
```

### 4. 包含密码的前端代码

**HTML 输入框：**
- ✗ 禁止：`<input type="password" value="admin123">`（预填充密码）
- ✓ 允许：`<input type="password">` 或 `<input type="password" value="admin">`（仅预填用户名）

**JavaScript：**
- 禁止在 JS 代码中存储、打印、日志记录任何密码或 token

### 5. `.env.example` 维护

**规则：**
- 每次添加新的环境变量时，同步更新 `.env.example`
- `.env.example` 中的值使用占位符：`<your_xxx_here>` 或 `<placeholder>`
- 不要在 `.env.example` 中存放真实密码

**检查清单：**
```bash
# 验证 .env 仍在 .gitignore
grep "\.env" .gitignore

# 验证 .env.example 不含真实值
grep -E "[a-f0-9]{8,}|[0-9]{6}" .env.example  # 应无结果（长十六进制或 6 位纯数字）
```

---

## 已知敏感信息清单

| 变量 | 值长度 | 脱敏格式 | 存储位置 |
|------|--------|----------|----------|
| `MARIA_PASS` | 6 位 | `25(还有四位)` | `.env` / 代码中 os.getenv() |
| `INFLUX_TOKEN` | 88 位 | `Yc(还有八十六位)` | `.env` / 代码中 os.getenv() |
| `QWEATHER_API_KEY` | 32 位 | `a7(还有三十位)` | `.env` / 代码中 os.getenv() |
| `ZAI_API_KEY` | 48 位 | `c0(还有四十六位)` | `.env` / 仅文档示例 |
| `FLASK_SECRET_KEY` | 30 位 | `sm(还有二十八位)` | `.env` / 代码中 os.getenv() |
| `admin` 用户密码 | 8 位 | `ad(还有六位)` | SQLite 数据库（平文存储） |

---

## 安全审计

定期执行以下检查确保无密码泄露：

```bash
# 查找所有可能的硬编码密码（grep 黑名单）
grep -r "254760" --include="*.py" --include="*.js" --include="*.html" .
grep -r "YcVbYL" --include="*.py" --include="*.js" --include="*.html" .
grep -r "a7c08eb2" --include="*.py" --include="*.js" --include="*.html" .
grep -r "admin123" --include="*.html" js/  # HTML 中查找

# 查找 CLI 命令中的内联密码
grep -r "\-p[0-9]" docs/ --include="*.md"

# 查找硬编码的 os.getenv() 默认值
grep -r "os.getenv.*," --include="*.py" . | grep -v "^[[:space:]]*#"
```

---

## ChangeLogs

- [2026-04-07 — 初始创建：密码脱敏规范、变量替换模式、.env.example 维护、安全审计清单](changes/2026-04-07)
