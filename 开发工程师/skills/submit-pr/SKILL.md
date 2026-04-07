---
name: submit-pr
description: >
  通用 PR 提交流程。自动检测项目语言/工具链，执行 lint、测试、文档检查，
  生成标准化 PR 并留言测试结果到关联 Issue。适用于任何项目。
  触发词: "提交PR", "submit pr", "create pr", "开PR", "发PR", "pr提交"
allowed-tools: Bash, Read, Write, Glob, Grep
argument-hint: <issue_url_or_number> [--skip-test] [--draft]
---

# 通用 PR 提交流程

> 基于 gongkar-dev-flow 的 PR 提交规范，抽象为适用于任何项目的通用流程。
> 所有步骤按顺序执行，**展示给用户确认后**再执行 push/PR/留言操作。

---

## Step 0: 环境检测

自动检测项目信息，无需用户手动配置：

```
检测项：
- [ ] Git 仓库信息（remote、当前分支、base 分支）
- [ ] 项目语言 & 包管理器（package.json / go.mod / requirements.txt / Cargo.toml 等）
- [ ] Lint 工具（eslint / golangci-lint / ruff / clippy / 项目 Makefile 等）
- [ ] Format 工具（prettier / gofmt / black / rustfmt 等）
- [ ] 测试框架（jest / pytest / go test / cargo test 等）
- [ ] 项目文档文件（CLAUDE.md / README.md / CHANGELOG.md / docs/ 等）
- [ ] 关联 Issue 编号（从分支名、commit message、或用户参数中提取）
```

**检测结果输出示例：**
```
📋 项目检测结果：
- 语言: TypeScript (Node.js)
- Lint: eslint (via npm run lint)
- Format: prettier (via npm run format)
- 测试: jest (via npm test)
- Base 分支: main
- 当前分支: feat/add-user-api
- 关联 Issue: #42
```

---

## Step 1: 代码质量检查

### 1.1 Lint 检查
根据检测到的工具自动执行：

| 语言 | 命令 |
|------|------|
| Go | `go vet ./...` + `golangci-lint run`（如有） |
| TypeScript/JavaScript | `npm run lint` 或 `npx eslint .` |
| Python | `ruff check .` 或 `flake8` 或 `pylint` |
| Rust | `cargo clippy` |
| 其他 | 检查 Makefile 中的 `lint` target |

- [ ] 所有 lint 错误已修复
- [ ] 如有 lint 错误，**在当前分支修复后再继续**

### 1.2 Format 检查

| 语言 | 命令 |
|------|------|
| Go | `gofmt -l .` |
| TypeScript/JavaScript | `npx prettier --check .` |
| Python | `black --check .` 或 `ruff format --check .` |
| Rust | `cargo fmt --check` |
| 其他 | 检查 Makefile 中的 `format` target |

- [ ] 代码格式化通过
- [ ] 如有格式问题，自动修复并 commit

---

## Step 2: 测试

根据项目类型执行测试：

| 语言 | 命令 |
|------|------|
| Go | `go test ./...` |
| TypeScript/JavaScript | `npm test` |
| Python | `pytest` |
| Rust | `cargo test` |
| 其他 | 检查 Makefile 中的 `test` target |

- [ ] 测试通过
- [ ] 记录测试结果摘要（通过数、失败数、跳过数）

> 如使用 `--skip-test` 参数，跳过此步但在 PR 中标注 "测试已跳过"。

---

## Step 3: 文档同步检查

检查是否需要更新文档：

- [ ] 如有新增/修改 API → 检查 `docs/API.md` 或 API 文档是否需要更新
- [ ] 如有 CLAUDE.md → 检查 Recent Changes 是否需要更新
- [ ] 如有 CHANGELOG.md → 检查是否需要新增条目
- [ ] 如有 README.md → 检查是否需要更新（新增功能/配置变更时）
- [ ] 如有 migration 文件 → 确认已包含在 commit 中

**原则：** 只检查项目中已有的文档文件，不创建新文档。遗漏的更新必须在提交 PR 前修复。

---

## Step 4: Diff 审查 & 自检

在提交前进行最终审查：

```bash
# 查看完整 diff
git diff <base-branch>...HEAD

# 查看 commit 历史
git log <base-branch>..HEAD --oneline
```

自检清单：
- [ ] 没有遗留的 debug 代码（console.log / fmt.Println / print / dbg! 等）
- [ ] 没有包含敏感信息（.env / credentials / API keys 等）
- [ ] 没有不相关的改动混入
- [ ] commit message 清晰描述了改动内容

---

## Step 5: 展示 PR 预览给用户确认

**⚠️ 关键步骤：在执行任何远程操作前，必须先展示给用户确认。**

向用户展示以下内容并等待确认：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 PR 预览 — 请确认后提交
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔀 分支: feat/xxx → main
📌 关联 Issue: #XX

📋 PR 标题: <简洁标题，70字符以内>

📝 PR 描述:
## Summary
- 改动点1
- 改动点2

## Checklist
- [x] Lint — ✅ pass
- [x] Format — ✅ pass
- [x] Tests — ✅ pass (XX tests, 0 failures)
- [x] Rebase to latest <base-branch> — done
- [x] Docs updated (如适用)
- [x] CLAUDE.md updated (如适用)

## Test Evidence
<测试结果摘要，见 Step 6 格式>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
确认提交？(y/n)
```

**只有用户确认后才执行 Step 6 和 Step 7。**

---

## Step 6: Push & 创建 PR

用户确认后执行：

```bash
# 推送到远程
git push -u origin <current-branch>

# 创建 PR
gh pr create --title "<标题>" --body "<正文>"
```

PR 正文标准模板：

````markdown
## Summary
- 改动点简述（1-3 条）

## Checklist
- [x] Lint — ✅ pass
- [x] Format — ✅ pass  
- [x] Tests — ✅ pass (XX passed, 0 failed, X skipped)
- [x] Rebase to latest `<base-branch>` — done
- [ ] Docs — updated / N/A
- [ ] CLAUDE.md — updated / N/A
- [ ] CHANGELOG — updated / N/A

## Test Evidence

### 测试环境
- 环境描述（本地 / CI / staging）

### 测试结果
```
# 场景1: 描述
$ <测试命令>
→ 关键结果摘要 ✅

# 场景2: 描述  
$ <测试命令>
→ 关键结果摘要 ✅

# 场景3: 异常场景
$ <测试命令>
→ 预期错误信息 ✅
```

Closes #XX
````

**要求：**
- [ ] PR 标题 70 字符以内，清晰描述改动
- [ ] Checklist 中不适用的项标注 N/A，不要删除
- [ ] 如使用 `--draft` 参数，创建 Draft PR

---

## Step 7: Issue 留言（如有关联 Issue）

如果有关联的 GitHub Issue，将测试结果留言到 Issue：

```bash
gh issue comment <issue-number> --body "<留言内容>"
```

**留言模板：**

````markdown
## PR Test Results (PR #XXX)

### Checklist
- [x] Lint — ✅ pass
- [x] Format — ✅ pass
- [x] Tests — ✅ pass (XX passed, 0 failed, X skipped)
- [x] Rebase to latest `<base-branch>` — done
- [x] Docs — updated / N/A

### Test Evidence

```bash
# 场景1: 描述
$ <测试命令>
→ 关键结果摘要 ✅

# 场景2: 异常场景
$ <测试命令>  
→ 预期错误信息 ✅
```

PR: #XXX
````

**⚠️ 留言前也必须先展示给用户确认。**

---

## ⚠️ 强制规则

1. **Lint/Format 必须通过** — 不通过不提交
2. **测试必须通过** — 不通过不提交（除非 `--skip-test`）
3. **PR 正文必须包含 Checklist** — 不可省略
4. **远程操作前必须用户确认** — push、创建 PR、Issue 留言都要先展示后确认
5. **测试证据必须包含** — PR 和 Issue 留言中都要有测试结果
6. **不适用项标注 N/A** — Checklist 中的项不删除，不适用的标 N/A

---

## 使用方式

```bash
# 基本用法：提交 PR 并关联 Issue
/submit-pr #42

# 提交 PR 但跳过测试
/submit-pr #42 --skip-test

# 创建 Draft PR
/submit-pr #42 --draft

# 不关联 Issue
/submit-pr

# 直接传 Issue URL
/submit-pr https://github.com/owner/repo/issues/42
```

AI 助手收到后按流程逐步执行，每步完成后汇报进度，远程操作前等待用户确认。
