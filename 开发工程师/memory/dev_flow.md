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
- **必须跑项目级 linter**（如 `golangci-lint run`、`make lint`），不能只跑 `go vet` / `python -m py_compile`。CI lint 规则通常比基础工具严得多（golines/wsl/gci 等）

### Step 4.5: Curl Test（e2e 验证）
**如果改动涉及 HTTP API，必须做真 curl test：**
1. `make build`（或对应的构建命令）
2. 启动本地服务，连接真实数据库
3. 用 `curl` 打真实 HTTP 请求，覆盖正常 + 异常场景
4. 记录真实的请求命令和响应摘要

**不能用单元测试/集成测试的输出冒充 curl test。** 单元测试过了不代表 API 端到端能用。

如果没有本地数据库环境 → 先问用户要连接信息，不能跳过。

### Step 5: 提交前自检（Constitution Review）
**此步骤必须自动执行，不可跳过。** 对照项目规范逐条检查：
- [ ] API 变更（新增/修改字段、endpoint）→ API 文档必须同步更新
- [ ] `CLAUDE.md` Recent Changes 必须更新
- [ ] 如果项目有 constitution/规范文件 → 对照检查（validator 测试、结构化日志、ACL 文档等）
- [ ] 检查是否有遗漏的文档更新
- 只更新已有文档，不创建新文档
- 遗漏的更新必须在提 PR 前补上，**不能留到 review 阶段被人指出**

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

## 🚨 历史教训（每次执行前必读）

### 教训 1: 必须读完 issue 所有 comments 再动手
> **事故**: Issue body 推荐方案 A，但需求方在 comment 里明确要求方案 B。没读 comment 直接用了方案 A，整个 PR 重做。
>
> **规则**: `gh issue view` 后**必须**接 `--json comments` 读完所有评论。需求方可能在 comment 里推翻 body 的方案。

### 教训 2: curl test 必须是真 curl，不是单元测试
> **事故**: 用集成测试的输出冒充 curl test，被指出"过于敷衍"。
>
> **规则**: e2e test = 启动服务 → `curl` 打真实 HTTP 请求 → 贴真实命令和响应。不能用 `go test` / `pytest` 的输出格式化成 curl 样子。

### 教训 3: 提交前自检不可跳过
> **事故**: 漏了 API 文档和 CLAUDE.md 更新，被 review 打回。
>
> **规则**: Step 5 (Constitution Review) 不可跳过。遗漏的文档更新必须在同一次提交中补上。

### 教训 4: CI lint 比本地严
> **事故**: 多次本地通过但 CI lint 失败（golines/wsl/gci/goconst 等严格规则）。
>
> **规则**: 本地必须跑项目级 linter（`golangci-lint run` / `make lint`），不能只靠基础工具。

### 教训 5: CI 结果必须等跑完再更新
> **事故**: Issue 留言时 CI 还没跑完就标了"待 CI 运行"，被追问。
>
> **规则**: `gh pr checks --watch` 等 CI 全部完成后，再更新 Issue 留言把 CI 结果勾上。
