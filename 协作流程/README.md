# 三角色协作流程

## 角色定义

| 角色 | 职责 | 不做什么 |
|------|------|---------|
| **产品经理 (PM)** | 拆解需求、写 issue、review PR 是否符合需求、检查 bug、架构决策 | 不写代码、不合并 PR、不跑测试 |
| **开发工程师 (Dev)** | 接 issue 写代码、提 PR、根据 review 返工 | 不自己决定需求范围、不跳过 lint/test |
| **测试工程师 (QA)** | 验证 PR 功能、跑测试、确认可合并 | 不改代码、不决定需求 |

## 完整协作流程

```
需求 → PM 拆解 → Issue → Dev 开发 → PR → PM Review → QA 测试 → 合并
                                         ↑                    |
                                         └── 返工 ────────────┘
```

### 阶段 1：需求拆解（PM）

```
用户提出需求
→ PM 确认需求细节（不确定就问用户）
→ PM 创建 Issue（6 部分：背景、错误示例、需求、验收标准、优先级、相关文件）
→ 用户确认 Issue 内容
→ Issue 创建到 GitHub
```

### 阶段 2：开发（Dev）

```
Dev 认领 Issue
→ 创建分支（feat/fix/refactor + 简短描述）
→ 写代码
→ 本地 lint + format + test 通过
→ 提交 PR（关联 Issue、包含 checklist）
→ 在 Issue 下留言测试结果
```

### 阶段 3：PR Review（PM）

```
PM 检查到新 PR（定期轮询或通知）
→ 获取 diff，分析代码
→ 对照 Issue 验收标准逐条检查
→ 检查 bug、安全问题、代码质量
→ 写 review 评论：
  - LGTM → "代码 review 通过，待测试工程师验证后合并"
  - Needs Changes → 列出问题 + 验收标准，Dev 返工
```

### 阶段 4：返工循环（Dev ↔ PM）

```
Dev 根据 review 修复问题
→ push 新 commit
→ PM re-review：逐条对照之前的问题
  - 全部修好 → LGTM
  - 还有问题 → 继续 Needs Changes
→ 循环直到 LGTM
```

### 阶段 5：测试验证（QA）

```
PM 给出 LGTM 后
→ QA 拉取 PR 分支
→ 跑自动化测试（单元测试、集成测试）
→ 手动验证关键功能（按 Issue 验收标准）
→ 测试通过 → 在 PR 留言"测试通过，可以合并"
→ 测试不通过 → 在 PR 留言具体问题，Dev 修复后重新验证
```

### 阶段 6：合并（用户/PM 授权）

```
QA 确认测试通过
→ PM 或用户确认合并
→ 执行 gh pr merge
→ 删除分支
```

## 异常流程

### 架构方向变更
```
用户决定架构变更
→ PM 检查所有 open PR 和 Issue
→ 和新方向冲突的 → 关闭，写明原因
→ 需要调整的 → 写 comment 要求返工
→ 过时的 Issue → 关闭，标注被哪个取代
```

### PR 有 Git 冲突
```
PM review 时发现冲突
→ 在 PR 留言"需要 rebase 到最新 main"
→ Dev 执行 rebase 解决冲突
→ PM 重新 review
```

### 多个 PR 互相依赖
```
PM 确定合并顺序
→ 按顺序 review + 测试 + 合并
→ 后续 PR 需要 rebase
```
