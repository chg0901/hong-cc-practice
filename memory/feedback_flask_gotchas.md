---
name: Flask + Frontend 踩坑集
description: Flask debug reloader 双进程、AJAX 竞态、dict.get(None) 陷阱
type: feedback
originSessionId: 85b62cf5-e098-4ce2-b282-fb41489b8bb7
---
## Flask + Frontend 踩坑集

### 1. Flask debug reloader 双进程陷阱

**问题**：Flask debug 模式的 reloader 会生成 2 个进程（parent watcher + child app），都执行 `__main__`，导致调度线程/定时器重复创建。

**解决方案**：使用 `WERKZEUG_RUN_MAIN` 环境变量检测：

```python
if app.debug and os.environ.get('WERKZEUG_RUN_MAIN') != 'true':
    return  # Skip scheduler init in parent process
```

**Why**：`WERKZEUG_RUN_MAIN` 在子进程中为 `'true'`，在父进程中不存在（不是 `'false'`）。只有子进程应该初始化调度器。

### 2. AJAX 竞态双触发

**问题**：下拉框同时绑定了内联 `onchange` 和回调自动查询，触发两次 `loadWeatherDataList()`，两个并发请求的响应顺序不确定导致数据闪烁。

**解决方案**：
1. 移除内联 `onchange`，只保留回调绑定
2. 发新请求前 `abort` 旧请求
3. 下拉重建时临时 `off('change')`，重建后 `on('change')` 重新绑定

### 3. dict.get() None 值陷阱

**问题**：当 key 存在但值为 `None` 时，`d.get("chart", "")` 返回 `None` 而非 `""`。

**解决方案**：`(d.get("chart") or "")` 确保 None 也被替换为默认值。

### 4. WAL "disk full" 假阳性

**问题**：WAL 模式下 SQLite 将写操作追加到 WAL 文件而非主库。若 WAL 文件增长到数百 MB 且 checkpoint 因活跃读者被阻塞，SQLite 报 `database or disk is full`，即使磁盘空间充足。

**Why**：WAL checkpoint 需要无活跃读者时才能将 WAL 内容合并回主库。长时间运行的查询或 RealtimeSimulator 的频繁写入会阻止 checkpoint。

**解决方案**：后台线程定期执行 `PRAGMA wal_checkpoint(PASSIVE)`（每 30 分钟），防止 WAL 文件无限增长。

**How to apply**：任何使用 SQLite WAL 模式且长时间运行的 Python 服务都应添加此机制。

### 5. 全局 JS 函数命名冲突

**问题**：本项目 JS 模块以全局脚本方式加载（非 ES module），两个文件定义同名全局函数时后加载的覆盖先加载的。`control.js` 的 `renderLayoutDataToSVG` 覆盖了 `3d.js` 的同名函数，导致 3D 组态视图失效。

**Why**：传统 `<script>` 标签加载的 JS 文件共享全局命名空间，没有模块隔离。

**解决方案**：给函数名加模块前缀（如 `renderControlLayoutDataToSVG`），或改用 ES module（`import/export`）。

**How to apply**：在本项目中新增全局函数时，必须 grep 确认无同名函数冲突：`grep -rn "function XXX" js/modules/`

### 6. CSS show/hide vs AJAX re-render 做筛选

**问题**：下拉筛选触发 AJAX 重新获取数据并重新渲染整个组态图，导致页面崩溃（过滤后的设备列表与保存布局中的设备 ID 不匹配）。

**Why**：筛选是纯展示层需求，不应该改变底层数据。AJAX re-render 会触发完整的 DOM 重建，可能与缓存的布局数据产生不一致。

**解决方案**：筛选用纯 CSS `show()/hide()` 控制 DOM 可见性，零网络请求、瞬间响应、不影响布局。

**How to apply**：任何"显示/隐藏"类筛选（设备类型、状态、分组）都应优先用 CSS 而非 AJAX。只有"数据变更"类操作（增删改、排序）才需要 AJAX。

### 7. `in` vs `startswith` 前缀匹配陷阱

**问题**：`if 'P_' in 'soil_temp_10cm'` 为 True（`P_` 出现在 `soil_tem` 中），但 `'soil_temp_10cm'.startswith('P_')` 为 False。用 `in` 做前缀匹配是经典陷阱。

**Why**：Python 的 `in` 操作符搜索子串出现，不是前缀匹配。当匹配目标包含连续子串时会产生误匹配。

**解决方案**：前缀匹配必须用 `str.startswith()`，后缀匹配用 `str.endswith()`。

**How to apply**：`fault_detector.py` 等基于前缀的设备类型匹配场景。

### 8. MCP Server type 必须显式声明

**问题**：`~/.claude.json` 中 `jina-mcp-server` 配置只有 `url` + `headers`，缺少 `"type": "http"`，导致 `/doctor` schema 校验失败。

**Why**：stdio 类型有 `command` 字段可推断类型，但 HTTP 类型（`url` + `headers`）没有命令，Claude Code 无法推断，必须显式声明。

**解决方案**：HTTP 类型的 MCP Server 必须加 `"type": "http"`。

**How to apply**：每次安装新的远程/HTTP MCP Server 时，确保配置中包含 `type` 字段。

### 9. 数量型 vs 时间型备份保留

**问题**：时间型保留（如"保留 3 天"）允许无上限累积。数量型保留（如"保留 5 个最新"）保证磁盘空间可控。

**建议**：优先使用数量型保留：`all_backups = sorted(..., key=os.path.getmtime); to_delete = all_backups[:-MAX_COUNT]`
