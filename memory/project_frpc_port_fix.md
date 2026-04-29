---
name: frpc external access port hardcode fix
description: getApiBaseUrl() in js/main.js dropped port when hostname != localhost. Fixed 2026-04-28 by using window.location.port.
type: project
originSessionId: 17bc17d3-d2b0-4e95-9719-7d38809aa277
---
# 外网访问登录失败：端口丢失 Bug（2026-04-28 修复）

**事实**：`js/main.js` 的 `getApiBaseUrl()` 函数在非 localhost 环境下丢失端口号，导致外网通过 frpc 访问时 AJAX 请求发到错误端口（80）。

**Why**: frpc 映射：本地 5000→外网 43000，本地 5002→外网 43001。前端从 `http://47.101.204.138:43000` 访问时，`window.location.hostname` 返回 `47.101.204.138`（不含端口），旧代码走 else 分支只返回 `http://47.101.204.138`（缺少 `:43000`），所有 AJAX 请求发到 80 端口失败。

**修复**（`js/main.js` 约 line 43-47）：
```javascript
// 修复前（BUG）：
} else {
    return `${protocol}//${hostname}`;  // 丢失端口
}

// 修复后：
} else {
    const port = window.location.port;
    return `${protocol}//${hostname}${port ? ':' + port : ''}`;
}
```

**frpc 端口映射**：
- 外网 43000 → 本地 5000（Flask 主应用）
- 外网 43001 → 本地 5002（docs_server.py 反向代理）

**How to apply**: 如果再次出现外网访问 API 失败，先检查 `getApiBaseUrl()` 是否正确携带端口。`docs_server.py` 中的 `127.0.0.1:5000` 硬编码是正确的（服务端内部通信）。
