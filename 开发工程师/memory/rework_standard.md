---
name: 返工规范
description: 按 PR review 评论逐条修复的流程
type: feedback
---

## 返工流程

1. **读 review 评论** — 逐条理解提出的问题
2. **切到对应分支** — `git checkout <pr-branch>`
3. **按严重度排序修** — 🔴 先修，🟠 再修，🟡 最后
4. **修复 + 测试** — lint/test 必须通过（和首次提交同等标准）
5. **commit + push**
6. **在 PR 上回复** — 逐条说明已修复
7. **在 Issue 上补测试报告** — 返工也要有 Test Evidence

## PR 回复模板

```markdown
## Review 返工完成

- [x] Bug 1: xxx — 已修复
- [x] Bug 2: xxx — 已修复

### Test Evidence
```bash
$ pytest tests/ → XX passed ✅
```
```

## Issue 留言模板

```markdown
## PR Test Results — Review 返工 (PR #XX)

### Checklist
- [x] Lint — ✅ pass
- [x] Tests — ✅ pass (XX passed)

### 返工内容
- [x] Bug 1 修复
- [x] Bug 2 修复

PR: #XX
```

## 注意

- 返工后和首次提交同等标准（lint/test 必须过，Issue 必须有测试报告）
- 如果不理解 review 的某个要求，在 PR 下留言提问，不要猜
- 不要 amend 已有 commit，创建新 commit
- 不加额外改动——只修 review 提出的问题
