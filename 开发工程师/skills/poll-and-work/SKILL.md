---
name: poll-and-work
description: >
  定时轮询 GitHub 仓库，检查新 issue 和 PR review 返工需求，自动实现并提交。
  触发词: "开始轮询", "轮询", "poll", "监控issue", "自动做issue"
allowed-tools: Bash, Read, Write, Glob, Grep
argument-hint: <repo_url> [--interval 10m]
---

# 轮询并工作

> 开启定时 cron，每隔 N 分钟检查 GitHub 仓库，发现新 issue 或 PR 返工需求就自动处理。

## 启动

```
/loop <interval> 检查 <repo_url> 是否有新的 open issue（排除已有PR关联的），如果有就按 /submit-pr 流程实现并提交
```

默认 interval: 10m

## 每次轮询执行

### 1. 检查新 Issue

```bash
# 列出所有 open issue
gh issue list --repo <owner/repo> --state open --json number,title

# 列出所有 PR 关联的 issue 编号
gh pr list --repo <owner/repo> --state all --json body | 提取 Closes #N

# 差集 = 未关联 PR 的 issue
```

有新 issue → 按优先级（🔴 > 🟠 > 🟡）逐个处理：
1. 读 Issue（body + comments）
2. 创建分支
3. 实现代码
4. 质量检查（lint + test）
5. 按 /submit-pr 流程提交

### 2. 检查 PR Review 返工

```bash
# 遍历所有 open PR
for pr in open_prs:
    last_comment = gh pr view $pr --json comments | 最后一条
    if "仍需返工" or "Still Needs Changes" in last_comment:
        → 需要返工
```

有返工需求 → 按返工规范处理：
1. 读 review comment
2. 切到对应分支
3. 逐条修复
4. 测试 + push
5. 在 PR 和 Issue 上补测试报告

### 3. 无事可做

输出"无新 issue，无需返工的 PR"，等下次 cron 触发。

## 处理顺序

1. 新 issue 按优先级排序（🔴 先做）
2. PR 返工优先于新 issue（已有代码的修复比新功能重要）
3. 一次只做一个，做完提交后再处理下一个（避免上下文污染）

## 上下文管理

- 每个 issue/返工切到独立分支
- 做完一个就 `git checkout main`，再做下一个
- 不在一个分支上混合多个 issue 的改动
