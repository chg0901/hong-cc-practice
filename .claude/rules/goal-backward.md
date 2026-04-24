# Goal-Backward Methodology — 需求倒推规则

## 来源

提炼自 [get-shit-done](https://github.com/gsd-build/get-shit-done) 的 Goal-Backward Methodology。

## 核心方法

从"最终必须为真"的断言出发，倒推实现步骤。与传统 "正向规划" 相反：

| 方法 | 起点 | 方向 | 适用场景 |
|------|------|------|----------|
| 正向规划 | 需求描述 → 设计 → 实现 | 前→后 | 探索性开发 |
| Goal-Backward | 最终断言 → 倒推前提 → 最小实现 | 后→前 | API 端点、数据迁移、功能验证 |

## 使用方式

在 plan mode 中，当需要 Goal-Backward 时，使用以下模板：

```markdown
## Goal-Backward Plan: <feature-name>

### 必须为真（Goal Assertions）
- [ ] <断言 1: 具体的可验证条件>
- [ ] <断言 2: 具体的可验证条件>
- [ ] <断言 3: 具体的可验证条件>

### 倒推实现步骤
1. 要让断言 N 成立 → 需要 X → 检查/实现 X
2. 要让断言 N-1 成立 → 需要 Y → 检查/实现 Y
3. ...
4. 要让断言 1 成立 → 需要 Z → 检查/实现 Z

### 验证
每个断言必须有对应的验证命令或测试用例。
```

## 断言示例

| 场景 | 断言 | 验证 |
|------|------|------|
| API 端点 | `GET /api/xxx` 返回 200 + 正确 JSON 结构 | `curl + jq` 或测试脚本 |
| 数据迁移 | 新增的 company_id 列非空且有外键 | `PRAGMA table_info` + 查询 |
| 软删除 | `status=0` 的记录不出现在活跃列表 | API 调用验证 |
| 权限 | viewer 角色无法执行控制操作 | 测试脚本模拟 |
| 性能 | 查询响应时间 < 100ms | 计时测试 |

## 适用场景

| 适用 | 不适用 |
|------|--------|
| 新 API 端点开发 | UI 布局调整 |
| 数据库 schema 变更 | 配置文件修改 |
| 功能验证（有明确通过/失败标准） | 探索性实验（用 /spike） |
| 数据迁移（需要幂等安全） | 文档更新 |

## 与其他规则的关系

| 规则 | 互补关系 |
|------|---------|
| `deviation-handling.md` | Goal-Backward 定义目标，deviation 处理实现中的偏差 |
| `superpowers:writing-plans` | Goal-Backward 是 writing-plans 的增强子步骤 |
| `work-summary-rules.md` | 每个 Goal Assertion 完成后可独立 commit（Atomic Commits） |

## Atomic Commits（来自 GSD）

每个 Goal Assertion 满足后，**立即 commit**（不等全部完成）：

```
Assertion 1 满足 → git commit -m "feat: 实现 X"
Assertion 2 满足 → git commit -m "feat: 实现 Y"
Assertion 3 满足 → git commit -m "feat: 实现 Z + 集成测试"
```

**与 Task-Batch Checkpoint 的关系**：
- Atomic Commit = 断言级别（最细粒度）
- Task-Batch = 任务批次级别（多个相关原子 commit 的汇总检查点）
- 两者不冲突：Atomic Commit 保证粒度，Task-Batch 保证文档同步

## ChangeLogs

- [2026-04-26] Initial: Goal-Backward Methodology + Atomic Commits 规则
