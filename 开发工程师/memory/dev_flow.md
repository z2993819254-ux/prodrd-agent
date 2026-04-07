---
name: 开发流程
description: 从接 Issue 到提 PR 的完整开发流程
type: feedback
---

## 完整流程

### Step 1: 理解需求
- 阅读 Issue 全文（背景、需求、验收标准）
- **必须读 Issue comments**（`gh issue view --json comments`），需求方可能在 comment 里推翻 body 的方案
- 阅读关联 Issue 了解上下文
- 阅读相关文件列表中的代码
- 如果需求不清楚 → 在 Issue 下留言提问，不要猜

### Step 2: 创建分支
```bash
git checkout main && git fetch origin && git pull origin main
git checkout -b <type>/<description>
```

分支命名：
- `feat/add-xxx` — 新功能
- `fix/xxx-bug` — Bug 修复
- `refactor/xxx` — 重构
- `test/add-xxx-tests` — 测试
- `cleanup/remove-xxx` — 清理

### Step 3: 编码
- 按 Issue 需求编码，不扩大范围
- 先读现有代码理解上下文，再动手改
- 每个 Issue 一个分支一个 PR，不混入无关改动
- 不引入安全漏洞（注入、XSS 等）
- 不 hardcode 敏感信息（API key、密码）

### Step 4: 质量检查
```bash
# 根据项目配置执行（自动检测）
lint检查    # py_compile / eslint / ruff / golangci-lint / vite build
format检查  # prettier / black / gofmt
测试       # pytest / jest / go test
debug检查  # grep 确认无 print/console.log/debugger 遗留
```
- 所有检查必须通过，不通过不提交
- 如果有失败，修复后再继续

### Step 5: 文档同步
- 项目有 `CLAUDE.md` → 更新 Recent Changes
- 新增/修改 API → 更新 API 文档
- 只更新已有文档，不创建新文档
- 遗漏的更新必须在提 PR 前补上

### Step 6: 提交 PR
- 按 PR 提交规范（见 pr_standard.md）
- 关联 Issue（`Closes #XX`）
- 在 Issue 下留言测试结果
- **测试通过后直接提交，不需等用户确认**

### Step 7: 等待 Review
- Review 后根据评论返工（见 rework_standard.md）
- 不催 review，不自己合并
- Review 通过后等测试工程师验证，不主动 merge

## 轮询模式

开启 cron 定时轮询（默认 10 分钟）时，每次检查：
1. **新 Issue** — open 状态且未关联 PR 的 issue，按优先级（🔴>🟠>🟡）处理
2. **PR Review 返工** — open PR 的最新 comment 是否包含"仍需返工"/"Still Needs Changes"
