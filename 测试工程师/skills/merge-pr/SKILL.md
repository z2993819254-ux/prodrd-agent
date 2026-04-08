---
name: merge-pr
description: >
  测试通过后合并 PR。写测试报告到 PR comment，检查合并冲突，
  执行 merge，拉取最新 main。
  触发词: "合并PR", "merge pr", "测试通过合并"
allowed-tools: Bash, Read
argument-hint: <pr_number> [--repo <owner/repo>]
---

# 测试通过后合并 PR

## 前置条件

- 所有测试已通过（pytest + build + E2E）
- 已确认 PR review 为 LGTM

## Step 1: 检查合并冲突

```bash
gh pr view <number> --repo <owner/repo> --json mergeable --jq '.mergeable'
```

- `MERGEABLE` → 继续
- `CONFLICTING` → 尝试合并 main 解决冲突（简单冲突），或在 PR 写报告让 Dev rebase
- `UNKNOWN` → 直接尝试 merge，失败再处理

## Step 2: 写测试报告并合并

```bash
gh pr merge <number> --repo <owner/repo> --merge --body "$(cat <<'EOF'
## 🧪 测试报告 — PASSED

**测试人**: QA 测试工程师 | **日期**: <date>

| 项目 | 结果 |
|------|------|
| pytest | ✅ X passed |
| 前端 build | ✅ 通过 |
| E2E 浏览器 | ✅ 页面正常，零 JS 错误 |

### 验证内容
- 具体验证项 1 ✅
- 具体验证项 2 ✅
EOF
)"
```

## Step 3: 拉取最新 main

```bash
git checkout main && git pull origin main
```

## 冲突处理

如果 merge 失败（冲突）：

### 简单冲突（import 顺序、auto-merge 可解决）
```bash
git checkout <pr-branch>
git fetch origin main && git merge origin/main --no-edit
# 解决冲突
git push origin <pr-branch> --force-with-lease
gh pr merge <number> --merge --body "..."
```

### 复杂冲突（逻辑冲突）
在 PR 写报告，让 Dev 处理：
```bash
gh pr comment <number> --body "## 🧪 测试报告 — BLOCKED (合并冲突)
测试通过但有 X 处冲突，需要 Dev rebase。
冲突文件：...
"
```

## ⚠️ 规则

- 测试报告必须包含具体的测试项和结果
- 合并后必须拉取最新 main
- 不要用 `--force` push 到 main
- 返工 PR 重新测试后也要出完整测试报告
