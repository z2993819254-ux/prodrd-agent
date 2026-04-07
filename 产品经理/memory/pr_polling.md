---
name: PR 轮询机制
description: 定期检查仓库新 PR 和返工提交
type: feedback
---

## 轮询设置

使用 `/loop 30m` 设置每 30 分钟自动检查：

```
/loop 30m 检查 {owner/repo} 仓库是否有新的 open PR。如果有未 review 的 PR（没有 review comment 的），对每个 PR 执行：1) 用 gh pr diff 获取完整 diff 2) 分析代码变更 3) 生成 review 报告 4) 展示给用户确认。不要自动合并。
```

## 检查内容

### 新 PR
```bash
gh pr list --repo {owner/repo} --state open --json number,title,comments \
  --jq '.[] | select(.comments | length == 0) | {number, title}'
```

### 返工提交（已 review 但有新 commit）
```bash
for pr in {pr_numbers}; do
  gh pr view $pr --repo {owner/repo} --json commits \
    --jq '.commits[-1] | {date: .committedDate, msg: .messageHeadline}'
done
```

## 轮询注意事项

- `/loop` 只在当前会话生效，关会话就停
- 轮询时并行 review 多个 PR（用 Agent 工具）
- 检查到新 PR 或返工时立即 review，不等下一次轮询
