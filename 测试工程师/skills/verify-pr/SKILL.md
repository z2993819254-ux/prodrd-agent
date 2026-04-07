---
name: verify-pr
description: >
  验证 PR 功能是否正常。拉取 PR 分支，跑自动化测试，按 Issue 验收标准逐条手动验证，
  输出测试报告。只测 PM 标记 LGTM 的 PR。
  触发词: "测试PR", "verify pr", "验证PR", "QA测试", "跑测试"
allowed-tools: Bash, Read, Glob, Grep
argument-hint: <pr_number> [--repo owner/repo]
---

# PR 测试验证流程

## Step 1: 确认 PR 状态

```bash
# 确认 PM 已 LGTM
gh pr view <number> --repo <owner/repo> --json comments \
  --jq '.comments[-1].body' | grep -i "LGTM"
```

如果最新评论不包含 LGTM，停止——告诉用户"此 PR 尚未通过 PM review"。

## Step 2: 理解测试范围

1. 读 PR 正文 — 了解改了什么
2. 读关联 Issue 的验收标准 — 这是测试 checklist
3. 读 PM 的 review comment — 了解关注点

```bash
# 获取 PR 关联的 Issue
gh pr view <number> --json body --jq '.body' | grep -oP '#\d+'
# 读 Issue 验收标准
gh issue view <issue_number> --json body
```

## Step 3: 拉取并准备环境

```bash
gh pr checkout <number>
# 按项目类型安装依赖
```

## Step 4: 自动化测试

```bash
# 自动检测项目类型并执行
# Python: pytest tests/
# JavaScript: npm test
# Go: go test ./...
# 前端: npx vite build
```

记录通过/失败/跳过数量。

## Step 5: 手动验证

按 Issue 验收标准逐条测试。每条记录 pass/fail。

## Step 6: 输出报告

### 通过

```markdown
## QA Test Report (PR #XX)

### 自动化测试
- [x] 单元测试 — ✅ XX passed, 0 failed
- [x] 构建检查 — ✅ build success

### 验收标准验证（对照 Issue #YY）
- [x] 条件 1 — ✅
- [x] 条件 2 — ✅

### 回归检查
- [x] 相关功能不受影响 ✅

### 结论
✅ **测试通过，可以合并**
```

### 不通过

列出失败项 + 复现步骤，不需要修代码。

## Step 7: 检查合并冲突

```bash
gh pr view <number> --json mergeable
```

无冲突 → 报告中注明"可以合并"
有冲突 → 报告中注明"需要 Dev rebase"

## ⚠️ 规则

- 只测 PM 标记 LGTM 的 PR
- 验收标准每一条都要测
- 不通过时必须有复现步骤
- 不改代码
- 测试报告留到 PR 的 comment 里
