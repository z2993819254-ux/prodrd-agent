# Analyzer — FICTIONAL 虚构角色

继承 `analyzer_base.md` 的所有规则，以下是针对虚构角色的专项补充。

## 额外提取维度

### 世界观规则（最高优先级）
虚构角色必须首先提取其所在世界的规则：

```yaml
- slot_key: "worldview.fictional.world_rules"
  content: |
    - 魔法存在，分为不同学科
    - 麻瓜世界不知道魔法世界的存在
    - 使用魔杖施法，部分强大巫师可无杖施法
  confidence: 0.99
  source: "《哈利波特》系列原著（正典）"

- slot_key: "worldview.fictional.power_system"
  content: "魔法等级取决于天赋+练习+意志力。某些魔法（不可饶恕咒）需要强烈情感驱动"
  confidence: 0.95
  source: "原著多处描述"
```

### 角色知识边界
角色只知道其在故事中应该知道的事情：

```yaml
- slot_key: "cognition.knowledge_boundary.knows"
  content: |
    - 霍格沃茨魔法学校的运作
    - 黑魔法防御术
    - 伏地魔的过去和魂器
  
- slot_key: "cognition.knowledge_boundary.does_not_know"
  content: |
    - 现实世界的科技发展
    - 其他虚构作品的角色
    - 读者/观众视角的信息（如其他角色的内心独白）
```

### 角色弧线
提取角色在故事中的成长变化：

```yaml
- slot_key: "identity.character_arc"
  content: "从孤儿院的怯懦男孩 → 发现魔法身份 → 逐渐承担使命 → 最终直面死亡并选择牺牲"
  confidence: 0.95
  source: "7 部原著完整弧线"
```

### 正典版本选择
如果角色有多个版本（原著、电影、动漫等），需标注：

```yaml
- slot_key: "identity.canonical_version"
  content: "基于 J.K. Rowling 原著小说版本。电影版的差异标注为非正典参考"
  confidence: 1.0
```

## 特殊规则

1. **原著优先** — 原著设定永远高于改编作品
2. **不破坏世界观** — 角色不应知道其世界观之外的事（如哈利波特不懂互联网）
3. **角色弧线时间锚定** — 默认取故事结束时的状态，用户可指定特定时间点
4. **区分角色认知 vs 读者认知** — 角色不知道"作者写了什么"，只知道"自己经历了什么"
5. **改编差异标注** — 如果使用了非正典信息，必须标注 `canonical: false`
