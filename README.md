# anyone.skill

捏出任何人，和 TA 对话。越聊越像，记忆飞轮驱动化身成长。

## 这是什么

anyone-skill 是一个 Claude Code 元技能（meta-skill）。用户输入任意人名或描述，系统自动搜索互联网语料，构建该人物的 AI 化身。

支持四类人物：
- **真实公众人物** — Elon Musk、周杰伦、马云
- **历史人物** — 孔子、爱因斯坦、苏格拉底
- **虚构角色** — Sherlock Holmes、孙悟空、哈利波特
- **抽象人群** — 典型硅谷 VC、00 后大学生、东北大姨

## 核心机制

### 记忆飞轮

```
用户对话 → 记忆沉淀 → 主动调取 → 生成回复
    ↑                                  │
    └── 确认/纠正 ← 反馈强化 ←────────┘
```

每个化身拥有最多 **128 个记忆槽位**，分为 6 类。对话时不是全量灌入，而是动态选取最相关的 10-15 个槽位注入上下文。

### 成熟度

| 阶段 | 槽位数 | 表现 |
|------|-------|------|
| 🌱 Seed | 0-20 | 偏模板化 |
| 🌿 Growing | 20-50 | 能抓住核心特征 |
| 🌳 Mature | 50-100 | 高度拟真 |
| ⭐ Master | 100+ | 深层认知都被捕获 |

## 快速开始

```bash
# 创建化身
/anyone create "Elon Musk"

# 与化身对话
/anyone chat elon-musk

# 纠正人格
# （对话中直接说 "这不像他" 即可触发）

# 增量进化
/anyone evolve elon-musk

# 查看详情
/anyone inspect elon-musk
```

## 命令

| 命令 | 说明 |
|------|------|
| `/anyone create <name>` | 创建新��身 |
| `/anyone chat <id>` | 与化身对话 |
| `/anyone evolve <id>` | 增量进化 |
| `/anyone correct <id>` | 手动纠正 |
| `/anyone list` | 列出所有化身 |
| `/anyone inspect <id>` | 查看画像详情 |
| `/anyone delete <id>` | 删除化身 |

## 项目结构

```
anyone-skill/
├── SKILL.md              # 技能入口
├── CLAUDE.md             # Claude Code 配置
├── prompts/              # Prompt 模��
│   ├── intake.md         # 对象识别
│   ├── searcher.md       # 搜索策略
│   ├── analyzer/         # 语料分析（4 类型）
│   ├���─ builder/          # 画像构建（4 类型）
│   ���── memory_selector.md    # 主动记忆选择（飞轮核心）
���   ├── memory_consolidator.md # 记忆整理
│   ├── merger.md         # 增量合并
│   ├── correction_handler.md  # 纠正处理
│   ├── system_prompt_generator/ # base_prompt 生成
│   ├── quality_checker.md     # 质量检查
│   └── conflict_resolver.md   # 冲突解决
├── schemas/              # 数据 Schema
├── examples/             # 示例画像
│   ├── elon-musk/        # REAL_PUBLIC 示例
│   ���── confucius/        # HISTORICAL 示例
│   ├── sherlock-holmes/   # FICTIONAL 示���
│   └── silicon-valley-vc/ # ARCHETYPE 示例
└── docs/
    └── PRD.md            # 产品需求文档
```

## 示例画像

开箱即用的 4 个示例，全部预调教到 🌳 Mature 阶段：

| 化身 | 类型 | 槽位数 | 语言 |
|------|------|-------|------|
| Elon Musk | REAL_PUBLIC | 62 | EN |
| 孔子 | HISTORICAL | 55 | ZH |
| Sherlock Holmes | FICTIONAL | 58 | EN |
| 典型硅谷 VC | ARCHETYPE | 52 | EN |

## 许可

MIT License
