---
name: PR Review 流程
description: 完整的 review 流程：首次、返工验证、架构对齐、防遗漏检查
type: feedback
---

## 首次 Review

1. `gh pr diff <number>` 获取完整 diff
2. 检查是否符合 Issue 需求（对照验收标准逐条）
3. 检查 bug、安全问题、代码质量
4. 写 review 评论：
   - 改了什么（文件、行数）
   - 问题列表（严重度 🔴🟡🟠 + 具体位置 file:line + 修复建议）
   - Verdict：LGTM / Needs Changes
5. LGTM → "代码 review 通过，待测试工程师验证后合并"
6. Needs Changes → 列出问题 + 验收标准

## 返工 Re-review

1. 检查最新 commit 时间，确认有新提交
2. 逐条对照之前每轮提出的问题：修好的 ✅，没修的 ❌
3. 检查修复是否引入新问题
4. 全部通过 → LGTM，否则继续 Needs Changes

## 架构对齐检查

当架构方向变更时，检查所有 open PR：
- 方向一致 → 保留
- 方向冲突 → 关闭 + 写明原因和替代方案
- 需要调整 → 写 comment 要求返工
- 被取代 → 关闭 + 标注被哪个 PR/Issue 取代

## ⚠️ 防遗漏检查（每次 review 时必须执行）

**不能只看 comment 数为 0 的 PR！** 必须同时检查所有 open PR 的最新 commit 时间是否晚于最新 review comment 时间：

```bash
# 检查每个 open PR 的最新 commit vs 最新 review
for pr in $(gh pr list --repo {owner/repo} --json number --jq '.[].number'); do
  echo "PR #$pr:"
  gh pr view $pr --json commits --jq '.commits[-1] | {date: .committedDate, msg: .messageHeadline}'
  gh pr view $pr --json comments --jq '.comments[-1] | {date: .createdAt, body: .body[0:60]}'
done
```

以下情况容易遗漏：
- 开发修完 push 了但只在 Issue 留言，PR 上没新 comment → 以为没返工
- PR 做了 rebase（新 commit）但内容没变 → 需要确认 rebase 没破坏修复
- 上次 review 是 LGTM 但之后 PR 又有新 commit → 需要确认新改动没问题
- 开发提了新 PR 替代旧 PR → 旧 PR 应该关闭

## 强制规则

- review 通过不自动合并
- 只有用户明确要求时才 `gh pr merge`
- 并行 review 多个 PR 时用 Agent 并发
