---
name: PR 提交规范
description: PR 的标题、正文、checklist 标准模板
type: feedback
---

## PR 标题

格式：`<type>: <简短描述>`，70 字符以内

```
feat: add user authentication middleware
fix: handle null pointer in download service
refactor: split ProjectWizard into step components
test: add retry API endpoint tests
cleanup: remove deprecated mock data
```

## PR 正文模板

```markdown
## Summary
- 改动点 1
- 改动点 2

## Checklist
- [x] Lint — ✅ pass
- [x] Format — ✅ pass / N/A
- [x] Tests — ✅ pass (XX passed, 0 failed)
- [x] Rebase to latest main — done
- [ ] Docs updated — N/A
- [ ] CLAUDE.md updated — N/A

## Test Evidence

### 测试环境
- 环境描述

### 测试结果
```bash
# 场景 1: 正常路径
$ <command>
→ 结果摘要 ✅

# 场景 2: 异常路径
$ <command>
→ 预期错误 ✅
```

### 验收标准 (对照 Issue)
- [x] 验收条件 1
- [x] 验收条件 2

Closes #XX
```

## Issue 留言模板

PR 创建后，必须在关联 Issue 留言测试报告：

```markdown
## PR Test Results (PR #XX)

### Checklist
- [x] Lint — ✅ pass
- [x] Tests — ✅ pass (XX passed)
- [x] Rebase to latest main — done

### Test Evidence
```bash
$ <command> → 结果 ✅
```

### 验收标准
- [x] 条件 1
- [x] 条件 2

PR: #XX
```

## 规则

- Checklist 不适用的项标 N/A，不删除
- 测试结果用 `→` 箭头单行摘要，不贴完整 JSON
- 一个 PR 只关联一个 Issue
- 不混入无关改动
- PR 和 Issue 留言都要有 Test Evidence
