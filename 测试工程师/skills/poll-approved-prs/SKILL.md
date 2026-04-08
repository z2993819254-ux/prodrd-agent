---
name: poll-approved-prs
description: >
  轮询 review 通过的 PR。检查 GitHub formal approve 和 comment 中的 LGTM/✅，
  发现待测 PR 后触发测试流程。支持多仓库。
  触发词: "查看待测PR", "poll prs", "检查PR", "有没有待测的"
allowed-tools: Bash
argument-hint: [--repo <owner/repo>] [--interval <minutes>]
---

# 轮询待测 PR

## Step 1: 检查 GitHub formal approve

```bash
gh pr list --state open --repo <owner/repo> \
  --json number,title,reviewDecision,headRefName \
  --jq '.[] | select(.reviewDecision == "APPROVED") | "#\(.number) \(.title)"'
```

## Step 2: 检查 comment 中的 LGTM

团队可能用 PR comment 而非 formal review 来表示 approve：

```bash
for pr in $(gh pr list --state open --repo <owner/repo> --json number --jq '.[].number'); do
  LGTM=$(gh pr view $pr --repo <owner/repo> --json comments \
    --jq '.comments[-1].body' 2>/dev/null | grep -ci "LGTM\|Review.*✅")
  if [ "$LGTM" -gt 0 ]; then
    TITLE=$(gh pr view $pr --repo <owner/repo> --json title --jq '.title')
    echo "#$pr $TITLE"
  fi
done
```

## Step 3: 同时检查返工 PR

```bash
for pr in $(gh pr list --state open --repo <owner/repo> --json number --jq '.[].number'); do
  REWORK=$(gh pr view $pr --repo <owner/repo> --json comments \
    --jq '.comments[-1].body' 2>/dev/null | grep -ci "需要返工\|Needs Changes")
  if [ "$REWORK" -gt 0 ]; then
    echo "⚠️ #$pr 需要返工"
  fi
done
```

## Step 4: 设置定时轮询（可选）

```
/loop 30m /poll-approved-prs --repo owner/repo
```

## 多仓库支持

如果负责多个仓库，依次检查每个：

```bash
REPOS="arnold-gopagoda/video-magic-system arnold-gopagoda/MagicCut"
for repo in $REPOS; do
  echo "=== $repo ==="
  # ... 执行 Step 1-3
done
```

## ⚠️ 规则

- 没有待测 PR 时静默跳过，不输出多余信息
- 发现待测 PR 后立即开始测试流程
- 同时检查返工 PR，返工修复后也要重新测试
- 读完 PR 所有 comments 再判断（最新 comment 可能推翻之前的 LGTM）
