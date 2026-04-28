# GitHub MCP 工作流规范

## 触发场景

当用户进行以下操作时，优先使用 GitHub MCP 而非 `gh` CLI 或网页：

| 场景 | GitHub MCP 工具 | 说明 |
|------|--------------|------|
| 读取仓库文件/目录 | `get_file_contents` | 任意分支/路径直接读 |
| 搜索代码 | `search_code` | GitHub 搜索语法 (language:/repo:/path:) |
| 创建/管理 Issue | `create_issue` / `update_issue` / `list_issues` | 批量操作更高效 |
| PR 创建/审查 | `create_pull_request` / `create_pull_request_review` | 自然语言审查 |
| 调研同类项目 | `search_repositories` + `get_file_contents` | 组合调研工作流 |
| 监控 CI/CD | `list_workflow_runs` / `trigger_workflow` | Actions 工具集 |

## 工作流模板

### W1: 仓库调研工作流

```
1. search_repositories(q="<关键词> stars:>100 language:<语言>")
   → 获取仓库列表（名称/描述/星标数）
2. 对每个感兴趣的仓库：
   get_file_contents(owner="<user>", repo="<repo>", path="README.md")
   → 快速了解项目概述
3. get_file_contents(owner="<user>", repo="<repo>", path="<关键源码路径>")
   → 深入阅读核心代码
4. list_commits(owner="<user>", repo="<repo>", sha="main")
   → 查看提交历史和贡献者
```

### W2: Bug 追踪工作流

```
1. search_issues(q="<关键词> is:issue is:open label:bug repo:<owner>/<repo>")
   → 搜索相关未关闭 Bug
2. get_issue(owner="<owner>", repo="<repo>", issue_number=<N>)
   → 获取 Bug 详情（描述/评论/附件）
3. update_issue(issue_number=<N>, state="closed", labels=["fixed"])
   → 标记为已修复
   或
   create_issue(owner="<owner>", repo="<repo>", title="<新 Issue>", body="<描述>", labels=["bug"])
   → 创建新 Issue
```

### W3: PR 审查工作流

```
1. get_pull_request(owner="<owner>", repo="<repo>", pull_number=<N>)
   → 获取 PR 基本信息（标题/描述/状态）
2. get_pull_request_files(owner="<owner>", repo="<repo>", pull_number=<N>)
   → 获取变更文件列表
3. 对每个变更文件：
   get_file_contents(owner, repo, path, branch=<head>)
   → 阅读变更代码
4. create_pull_request_review(
     owner, repo, pull_number,
     body="<审查意见>",
     event="APPROVE" | "REQUEST_CHANGES" | "COMMENT"
   )
   → 提交审查结果
```

### W4: 仓库备份/同步工作流

```
1. get_file_contents(owner, repo, path="", branch="main")
   → 获取仓库根目录文件列表
2. 对每个需要同步的文件：
   get_file_contents(owner, repo, path=<文件路径>)
   → 读取文件内容
3. create_or_update_file / push_files
   → 写入到目标仓库
4. create_branch / push_files
   → 创建分支并推送变更
```

## 工具选择原则

| 原则 | 说明 |
|------|------|
| 读多写少 | 优先 `get_file_contents` / `search_code` 等只读工具 |
| 幂等操作 | `create_or_update_file` 可安全重跑 |
| 分支隔离 | 写入操作默认在新分支上进行，不污染默认分支 |
| 批量优先 | 多个文件用 `push_files` 而非循环调用 |
| 权限最小 | 仅启用所需的 toolsets，不开全部权限 |

## 与 Git CLI 的边界

| 操作 | 优先工具 | 原因 |
|------|---------|------|
| `git add` / `git commit` | `git` CLI | 成熟的暂存区+提交管理 |
| `git push` / `git pull` | `git` CLI | 分支跟踪+远程协作 |
| 读取任意分支文件 | GitHub MCP | 无需本地 clone |
| 批量文件搜索 | GitHub MCP `search_code` | GitHub 索引更全 |
| PR 创建 | GitHub MCP | 原生 API 支持 |
| 仓库元数据 | GitHub MCP | `list_issues` / `list_commits` |
| Git LFS 操作 | `git` CLI | MCP 文件工具有大小限制 |
| 子模块操作 | `git` CLI | MCP 不支持 |

## 安全约束

- `delete_repository` 工具**禁止使用**（删除操作应通过 GitHub 网页确认）
- `merge_pull_request` 前必须确认 PR 状态和 Review 结果
- `push_files` 操作默认在新分支上进行
- `trigger_workflow` 前确认 Workflow 名称和参数

## ChangeLogs

- [2026-04-21 — Initial: GitHub MCP 工作流规范，4 个工作流模板，工具选择原则，与 Git CLI 边界](changes/2026-04-21)
