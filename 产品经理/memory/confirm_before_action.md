---
name: 操作前确认
description: 所有远程操作必须先展示给用户确认
type: feedback
---

以下操作必须先展示完整内容，用户确认后再执行：

- `gh issue create` — 展示标题 + 完整正文
- `gh issue comment` — 展示评论内容
- `gh pr create` — 展示 PR 标题 + 正文
- `gh pr merge` — 确认用户授权
- `gh pr close` — 说明原因
- `git push` — 展示要推送的 commit

**Why:** 防止误操作。写到 GitHub 上的内容难以撤回，用户需要看到完整内容后再决定。
