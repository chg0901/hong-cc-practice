---
name: Password masking preference for documentation
description: How to display sensitive credentials in docs - mask with first 2 chars + remaining digit count in Chinese
type: feedback
---

## Password Masking Format

When displaying passwords/API keys/secrets in documentation, use the following masking format:

**First 2 characters + "(还有N位中文数字)"**

### Format rules

- Show: First 2 characters of the secret
- Hide: Remaining characters (count them)
- Count: Write remaining count in Chinese numerals
  - 4 → 四，5 → 五，6 → 六，8 → 八，10 → 十，16 → 十六，30 → 三十，32 → 三十二，46 → 四十六，88 → 八十八

### Examples

| Secret | Original | Masked Format |
|--------|----------|---------------|
| MariaDB password | `254760` (6 chars) | `25(还有四位)` |
| Admin password | `admin123` (8 chars) | `ad(还有六位)` |
| QWeather API Key | `a7c08eb2...a617` (32 chars) | `a7(还有三十位)` |
| InfluxDB Token | `YcVbYL...M8hg==` (88 chars) | `Yc(还有八十六位)` |
| ZAI API Key | `c0d692a8...NeTA` (48 chars) | `c0(还有四十六位)` |

### When to use this format

- ✓ Documenting `.env` variable values: `MARIA_PASS=25(还有四位)`
- ✓ Explaining configuration in markdown docs
- ✓ Adding API key references in comments

### When NOT to use this format

- ✗ Skip in `.env` file itself (it's already gitignored, keep real values)
- ✗ Skip in CLI commands (use variable substitution instead: `-p"$MARIA_PASS"`)
- ✗ Skip in work_summary historical docs (can keep as-is for context)

## Why this approach

**Why it works better than just `***`:**
- Developers can verify they're using the correct password
  - `ad(还有六位)` confirms it's the 8-character admin password, not a different one
  - First 2 chars help identify which secret without exposing all
- Still safe from accidental commits/logs
- Single glance recognition without needing to count

**Why maintain in `.env` only:**
- True single source of truth
- `.env` is in `.gitignore` so no history leakage
- Code uses `os.getenv()` without fallback values
- `.env.example` provides template for developers to copy and fill in

## Related rules

- Full guidelines: [.claude/rules/secrets.md](../../.claude/rules/secrets.md)
- DB rules: [.claude/rules/database.md](../../.claude/rules/database.md#sqlite-操作规则)
