# anyone-skill — Claude Code 项目配置

## 项目概述

anyone-skill 是一个 Claude Code 元技能，让用户将任何人蒸馏为可对话的 AI 化身。核心是 128 记忆槽位 + 记忆飞轮机制。

## 关键路径

- 技能入口: `SKILL.md`
- Prompt 模板: `prompts/`
- Schema 定义: `schemas/`
- 示例画像: `examples/`
- 用户数据: `~/.anyone-skill/personas/`

## 开发约定

- 所有 prompt 模板使用 Markdown 格式
- persona.yaml 使用槽位化结构（每个槽位含 content/confidence/source/reinforced_count/last_activated）
- base_prompt.md 只包含固定部分（L0 安全 + L1 身份 + CL 条件规则）
- 动态上下文由 memory_selector.md 在对话时实时生成
- 示例画像必须预调教到 Mature（🌳）阶段（50+ 槽位）

## 对象类型

- REAL_PUBLIC: 真实公众人物
- HISTORICAL: 历史人物
- FICTIONAL: 虚构角色
- ARCHETYPE: 抽象人群

## 命令

```
/anyone create <name>    # 创建化身
/anyone chat <id>        # 对话
/anyone evolve <id>      # 进化
/anyone correct <id>     # 纠正
/anyone list             # 列表
/anyone inspect <id>     # 详情
/anyone delete <id>      # 删除
```

## 编排文件

命令的详细编排逻辑在 `commands/` 目录：

- `commands/create.md` — `/anyone create` 的 9 步编排流程（Intake → Search → Fetch → Analyze → Build → SysGen → Quality → Save → Present）
- `commands/correct.md` — `/anyone correct` 的 6 步纠正流程（含对话中隐式触发 + 场景化纠正）

SKILL.md 作为入口路由，收到命令后读取对应的 `commands/{子命令}.md` 执行。

## Recent Changes

- 2026-04-07: 实现 `/anyone correct` 纠正处理（#3）— 新增 `commands/correct.md`，支持通用纠正 + 场景化纠正
- 2026-04-06: 实现 `/anyone create` 编排流程（#1）— 新增 `commands/create.md`，更新 SKILL.md 路由机制
