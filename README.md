# prodrd-agent

三个 Claude Code Agent 协作完成软件开发：产品经理拆需求、开发工程师写代码、测试工程师验功能。

## 架构

```
用户（你）
  │
  ├── 产品经理 Agent ──── 拆需求 → 写 Issue → Review PR → 判定返工/通过
  │
  ├── 开发工程师 Agent ── 领 Issue → 写代码 → 提 PR → 按 Review 返工
  │
  └── 测试工程师 Agent ── PM 通过后 → 跑测试 → 验收标准逐条过 → 确认可合并
```

## 完整流程

```
需求 → PM 拆解 → Issue → Dev 开发 → PR → PM Review ─┬─ LGTM → QA 测试 ─┬─ 通过 → 合并
                                                      │                    │
                                                      └─ 返工 → Dev 修复 ──┘  失败 → Dev 修复
```

### 1. 需求拆解（产品经理）

用户提出需求后，产品经理 Agent 负责：

- 阅读相关代码，理解现状
- 搜索已有 Issue 避免重复
- 创建标准化 Issue，必须包含 6 部分：

| 部分 | 内容 |
|------|------|
| 背景 | 问题描述（≤3句）+ 错误示例（代码/日志/截图） |
| 需求 | 实现步骤 + 字段/接口变更表 + 改进后代码示例 |
| 验收标准 | 可验证的 checklist，覆盖正常和异常路径 |
| 优先级 | P0 紧急 / P1 本周 / P2 本月 / P3 待定 |
| 相关文件 | 具体路径 + 说明 |
| 关联 Issue | 依赖/被依赖/相关/替代关系 |

### 2. 开发实现（开发工程师）

开发工程师 Agent 领取 Issue 后：

1. **读 Issue** — body + comments 都要读（comments 可能推翻 body 方案）
2. **建分支** — `feat/issue-1-xxx`、`fix/issue-2-xxx`，从最新 main 拉
3. **写代码** — 严格按 Issue 做，不扩大范围，不加"顺手优化"
4. **质量检查** — lint + format + test 全部通过，不跳 `--no-verify`
5. **同步文档** — 有 API 变更更新 API 文档，有配置变更更新 CLAUDE.md
6. **提 PR** — 关联 `Closes #XX`，附完整 checklist 和测试证据
7. **Issue 留言** — 把测试报告也贴到 Issue 评论区

PR 模板：

```markdown
## Summary
- 变更点 1
- 变更点 2

## Checklist
- [x] Lint ✅
- [x] Format ✅
- [x] Tests ✅ 12 passed, 0 failed
- [x] Rebase done
- [x] Docs updated

## Test Evidence
- 环境: ...
- 测试命令及结果: ...

Closes #XX
```

### 3. PR Review（产品经理）

产品经理 Agent 对 PR 做 Review：

- **对照验收标准逐条检查** — 不只是找 bug，是看需求有没有做到
- **检查代码质量** — 安全漏洞、边界处理、逻辑错误
- **给出明确结论**：
  - ✅ **LGTM** — "Code review passed, awaiting QA verification before merge"
  - ❌ **Needs Changes** — 列出每个问题（严重度 + 位置 + 修复建议）

返工 Re-review 时会逐条追踪：修复的标 ✅，未修复的标 ❌，检查修复是否引入新问题。

### 4. 测试验证（测试工程师）

PM 给出 LGTM 后，测试工程师 Agent 介入：

1. Checkout PR 分支，安装依赖
2. 跑自动化测试（unit / integration / build）
3. **逐条验证 Issue 验收标准**（手动测试）
4. 回归测试相关功能
5. 发布测试报告：

```markdown
## QA Test Report (PR #XX)

### Automated Tests
- [x] Unit tests — ✅ 24 passed, 0 failed
- [x] Build check — ✅ success

### Acceptance Criteria Verification (Issue #YY)
- [x] 标准 1 — ✅ Pass
- [x] 标准 2 — ✅ Pass

### Regression Check
- [x] 相关功能 A — 无影响 ✅

### Conclusion
✅ Tests passed, ready to merge
```

6. 检查合并冲突，确认可合并后留言通知

### 5. 合并

三关全过（Dev 自测 → PM Review → QA 验证），由用户决定合并。**任何 Agent 都不会自动合并 PR。**

## 轮询模式

开发工程师和产品经理支持定时轮询（`/loop 30m`）：

| Agent | 轮询内容 |
|-------|---------|
| 产品经理 | 新 PR（无 review）、返工 PR（review 后有新 commit） |
| 开发工程师 | 新 Issue（open 且无关联 PR，按 P0>P1>P2 排序）、需返工的 PR |

处理优先级：返工 > 新任务。

## 角色边界

| | 产品经理 | 开发工程师 | 测试工程师 |
|---|:---:|:---:|:---:|
| 拆需求/写 Issue | ✅ | ❌ | ❌ |
| 写代码 | ❌ | ✅ | ❌ |
| Review PR | ✅ | ❌ | ❌ |
| 跑测试/验收 | ❌ | 自测 | ✅ |
| 合并 PR | ❌ | ❌ | ❌ |
| 决定需求范围 | ✅ | ❌ | ❌ |

## 目录结构

```
prodrd-agent/
├── 产品经理/
│   ├── agent.md         # Claude Code Agent 定义（frontmatter + 系统提示）
│   ├── memory/          # 角色定义、Issue 规范、Review 流程、轮询规则
│   └── skills/
│       └── create-issue/   # 标准化 Issue 创建技能
├── 开发工程师/
│   ├── agent.md         # Claude Code Agent 定义（建议 model: opus）
│   ├── memory/          # 角色定义、开发流程、Git 规范、PR 规范、返工规范
│   └── skills/
│       ├── poll-and-work/  # 轮询领任务+自动开发
│       └── submit-pr/      # 标准化 PR 提交技能
├── 测试工程师/
│   ├── agent.md         # Claude Code Agent 定义
│   ├── memory/          # 角色定义、测试流程、报告模板、回归检查
│   └── skills/
│       └── verify-pr/     # PR 验证技能
├── 协作流程/
│   └── README.md        # 流程总览文档
└── LESSONS-LEARNED.md   # 推行经验：模型选型、Team 模式、串行门禁、通信分层等
```

## 使用方式

每个角色目录是一个独立的 Claude Code Agent 配置：

1. 把 `<角色>/agent.md` 放到 `~/.claude/agents/<name>.md`（用户级）或项目 `.claude/agents/<name>.md`（项目级）——这是该 Agent 的入口定义。
2. 把 `<角色>/memory/` 和 `<角色>/skills/` 放到该 Agent 对应的 Claude Code 项目配置中。
3. 三个 Agent 通过 GitHub Issue / PR 评论区沉淀正式结论；实时拉齐走 Claude Code Team 内部消息（详见 [LESSONS-LEARNED.md](./LESSONS-LEARNED.md)）。

## 推行经验

首次在项目里落地这套流程前建议先读 [LESSONS-LEARNED.md](./LESSONS-LEARNED.md)，里面记录了模型选型（Dev 用 Opus）、Team 模式边界、Skill 显式调用、串行门禁等踩坑后总结的规则。
