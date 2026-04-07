# anyone.skill — 产品需求文档 v1.0

---

## 一、产品概述

**anyone.skill** 是一个运行在 Claude Code 上的 meta-skill（元技能）。

用户通过对话式交互指定一个目标人物——真实公众人物、历史人物、虚构角色、或抽象人群——系统自动从互联网搜索语料并结合用户提供的材料，生成一个可独立运行的 **AI 化身 Skill**。

生成的 Skill 由两个独立部分组成：
- **Part A — Persona**：该人物的性格、世界观、沟通风格、行为模式
- **Part B — Expertise**：该人物的认知框架、专业知识、思维方式

两部分可以独立使用，也可以组合运行（默认组合）。生成后的 Skill 通过**记忆飞轮**机制持续进化——越对话越像，越纠正越准。

### 与 colleague-skill 的关系

colleague-skill 是 anyone-skill 的特化子集：colleague-skill 专注"你认识的同事"，数据来自企业内部；anyone-skill 覆盖"任何人"，数据来自公开互联网 + 用户上传。两者共享相同的底层人格模型架构。

---

## 二、目标用户与使用场景

### 2.1 目标用户

| 用户类型 | 典型需求 |
|---------|---------|
| 内容创作者 | 模拟 KOL 风格写文案、模拟历史人物对谈 |
| 教育从业者 | 创建历史人物 / 科学家 AI 供学生对话式学习 |
| 产品经理 / 用研 | 构建目标用户群体画像，进行模拟用户访谈 |
| 作家 / 编剧 | 创建虚构角色 AI 辅助剧情推演、对白生成 |
| 营销团队 | 模拟目标受众测试营销话术 |
| 个人用户 | 与偶像 / 历史偶像"对话"，娱乐与学习 |

### 2.2 核心场景

**场景 A — 公众人物化身：** 用户输入 "Elon Musk"，系统自动搜索 Twitter、访谈、演讲、新闻，提取人格特征，生成 Musk 化身。用户可以问"你怎么看 AI 监管？"并得到符合 Musk 风格的回答。

**场景 B — 历史人物对话：** 用户创建"孔子"化身，系统从论语、史记、学术研究中提取人格。用户可与之讨论现代伦理问题，孔子以其思想框架作答。

**场景 C — 虚构角色扮演：** 用户创建"Sherlock Holmes"，系统从原著 + 主流改编中提取人格，严格遵守世界观设定。用户可以给他一个谜题让他推理。

**场景 D — 群体画像模拟：** 用户创建"典型硅谷 VC"，系统综合多个 VC 的公开言论、投资逻辑、决策模式，生成群体共性画像。用户可以 pitch 项目给这个"VC"获得模拟反馈。

**场景 E — 自定义人物：** 用户手动描述一个不存在的人物（如"深圳工厂工作 10 年的 35 岁质检员"），通过引导式问答逐步丰富人格。

---

## 三、对象分类体系

### 3.1 四大对象类型

```
ObjectType
├── REAL_PUBLIC        # 真实公众人物（有大量公开语料）
│   ├── celebrity      # 明星、KOL、网红
│   ├── entrepreneur   # 企业家、创业者
│   ├── politician     # 政治人物
│   ├── scholar        # 学者、科学家
│   └── artist         # 艺术家、作家、导演
│
├── HISTORICAL         # 历史人物（语料有限，需学术推断）
│   ├── philosopher    # 哲学家、思想家
│   ├── scientist      # 历史科学家
│   ├── ruler          # 统治者、政治家
│   └── artist         # 历史艺术家
│
├── FICTIONAL          # 虚构角色（来自文学/影视/游戏）
│   ├── literary       # 文学角色
│   ├── film_tv        # 影视角色
│   ├── anime_manga    # 动漫角色
│   ├── game           # 游戏角色
│   └── mythological   # 神话 / 民间传说角色
│
└── ARCHETYPE          # 抽象人群（群体画像，非个体）
    ├── demographic    # 人口统计群体（Z 世代、银发族）
    ├── professional   # 职业群体（硅谷 VC、外卖骑手）
    ├── cultural       # 文化群体（二次元宅、嬉皮士）
    └── composite      # 复合群体（用户自定义交叉属性）
```

### 3.2 对象元数据

```yaml
meta:
  id: string                    # 唯一标识 e.g. "elon-musk-2026"
  name: string                  # 显示名
  type: ObjectType              # 四大类型之一
  subtype: string               # 子类型
  era: string | null            # 时代 "contemporary" | "historical:500BC" | "fictional:1890s"
  language_primary: string      # 主要语言
  source_richness: "abundant" | "moderate" | "scarce" | "none"
  worldview_id: string | null   # 虚构角色所属世界观 ID
  confidence_level: float       # 0.0-1.0，整体画像置信度
  maturity:                     # 记忆飞轮成熟度
    stage: "seed" | "growing" | "mature" | "master"
    slot_count: int             # 有效记忆槽位数
    correction_count: int       # 累计纠正次数
    reinforcement_count: int    # 累计强化次数
  created_at: datetime
  updated_at: datetime
  version: string
```

---

## 四、人格模型设计 — 6+1 层通用人格架构

colleague-skill 的 5 层模型面向"职场同事"。anyone-skill 需要处理截然不同的对象类型，升级为 **6 层核心 + 1 层条件层**。

### 4.1 架构总览

```
Layer 0: Override Guard        # 安全硬覆盖（不可绕过）
Layer 1: Identity Core         # 身份内核
Layer 2: Worldview & Belief    # 世界观与信念体系 ← 新增核心层
Layer 3: Expression Style      # 表达风格
Layer 4: Cognition & Expertise # 认知与专业 ← 升级自 Work Skill
Layer 5: Social & Relational   # 社交与关系模式
Layer 6: Correction Layer      # 纠正层

Conditional Layer: Context Adapter  # 条件适配器 ← 新增
```

### 4.2 各层设计

#### Layer 0: Override Guard（安全硬覆盖）

所有对象类型共享：
- 不生成非法内容
- 不冒充真人进行欺诈
- 明确声明"这是 AI 模拟，非真人"

按类型追加：
- **REAL_PUBLIC**：不生成可能被误认为真实声明的内容；争议话题标注"基于公开信息的模拟推断"
- **HISTORICAL**：标注历史推断的置信度；区分有史料支持 vs 合理推测
- **FICTIONAL**：尊重原著设定；不违反世界观规则
- **ARCHETYPE**：标注"群体典型特征，不代表个体"；避免强化有害刻板印象

#### Layer 1: Identity Core（身份内核）

- **所有类型**：姓名、别名、时代背景
- **REAL_PUBLIC / HISTORICAL**：生平事实（出生、国籍、教育、职业里程碑、以何闻名）
- **FICTIONAL**：叙事身份（来源作品、创作者、角色定位、角色弧线、正典版本）
- **ARCHETYPE**：群体身份（定义性特征、人口统计区间、文化语境、反例说明）

#### Layer 2: Worldview & Belief（世界观与信念）— 核心新增层

这是 anyone-skill 与 colleague-skill 最关键的差异。对公众人物、历史人物、虚构角色，世界观是人格的核心。

- **核心信念**：每条信念附带置信度和来源（如"技术是解决人类问题的核心手段"— 来自多次公开演讲，置信度 0.9）
- **价值观排序**：如"创新 > 稳定 > 传统"及其适用语境
- **认识论框架**：经验主义 / 理性主义 / 儒家天命观 / …
- **已知公开立场**（仅 REAL_PUBLIC）：具体议题 + 立场 + 来源 + 最后更新时间
- **虚构世界观**（仅 FICTIONAL）：世界规则、能力体系、社会结构、角色认知边界

#### Layer 3: Expression Style（表达风格）

- **语言特征**：词汇水平、句式结构、标志性用语、幽默风格
- **修辞特征**：偏好的修辞手法、论证方式
- **情感表达**：基线情绪、强度范围、触发条件
- **多媒介差异**（REAL_PUBLIC）：Twitter 风格 vs 访谈风格 vs 演讲风格 vs 写作风格

#### Layer 4: Cognition & Expertise（认知与专业）

升级自 colleague-skill 的 "Work Skill"，不局限于工作能力：

- **专业领域**：核心领域及深度（世界级 / 专业级 / 业余）
- **思维模式**：推理方式（第一性原理 / 类比推理 / 直觉判断）、决策方式、问题解决方式
- **认知盲区**：已知偏见和盲区
- **知识边界**：非常了解 / 有所了解 / 明确不了解的领域；不了解时是否会假装了解
- **时间锚定**：知识截止时间 + 时代错误处理策略（严格 / 感知 / 灵活）

#### Layer 5: Social & Relational（社交与关系模式）

- **人际风格**：热情度、支配性、正式度、被质疑时的反应
- **关系模式**：对权威 / 同级 / 下属 / 陌生人的不同态度
- **冲突处理**：直面 / 回避 / 幽默化解 / 压制
- **影响力策略**：逻辑说服 / 情感感召 / 权威施压
- **群体社交规范**（ARCHETYPE）：群内行为、对外群体态度、地位信号

#### Layer 6: Correction Layer（纠正层）

- 用户纠正记录：场景 + 错误行为 + 正确行为
- 置信度覆盖：可手动调整某个数据点的置信度
- 行为补丁：触发条件 + 响应规则，优先级最高
- 最多保留 50 条，超出后合并归纳

#### Conditional Layer: Context Adapter（条件适配器）

根据对话上下文动态调整行为的规则：

- **时间上下文**：讨论人物时代之后的事件 → 以其价值观评论，标注推测
- **角色上下文**：以角色身份给建议 → 从其认知框架出发，安全/健康议题可跳出角色
- **群体上下文**（ARCHETYPE）：个体视角 → 生成具体人物实例；群体视角 → 保持统计表述
- **深度上下文**：超出知识边界 → 诚实承认

### 4.3 各对象类型的层激活矩阵

| 层 | REAL_PUBLIC | HISTORICAL | FICTIONAL | ARCHETYPE |
|----|-----------|-----------|----------|----------|
| L0 Override | 完整 | 完整 | 完整 | 完整 |
| L1 Identity | 完整 | 完整 | 叙事身份 | 群体身份 |
| L2 Worldview | 完整+公开立场 | 完整（推断标注） | 虚构世界观 | 简化（群体共识） |
| L3 Expression | 完整+多媒介 | 基于文献推断 | 基于原著台词 | 群体语言特征 |
| L4 Cognition | 完整 | 基于著作推断 | 基于剧情推断 | 群体认知倾向 |
| L5 Social | 完整 | 有限推断 | 基于角色关系 | 群体社交规范 |
| L6 Correction | 完整 | 完整 | 完整 | 完整 |
| CL Adapter | 完整 | +时间穿越规则 | +世界观边界 | +个体/群体切换 |

### 4.4 记忆槽位存储结构

6+1 层模型的底层存储不是扁平文本，而是 **128 个动态记忆槽位（Memory Slots）**。每个槽位是人格画像的最小可编辑单元。

**槽位分类：**

| 类别 | 对应层 | 槽位示例 |
|------|-------|---------|
| identity | L1 | `identity.birthplace`, `identity.career_milestones` |
| worldview | L2 | `worldview.core_belief.tech_optimism`, `worldview.position.ai_regulation` |
| expression | L3 | `expression.catchphrase`, `expression.humor_style`, `expression.emoji_habit` |
| cognition | L4 | `cognition.reasoning_style`, `cognition.blind_spots`, `cognition.expertise.rocketry` |
| emotion | L3+L5 | `emotion.baseline_tone`, `emotion.trigger.被质疑`, `emotion.stress_response` |
| social | L5 | `social.with_authority`, `social.conflict_style`, `social.influence_tactic` |

**单个槽位结构（在 persona.yaml 中）：**

```yaml
slots:
  expression.catchphrase:
    content: "哎哟不错哦"
    confidence: 0.92
    source: "多个访谈视频 (Tier 2)"
    reinforced_count: 7        # 被用户行为确认的次数
    last_activated: "2026-04-05"
    
  worldview.position.ai_regulation:
    content: "反对过度监管，认为会扼杀创新"
    confidence: 0.85
    source: "Twitter 发言 + 国会听证会 (Tier 1)"
    reinforced_count: 3
    last_activated: "2026-04-03"
```

**槽位与层的关系：**
- Layer 0（Override Guard）不使用槽位 — 硬编码在 system_prompt 中
- Layer 1-5 的所有数据点都存为独立槽位
- Layer 6（Correction）的纠正会直接修改对应槽位的 content 和 confidence
- Conditional Layer 是规则，不是数据，硬编码在 system_prompt 中

---

## 五、数据采集方案

### 5.1 语料分级体系

```
Tier 1: 第一手语料（本人直接产出）
  - 社交媒体帖文（Twitter/X, 微博, 小红书, LinkedIn）
  - 个人博客 / 专栏
  - 书籍 / 著作
  - 演讲稿 / 演讲视频转录
  - 播客转录
  - 公开信 / 声明

Tier 2: 高可信二手语料
  - 经审核的访谈
  - 官方传记
  - 维基百科
  - 学术论文（关于历史人物）
  - 原著文本（虚构角色）

Tier 3: 一般参考语料
  - 新闻报道
  - 第三方分析 / 评论
  - 非官方传记
  - 改编作品（虚构角色）

Tier 4: 用户提供语料
  - 手动文本输入
  - 文件上传
  - URL 指定
  - 问答式输入
```

### 5.2 自动采集流程

利用 `WebSearch` + `WebFetch` 实现：

```
Step 1: 初始搜索（WebSearch）
  按对象类型生成搜索策略：
  - REAL_PUBLIC: "{name} interview quotes" / "{name} speech transcript" / "{name} twitter"
  - HISTORICAL: "{name} primary sources" / "{name} 原著" / "{name} 学术研究"
  - FICTIONAL: "{character} {source_work} quotes" / "{character} character analysis"
  - ARCHETYPE: "{group} typical behavior study" / "{group} 调研报告"

Step 2: 页面获取（WebFetch）
  - 抓取搜索结果中的高质量页面
  - 提取正文，过滤导航/广告
  - 标注来源和 Tier 等级

Step 3: 语料过滤与去重
  - 去除重复 / 低质量 / 不相关内容
  - 标注时间戳
  - 计算语料覆盖度（各层数据是否充足）

Step 4: 人格提取（Analyzer Prompt）
  - 输入：原始语料
  - 输出：各层结构化人格数据
  - 标注每个提取点的来源和置信度
```

### 5.3 用户上传支持

| 格式 | 处理方式 |
|------|---------|
| `.txt` / `.md` | 直接文本解析 |
| `.pdf` | Claude PDF 工具 |
| `.docx` | 提示用户转 PDF |
| `.srt` / `.vtt` | 字幕文件解析 |
| URL | WebFetch 抓取 |
| 手动输入 | 直接录入 |

---

## 六、用户流程

### 6.1 创建流程

```
用户: /anyone create "Elon Musk"
        │
        ▼
┌──────────────┐
│ Intake Phase │  识别对象类型、确认基本信息
│              │  "检测到 Elon Musk 是当代公众人物（REAL_PUBLIC/entrepreneur）"
│              │  "确认创建？有什么特别侧重点？"
└──────┬───────┘
       │ 用户确认
       ▼
┌──────────────┐
│ Gather Phase │  自动搜索 + 用户补充
│              │  WebSearch → WebFetch → 语料收集
│              │  "已收集 47 条语料，覆盖：身份 ✓ 世界观 ✓ 表达风格 ✓
│              │   认知 ✓ 社交 △（数据不足）"
│              │  "是否补充社交方面的语料？或跳过继续？"
└──────┬───────┘
       ▼
┌──────────────┐
│ Analyze Phase│  人格提取
│              │  语料 → Analyzer Prompt → 结构化数据
│              │  标注每项的置信度和来源
└──────┬───────┘
       ▼
┌──────────────┐
│ Build Phase  │  生成完整画像
│              │  Builder Prompt → 6+1 层 persona.yaml
│              │  System Prompt 生成
└──────┬───────┘
       ▼
┌──────────────┐
│ Review Phase │  用户确认
│              │  展示画像摘要，用户可修改
└──────┬───────┘
       │ 确认
       ▼
┌──────────────┐
│ Save & Done  │  保存文件，输出可用的对话入口
└──────────────┘
```

### 6.2 对话流程（主动记忆驱动）

```
用户: /anyone chat "elon-musk-2026"
  → 加载 base_prompt（L0 安全 + L1 身份固定部分）
  → 进入对话模式
  
每轮对话：
  → 用户发送消息
  → 主动记忆选择：从 128 槽位中选取 Top 10-15 最相关槽位
    （基于话题匹配 + 置信度 + reinforced_count 排序）
  → 拼装上下文：base_prompt + 动态槽位 + 纠正补丁 + 对话历史
  → 生成回复
  
对话后处理（飞轮转动）：
  → 隐式强化：用户继续深聊 → 当前活跃槽位 reinforced_count +1
  → 用户纠正 → 更新槽位 → 重新生成
  → 对话结束时写回 persona.yaml
```

### 6.3 进化流程

```
用户: /anyone evolve "elon-musk-2026"
  → 搜索最新语料（上次更新之后的）
  → 用户可提供新材料
  → 增量分析 → 合并（Merger Prompt）
  → 冲突检测与解决
  → 展示变更 diff，用户确认或回滚
```

### 6.4 命令清单

| 命令 | 说明 |
|------|------|
| `/anyone create <name>` | 创建新化身 |
| `/anyone chat <id>` | 与化身对话 |
| `/anyone evolve <id>` | 增量进化 |
| `/anyone correct <id>` | 手动纠正 |
| `/anyone list` | 列出所有化身 |
| `/anyone inspect <id>` | 查看画像详情 |
| `/anyone export <id>` | 导出画像 |
| `/anyone import <file>` | 导入画像 |
| `/anyone compare <id1> <id2>` | 对比两个化身 |
| `/anyone merge <id1> <id2>` | 合并两个画像 |
| `/anyone delete <id>` | 删除化身 |

---

## 七、生成内容规范

### 7.1 文件结构

```
personas/<persona_id>/
├── meta.yaml                  # 对象元数据（含成熟度）
├── persona.yaml               # 记忆槽位存储（128 slots，按类别组织）
├── base_prompt.md             # 固定 System Prompt（L0 安全 + L1 身份 + CL 规则）
├── sources/
│   └── manifest.yaml          # 语料清单（来源、Tier、时间）
├── corrections/
│   └── corrections.yaml       # 纠正记录（含槽位映射）
├── flywheel/
│   └── reinforcement_log.yaml # 强化日志（哪些槽位被强化、何时、多少次）
└── versions/                  # 版本历史
    ├── v1.0.0/
    └── v1.1.0/
```

### 7.2 双文件 Prompt 架构

每个化身的 Prompt 由两部分构成，对话时动态拼装：

**固定部分 — `base_prompt.md`（一次生成，很少变动）：**

```
## Safety（Layer 0 — 硬编码）
[安全规则，按对象类型不同]
"This is an AI simulation, not the real person."

## Identity（Layer 1 核心 — 压缩版）
[姓名、时代、基本身份，~200 tokens]

## Context Rules（Conditional Layer — 硬编码）
[时间上下文/角色上下文/群体上下文等规则]

## Memory-Driven Response Protocol
当你收到动态记忆槽位时，严格按以下优先级响应：
1. is_core: true 的槽位 → 必须体现
2. reinforced_count > 5 的槽位 → 强烈倾向体现
3. 其余槽位 → 作为参考
```

**动态部分 — 每轮对话由 `memory_selector.md` 生成：**

```
## Active Memory Context（本轮动态注入）

[expression.catchphrase] (confidence: 0.92, reinforced: 7, core: true)
标志性用语："哎哟不错哦"

[cognition.reasoning_style] (confidence: 0.88, reinforced: 5)
推理方式：直觉驱动，喜欢用音乐类比解释事物

[worldview.position.华语乐坛] (confidence: 0.80, reinforced: 3)
观点：认为年轻人创作力有但太急，音乐要慢慢磨

... (共 10-15 个槽位)
```

### 7.3 组合运行规则

```
用户提问
  ↓
memory_selector 选取相关槽位
  ↓
拼装：base_prompt + 动态槽位 + 纠正补丁 + 对话历史
  ↓
Persona 层（emotion/social 槽位）判断：愿不愿意回答？态度如何？
  ↓
Expertise 层（cognition 槽位）执行：用对应的认知框架回答
  ↓
Expression 层（expression 槽位）润色：用对应的语言风格输出
  ↓
对话后处理：隐式强化活跃槽位
```

---

## 八、记忆飞轮机制

核心理念（借鉴 Elys）：**"一个人的灵魂 = 其所有上下文的总和"**。化身不是一次生成的静态文件，而是通过记忆飞轮持续进化的动态实体。

### 8.1 飞轮循环

```
用户对话行为 → 被动记忆沉淀 → 主动记忆调取 → AI 生成回复
      ↑                                          │
      └──── 用户确认/纠正 ← 反馈强化 ←────────────┘
```

每一轮对话都在转动飞轮：对话产生记忆 → 记忆提升下次回复质量 → 更好的回复引发更多对话 → 更多对话产生更多记忆。

### 8.2 双层记忆

| 类型 | 机制 | Skill 实现方式 |
|------|------|--------------|
| **被动记忆** | 用户问什么，检索什么 | `memory_selector.md` 根据用户消息关键词匹配槽位 |
| **主动记忆** | 系统主动调取当前对话最相关的上下文，不等用户触发 | `memory_selector.md` 分析对话意图，预判需要哪些槽位（如聊到创业 → 主动调取"第一性原理思维"+"对失败的态度"） |

**为什么主动记忆比全量灌入好：**

- **省 token**：不是每次都灌入 128 个槽位，只选 10-15 个
- **更准确**：只有高相关上下文，LLM 不会被无关信息干扰
- **更一致**：精准投喂 → 人格表现更稳定

### 8.3 主动记忆选择流程

```
用户消息 + 最近 3 轮对话
    │
    ▼
memory_selector.md 执行：
1. 意图识别 → 话题域（技术/情感/观点/闲聊/创作）
2. 槽位匹配 → 从 128 个槽位中候选
3. 排序规则：
   - 话题相关性（最高权重）
   - reinforced_count 高的优先（被验证过的更可靠）
   - confidence 高的优先
   - last_activated 近的优先（近期活跃的更相关）
4. 输出 Top 10-15 个槽位 → 注入对话上下文
    │
    ▼
拼装 System Prompt：
  base_prompt.md（固定 ~500 tokens）
  + 动态槽位（~1500-2500 tokens）
  + 相关纠正补丁（~200 tokens）
  + 对话历史
  + 用户当前输入
```

### 8.4 化身成熟度

| 阶段 | 有效槽位数 | 化身表现 | 标记 |
|------|----------|---------|------|
| **Seed（初生）** | 0-20 | 基于初始语料，回复偏模板化 | 🌱 |
| **Growing（成长）** | 20-50 | 能抓住核心特征，偶尔偏差 | 🌿 |
| **Mature（成熟）** | 50-100 | 高度拟真，能在新场景下合理推断 | 🌳 |
| **Master（大师）** | 100+ | 深层认知和情感模式都被捕获 | ⭐ |

成熟度写入 `meta.yaml`，`/anyone inspect` 时展示。

### 8.5 隐式强化规则

不需要用户主动纠正，对话行为本身就在强化记忆：

| 用户行为 | 推断 | 记忆操作 |
|---------|------|---------|
| 继续深聊同一话题 | 回复风格正确 | 当前活跃槽位 reinforced_count +1 |
| 用户说"对""没错""就是这样" | 明确正向确认 | 当前槽位 reinforced_count +2 |
| 用户纠正 "这不像 TA" | 回复偏离人格 | 定位槽位，更新 content，旧值 confidence -0.3 |
| 切换话题 / 短回复 | 可能无关 | 不做负面推断（避免误判） |

### 8.6 记忆整理（Memory Consolidation）

每次 `/anyone evolve` 或累计 50 次强化后触发 `memory_consolidator.md`：

```
1. 合并：内容相似的槽位合并（如两个关于"说话风格"的槽位）
2. 升级：reinforced_count > 10 且 confidence > 0.9 的槽位标记为 is_core
3. 降权：confidence < 0.3 且 last_activated > 30 天的槽位降为候选删除
4. 补缺：分析 6 类记忆的覆盖率，标注薄弱维度供用户补充
5. 置信度衰减（仅 REAL_PUBLIC）：
   - 30 天未刷新 → confidence × 0.9
   - 90 天未刷新 → confidence × 0.75
   - 180 天未刷新 → 标记"建议更新"
```

### 8.7 纠正处理

```
用户: "他不会这样说话，他更直接"
  ↓
correction_handler.md 执行：
  1. 识别纠正对应的记忆槽位（如 expression.formality）
  2. 记录纠正：场景 + 旧值 + 新值 → corrections.yaml
  3. 更新槽位：content 替换 + confidence 重置为 0.7（待后续强化验证）
  4. 立即生效：用更新后的槽位重新生成回复
  5. 成熟度评估：检查是否触发阶段升级
```

### 8.8 增量合并（Merger）

`/anyone evolve` 时执行 `merger.md`：

- 新语料提取 → 生成候选槽位
- 与已有槽位对比：补充 / 确认 / 矛盾
- 补充 → 创建新槽位
- 确认 → 已有槽位 reinforced_count +1
- 矛盾 → 标记冲突，询问用户（高 Tier 优先，用户纠正最高优先级）
- 版本存档 → `versions/` 目录

### 8.9 版本管理

- 每次 evolve 或重大纠正自动存档到 `versions/`
- 保留最近 10 个版本
- `/anyone rollback <id> <version>` 支持回滚

### 8.10 语料时效性

| 对象类型 | 刷新建议 | 过期警告 |
|---------|---------|---------|
| REAL_PUBLIC | 30 天 | 90 天（触发置信度衰减） |
| HISTORICAL | 不需要 | — |
| FICTIONAL | 新作品发布时 | — |
| ARCHETYPE | 90 天 | 180 天 |

---

## 九、Prompt 模板设计

```
prompts/
├── intake.md                  # 意图识别 + 对象分类
├── searcher.md                # 搜索策略生成
├── analyzer/                  # 语料分析（按类型）
│   ├── analyzer_base.md       # 通用分析框架
│   ├── analyzer_real.md       # 真实公众人物
│   ├── analyzer_historical.md # 历史人物
│   ├── analyzer_fictional.md  # 虚构角色
│   └── analyzer_archetype.md  # 群体画像
├── builder/                   # 画像构建（按类型）
│   ├── builder_base.md
│   ├── builder_real.md
│   ├── builder_historical.md
│   ├── builder_fictional.md
│   └── builder_archetype.md
├── memory_selector.md         # ← 新增：主动记忆选择（飞轮核心）
├── memory_consolidator.md     # ← 新增：记忆整理与合并
├── merger.md                  # 增量合并
├── correction_handler.md      # 纠正处理（升级：含槽位定位）
├── system_prompt_generator/   # Base Prompt 生成
│   ├── sysgen_real.md
│   ├── sysgen_historical.md
│   ├── sysgen_fictional.md
│   └── sysgen_archetype.md
├── quality_checker.md         # 画像质量检查（升级：含槽位覆盖率）
└── conflict_resolver.md       # 冲突解决
```

各 Prompt 职责：

| Prompt | 输入 | 输出 | 职责 |
|--------|------|------|------|
| intake | 用户自然语言 | 对象类型 + 元数据 | 识别意图、判断类型、引导补充信息 |
| searcher | 元数据 | 搜索查询列表 | 生成最优搜索策略 |
| analyzer_* | 原始语料 | 结构化槽位数据 | 从语料提取各层数据，输出为记忆槽位格式，标注来源和置信度 |
| builder_* | 分析结果 | persona.yaml（槽位化） | 组装完整画像为 128 槽位结构，填补空白，标注推断 |
| **memory_selector** | 用户消息 + 近 3 轮对话 + 全部槽位 | Top 10-15 槽位 | **飞轮核心**：意图识别 → 话题域判断 → 槽位匹配排序 → 输出动态上下文 |
| **memory_consolidator** | 全部槽位 + 强化日志 | 整理后的槽位 + 报告 | 合并重复槽位、升级核心记忆、降权过期槽位、标注薄弱维度 |
| merger | 旧槽位 + 新语料槽位 | 合并后槽位 + diff | 智能合并、冲突检测、reinforced_count 累加 |
| correction_handler | 用户纠正 + 当前槽位 | 更新后槽位 | 定位纠正对应的槽位 → 更新 content → 重置 confidence |
| sysgen_* | persona.yaml | base_prompt.md | 生成固定部分（L0 + L1 身份 + CL 规则） |
| quality_checker | persona.yaml | 质量报告 | 检查槽位覆盖率、置信度分布、成熟度评估、薄弱维度 |
| conflict_resolver | 冲突数据 | 解决建议 | 分析冲突原因，生成选项 |

---

## 十、项目结构

```
anyone-skill/
├── SKILL.md                       # 技能入口
├── README.md                      # 项目说明
├── CLAUDE.md                      # Claude Code 项目配置
│
├── prompts/                       # Prompt 模板（详见第九节）
│   ├── intake.md
│   ├── searcher.md
│   ├── analyzer/
│   ├── builder/
│   ├── memory_selector.md         # 主动记忆选择（飞轮核心）
│   ├── memory_consolidator.md     # 记忆整理与合并
│   ├── merger.md
│   ├── correction_handler.md
│   ├── system_prompt_generator/
│   ├── quality_checker.md
│   └── conflict_resolver.md
│
├── schemas/                       # 数据 Schema
│   ├── meta.schema.yaml
│   ├── persona.schema.yaml
│   ├── correction.schema.yaml
│   └── source_manifest.schema.yaml
│
├── templates/                     # 输出模板
│   ├── persona_template.yaml
│   └── system_prompt/
│       ├── real.md.tmpl
│       ├── historical.md.tmpl
│       ├── fictional.md.tmpl
│       └── archetype.md.tmpl
│
├── examples/                      # 示例画像（开箱即用）
│   ├── elon-musk/
│   ├── confucius/
│   ├── sherlock-holmes/
│   └── silicon-valley-vc/
│
├── docs/
│   ├── PRD.md                     # 本文档
│   └── ethics.md                  # 伦理指南
│
└── test/
    ├── fixtures/
    └── prompt_tests/
```

运行时用户数据目录：

```
~/.anyone-skill/
├── config.yaml                    # 用户配置
├── personas/                      # 所有已创建的化身
│   └── <persona_id>/
│       ├── meta.yaml              # 含成熟度
│       ├── persona.yaml           # 128 记忆槽位
│       ├── base_prompt.md         # 固定 System Prompt
│       ├── sources/
│       ├── corrections/
│       ├── flywheel/              # 强化日志
│       └── versions/
└── cache/                         # 搜索缓存
```

---

## 十一、实现优先级

### P0 — 核心可用（MVP）

- [ ] `SKILL.md` + `CLAUDE.md` 基础骨架
- [ ] `prompts/intake.md` 对象类型识别
- [ ] `persona.yaml` Schema 定义（128 槽位结构）
- [ ] REAL_PUBLIC 完整流程：搜索 → 分析 → 构建槽位 → 生成 base_prompt
- [ ] `/anyone create` + `/anyone chat` 基本命令
- [ ] `prompts/memory_selector.md` 主动记忆选择（飞轮核心）
- [ ] 隐式强化机制（对话后更新 reinforced_count）
- [ ] 成熟度 4 阶段计算与展示
- [ ] `prompts/correction_handler.md` 纠正 → 槽位更新
- [ ] 4 个示例画像（每种类型 1 个，预调教到 🌳 成熟阶段）

### P1 — 完整体验

- [ ] HISTORICAL 完整流程
- [ ] FICTIONAL 完整流程（含世界观层）
- [ ] ARCHETYPE 完整流程（群体画像）
- [ ] `/anyone evolve` 增量进化 + `merger.md` 槽位合并
- [ ] `prompts/memory_consolidator.md` 记忆整理
- [ ] Conditional Layer 实现
- [ ] 版本管理
- [ ] 用户文件上传（PDF/TXT）
- [ ] `quality_checker.md`（含槽位覆盖率分析）

### P2 — 进阶功能

- [ ] `/anyone compare` 画像对比
- [ ] `/anyone merge` 画像合并
- [ ] `/anyone export` / `/anyone import`
- [ ] 语料时效性 + 置信度衰减
- [ ] `conflict_resolver.md`
- [ ] 完整测试套件
- [ ] 更多示例画像（10+）

### P3 — 远期愿景

- [ ] 社区画像共享（Gallery）
- [ ] 多化身对话（让两个化身对谈）
- [ ] 自动定期刷新（scheduled evolve）
- [ ] 与 colleague-skill 互操作

---

## 十二、约束与边界

### 12.1 法律风险

| 风险 | 缓解措施 |
|------|---------|
| 肖像权 / 人格权 | Layer 0 强制声明"AI 模拟，非真人"；不生成可被误认为真实声明的内容 |
| 版权 | 虚构角色画像不复制原文，只提取特征；标注原著来源 |
| 诽谤 | 所有公开立场必须有来源标注；争议话题标注"基于公开信息的推断" |
| 隐私 | 只使用公开信息；不采集私人社交账号 |

### 12.2 伦理红线

1. **不冒充真人** — 所有交互必须有 AI 模拟声明
2. **不强化刻板印象** — ARCHETYPE 必须包含反例和个体差异声明
3. **不美化 / 丑化** — 忠于语料，不添加价值判断
4. **不模拟私密场景** — 不模拟真实人物的私人对话、亲密关系
5. **尊重逝者** — 历史人物画像标注学术推断性质

### 12.3 技术限制

| 限制 | 应对 |
|------|------|
| WebSearch / WebFetch 速率 | 缓存层；分批采集；优先高价值源 |
| LLM 上下文窗口 | 分块分析 → 合并；摘要 + 原文双轨 |
| 语料质量不均 | confidence_level 标注；鼓励用户补充 |
| 多语言 | 标注主要语言；保留原文关键表达 |
| 原著版权保护 | 依赖公开摘要 / 评论；用户上传自有内容 |

### 12.4 明确不做的事

- 不做语音 / 视频克隆 — 只做文本人格模拟
- 不做实时社交媒体监控 — 只在用户触发时搜索
- 不做付费墙内容抓取 — 只使用公开可访问内容
- 不保证 100% 准确 — 始终标注存在误差
