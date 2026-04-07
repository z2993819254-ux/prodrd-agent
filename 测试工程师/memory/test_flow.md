---
name: 测试流程
description: 从 PM LGTM 到确认合并的完整测试流程
type: feedback
---

## 完整流程

### Step 1: 识别待测 PR

查找 PM 已标记 LGTM 的 PR：
```bash
# 查看 open PR 的最新 comment，找包含"LGTM"的
gh pr list --repo {owner/repo} --state open --json number,title,comments \
  --jq '.[] | select(.comments[-1].body | test("LGTM")) | {number, title}'
```

### Step 2: 理解测试范围

1. 读 PR 正文（改了什么、关联哪个 Issue）
2. 读 Issue 的验收标准（必须逐条测试）
3. 读 PM 的 review comment（了解 PM 关注的点）

### Step 3: 环境准备

```bash
# 拉取 PR 分支
gh pr checkout <number>

# 安装依赖
npm install  # 或 pip install -r requirements.txt

# 启动服务（如需要）
```

### Step 4: 自动化测试

```bash
# 根据项目类型执行
pytest tests/         # Python
npm test              # JavaScript
go test ./...         # Go
npx vite build        # 前端构建检查
```

记录：通过数、失败数、跳过数

### Step 5: 手动验证

按 Issue 验收标准逐条验证：
- 每条标准都要测试
- 记录 pass/fail
- 失败的记录复现步骤

### Step 6: 报告结果

**通过 →** 在 PR 留言（见 test_report.md）
**不通过 →** 在 PR 留言具体问题 + 复现步骤，等 Dev 修复后重新测试

### Step 7: 确认合并

测试通过后：
```bash
# 检查是否有 merge 冲突
gh pr view <number> --json mergeable

# 无冲突 → 在 PR 留言"测试通过，可以合并"
# 有冲突 → 留言"测试通过但有 merge 冲突，需要 Dev rebase"
```
