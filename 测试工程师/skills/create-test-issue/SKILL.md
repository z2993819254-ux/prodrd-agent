---
name: create-test-issue
description: >
  测试发现 bug 时创建 GitHub Issue。包含复现步骤、错误截图、
  影响范围、优先级。遵循 6 部分标准模板。
  触发词: "报bug", "创建测试issue", "test issue", "报告bug"
allowed-tools: Bash, Read, Glob, Grep
argument-hint: <bug_description> [--repo owner/repo] [--priority P0|P1|P2|P3]
---

# 测试发现 Bug 创建 Issue

## Step 1: 收集 Bug 信息

从测试过程中收集：

1. **触发条件** — 在哪个页面/API/操作中发现
2. **错误现象** — 具体错误信息、截图、console 日志
3. **预期行为** — 应该是什么样
4. **复现步骤** — 1-2-3 步骤
5. **影响范围** — 哪些功能受影响
6. **关联 PR** — 在测试哪个 PR 时发现的

## Step 2: 按模板组织

```markdown
## 背景

### 问题描述
在测试 PR #XX 时发现 [简述问题]。

### 错误示例
<!-- 具体错误信息/截图/console 日志 -->

## 复现步骤

1. 打开 http://localhost:5174/xxx
2. 点击 xxx 按钮
3. 观察到 xxx 错误

**预期行为**: xxx
**实际行为**: xxx

## 需求

### 修复方案
- 文件: `path/to/file.py:line`
- 问题: [根因分析]
- 建议修复: [代码示例]

## 验收标准

- [ ] [具体可验证条件 1]
- [ ] [具体可验证条件 2]
- [ ] 回归：已有功能不受影响

## 元信息

**优先级**: P0 🔴 / P1 🟡 / P2 🟠 / P3 🔵

**相关文件**:
- `file1.py` — 问题所在
- `file2.jsx` — 受影响

**关联 Issue/PR**:
- PR #XX — 测试时发现
```

## Step 3: 确认后创建

**展示给用户确认后再创建。**

```bash
gh issue create --repo <owner/repo> \
  --title "<emoji> <标题>" \
  --label "bug" \
  --body "<正文>"
```

## 优先级判断

| 级别 | 判断标准 |
|------|----------|
| P0 🔴 | 页面崩溃、数据丢失、安全漏洞 |
| P1 🟡 | 核心功能不可用、严重体验问题 |
| P2 🟠 | 非核心 bug、UI 不一致、边界条件 |
| P3 🔵 | 优化建议、代码质量、文档缺失 |

## ⚠️ 规则

- 验收标准必须可验证（不接受"正常工作"）
- 必须有复现步骤
- 创建前展示给用户确认
- 标注关联 PR（在测试哪个 PR 时发现的）
