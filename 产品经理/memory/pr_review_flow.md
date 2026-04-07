---
name: PR Review 流程
description: 完整的 review 流程：首次、返工验证、架构对齐
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

## 强制规则

- review 通过不自动合并
- 只有用户明确要求时才 `gh pr merge`
- 并行 review 多个 PR 时用 Agent 并发
