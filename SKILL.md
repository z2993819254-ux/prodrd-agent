---
name: anyone
description: "Distill anyone into an AI persona — public figures, historical figures, fictional characters, or archetypes. 将任何人蒸馏为可对话、可进化的 AI 化身。"
argument-hint: "[create|chat|evolve|correct|list|inspect|delete] [name-or-id]"
version: "1.0.0"
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, Agent
---

# anyone.skill — 捏出任何人，和 TA 对话

## 触发命令

| 命令 | 说明 |
|------|------|
| `/anyone create <name>` | 创建新化身 |
| `/anyone chat <id>` | 与化身对话 |
| `/anyone evolve <id>` | 增量进化（追加语料） |
| `/anyone correct <id>` | 手动纠正人格 |
| `/anyone list` | 列出所有化身 |
| `/anyone inspect <id>` | 查看画像详情与成熟度 |
| `/anyone delete <id>` | 删除化身 |

## 核心机制

本技能使用 **记忆飞轮** 驱动化身成长：

```
用户对话 → 被动记忆沉淀 → 主动记忆调取 → AI 生成回复
    ↑                                        │
    └── 用户确认/纠正 ← 反馈强化 ←──────────┘
```

每个化身拥有最多 **128 个记忆槽位**，分为 6 类（identity / worldview / expression / cognition / emotion / social）。对话时不是全量灌入，而是由 `memory_selector.md` 动态选取最相关的 10-15 个槽位注入上下文。

## 执行流程

> **编排指令：** 收到 `/anyone` 命令后，先解析子命令和参数，然后读取对应的 `commands/{子命令}.md` 文件，严格按其中的步骤顺序执行。每一步必须完成才能进入下一步。

### `/anyone create <name>`

**详细编排 → 读取并执行 `commands/create.md`**

9 步流程概览：
1. **Intake** — 读取 `prompts/intake.md`，识别对象类型
2. **Search** — 读取 `prompts/searcher.md`，生成搜索策略
3. **Fetch** — 用 WebSearch + WebFetch 并行采集语料（≤ 20 页）
4. **Analyze** — 读取 `prompts/analyzer/analyzer_base.md` + `analyzer_{type}.md`，提取记忆槽位
5. **Build** — 读取 `prompts/builder/builder_base.md` + `builder_{type}.md`，组装 persona.yaml
6. **SysGen** — 读取 `prompts/system_prompt_generator/sysgen_{type}.md`，生成 base_prompt.md
7. **Quality** — 读取 `prompts/quality_checker.md`，检查质量与成熟度
8. **Save** — 保存完整文件到 `~/.anyone-skill/personas/<id>/`
9. **Present** — 展示画像摘要给用户

### `/anyone chat <id>`

1. 加载 `base_prompt.md`（固定部分）
2. 每轮对话：
   a. 读取 `prompts/memory_selector.md`，从 persona.yaml 选取 Top 10-15 相关槽位
   b. 拼装上下文：base_prompt + 动态槽位 + 纠正补丁 + 对话历史
   c. 生成回复
3. 对话后处理：
   a. 隐式强化：更新活跃槽位的 reinforced_count
   b. 用户纠正 → 执行 correction_handler.md → 更新槽位 → 重新生成
   c. 写回 persona.yaml

### `/anyone evolve <id>`

1. 搜索最新语料 / 接收用户上传
2. 读取 `prompts/analyzer/analyzer_*.md` 分析新语料
3. 读取 `prompts/merger.md` 与现有槽位合并
4. 读取 `prompts/memory_consolidator.md` 整理记忆
5. 版本存档 → 展示变更 diff

### `/anyone correct <id>`

**详细编排 → 读取并执行 `commands/correct.md`**

6 步流程概览：
1. **理解纠正意图** — 读取 `prompts/correction_handler.md`，提取 category + what_was_wrong + what_should_be
2. **定位目标槽位** — 在 persona.yaml 中找到或新建槽位
3. **判断策略** — 通用纠正（覆盖）or 场景化纠正（新建子槽位）
4. **执行更新** — 更新 persona.yaml + 写入 corrections.yaml
5. **重新生成** — 对话中触发时重新生成回复
6. **成熟度更新** — correction_count +1

## 成熟度

| 阶段 | 槽位数 | 标记 |
|------|-------|------|
| Seed | 0-20 | 🌱 |
| Growing | 20-50 | 🌿 |
| Mature | 50-100 | 🌳 |
| Master | 100+ | ⭐ |

## 数据目录

```
~/.anyone-skill/personas/<id>/
├── meta.yaml
├── persona.yaml          # 128 记忆槽位
├── base_prompt.md        # 固定 System Prompt
├── sources/manifest.yaml
├── corrections/corrections.yaml
├── flywheel/reinforcement_log.yaml
└── versions/
```

## 注意事项

- 所有化身必须声明 "AI 模拟，非真人"
- 不模拟真实人物的私密场景
- 虚构角色严格遵守原著世界观
- 群体画像标注"不代表个体"
