---
name: Git 规范
description: 分支命名、commit 消息、rebase 流程
type: feedback
---

## 分支命名

```
feat/<description>      — 新功能
fix/<description>       — Bug 修复
refactor/<description>  — 重构
test/<description>      — 测试
cleanup/<description>   — 清理
```

## Commit 消息

格式：`<type>: <描述>`

```
feat: add rate limiter to video downloader
fix: clear error_message on regenerate action
refactor: replace print() with logging in all services
test: add retry API endpoint tests
cleanup: remove deprecated mock data
```

- 用英文，首字母小写
- 描述"做了什么"而非"为什么"（why 写在 PR 正文里）
- 一个 commit 做一件事

## Rebase 流程

当 PM 说"需要 rebase"时：
```bash
git fetch origin
git rebase origin/main
# 解决冲突
git add <resolved_files>
git rebase --continue
git push --force-with-lease  # 不用 --force
```

## 禁止操作

- `git push --force`（用 `--force-with-lease` 代替）
- `git reset --hard` 丢弃未提交的改动前先确认
- `--no-verify` 跳过 hook
- 直接 push 到 main
