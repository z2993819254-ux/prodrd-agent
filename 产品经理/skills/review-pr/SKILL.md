---
name: review-pr
description: >
  PR Review 标准化流程。扫描仓库所有 open PR（含返工提交），获取 diff 分析代码，
  对照 Issue 验收标准逐条检查，输出结构化 review 报告，展示给用户确认后写到 PR comment。
  支持首次 review、返工 re-review、架构对齐检查。
  触发词: "review pr", "review一下", "检查pr", "看看pr", "有新pr吗"
allowed-tools: Bash, Read, Grep, Glob, Agent
argument-hint: [--repo owner/repo] [--pr <number>] [--all]
---

# PR Review 标准化流程

> 产品经理 review PR 的完整流程。不只看 bug，要对照 Issue 验收标准检查实现是否符合需求。
> review 通过不自动合并，写 LGTM 评论后等测试工程师验证。

---

## Step 1: 扫描待 review 的 PR

### 1.1 新 PR（没有 review comment 的）

```bash
gh pr list --repo {owner/repo} --state open --json number,title,comments \
  --jq '.[] | select(.comments | length == 0) | {number, title}'
```

### 1.2 返工 PR（有新 commit 晚于最新 review 的）

**⚠️ 不能只看 comment 数为 0 的 PR！必须检查所有 open PR：**

```bash
for pr in $(gh pr list --repo {owner/repo} --state open --json number --jq '.[].number'); do
  echo "PR #$pr:"
  gh pr view $pr --json commits --jq '.commits[-1] | "commit: \(.committedDate) \(.messageHeadline)"'
  gh pr view $pr --json comments --jq '.comments[-1] | "review: \(.createdAt) \(.body[0:60])"'
done
```

**如果最新 commit 时间 > 最新 review comment 时间 → 有返工需要 re-review。**

---

## Step 2: 获取 PR 信息

对每个待 review 的 PR：

```bash
# 获取 diff
gh pr diff <number> --repo {owner/repo}

# 获取关联 Issue（从 PR 正文中提取 #XX）
gh pr view <number> --json body --jq '.body' | grep -oP '#\d+'

# 读取 Issue 验收标准
gh issue view <issue_number> --repo {owner/repo} --json body
```

---

## Step 3: 分析代码

对每个 PR 检查：

### 3.1 需求符合度
- 对照 Issue 验收标准逐条检查
- 是否有遗漏的验收项
- 是否做了 Issue 没要求的额外改动

### 3.2 Bug 检查
- 逻辑错误
- 边界条件（空值、空列表、null）
- 类型安全（Python 类型不匹配、JS undefined 访问）
- 资源泄漏（未关闭连接、内存无限增长）

### 3.3 安全检查
- XSS（用户输入注入 HTML）
- 注入攻击（SQL 注入、命令注入）
- 敏感信息泄露（API key、密码在代码/日志中）
- 认证/授权缺失

### 3.4 代码质量
- 异常处理（不要 `except: pass` 静默吞掉）
- 命名清晰、逻辑自解释
- 不引入不必要的复杂度

### 3.5 架构对齐（如果有 architecture_decisions.md）
- PR 是否和当前架构方向冲突
- 冲突的 → 建议关闭 + 说明原因

---

## Step 4: 生成 review 报告

### 首次 Review 报告格式

```markdown
## Review 报告 — [Verdict]

### 改了什么
- 文件列表 + 行数

### 问题列表
| 严重度 | 问题 | 位置 |
|--------|------|------|
| 🔴 | 描述 | file:line |
| 🟡 | 描述 | file:line |

### 验收标准
- [ ] 条件 1 — ✅/❌
- [ ] 条件 2 — ✅/❌

### Verdict: LGTM / Needs Changes
```

### 返工 Re-review 报告格式

```markdown
## Re-review: [Verdict]

之前的问题：
- ✅ 问题 1 — 修复正确
- ❌ 问题 2 — 仍未修复
- 🟡 新问题 — xxx

### Verdict: LGTM / Still Needs Changes
```

---

## Step 5: 展示给用户确认

**写到 PR 前必须先展示完整报告给用户确认。**

如果有多个 PR，展示汇总表：

```
| PR | 标题 | Verdict | 关键问题 |
|----|------|---------|---------|
| #42 | xxx | LGTM | 无 |
| #43 | yyy | Needs Changes | XSS 漏洞 |
```

用户确认后才执行 `gh pr comment`。

---

## Step 6: 写 review 评论

```bash
gh pr comment <number> --repo {owner/repo} --body "<报告内容>"
```

### LGTM 评论模板

```markdown
## Review: ✅ LGTM

代码 review 通过，待测试工程师验证后合并。

**确认项：**
- ✅ xxx
- ✅ xxx
```

### Needs Changes 评论模板

```markdown
## Review 报告 — 需要返工

### 🔴 Bug: xxx
...具体描述 + 修复建议...

### 验收标准
- [ ] xxx
- [ ] xxx

### Verdict: Needs Changes
```

---

## Step 7: 处理冲突 PR

如果 PR 和架构方向冲突或被其他 PR 取代：

```bash
# 关闭并说明原因
gh pr close <number> --repo {owner/repo} --comment "关闭：<原因>"
```

如果 PR 有 merge 冲突：

```bash
# 留言要求 rebase
gh pr comment <number> --repo {owner/repo} --body "需要 rebase 到最新 main，解决冲突后重新 review。"
```

---

## ⚠️ 强制规则

1. **review 通过不自动合并** — 只写 LGTM，不执行 `gh pr merge`
2. **写评论前必须用户确认** — 展示完整报告后再 `gh pr comment`
3. **不能只看新 PR** — 必须检查所有 open PR 的返工提交
4. **对照验收标准** — 不只看 bug，要看是否满足 Issue 需求
5. **并行 review** — 多个 PR 时用 Agent 工具并发分析，提高效率
6. **架构对齐** — 如果有 architecture_decisions.md，对照检查

---

## 使用方式

```bash
# 扫描所有待 review 的 PR（新 PR + 返工提交）
/review-pr --repo owner/repo --all

# review 指定 PR
/review-pr --repo owner/repo --pr 42

# 不指定仓库则从 git remote 自动检测
/review-pr --all
```
