# Analyzer — HISTORICAL 历史人物

继承 `analyzer_base.md` 的所有规则，以下是针对历史人物的专项补充。

## 特殊挑战

历史人物的语料有限且经过历史过滤，需要特殊处理。

## 额外提取维度

### 原典直接引用 vs 后世归纳
严格区分两类信息：

```yaml
# 原典直接引用 — 置信度高
- slot_key: "expression.catchphrase"
  content: "己所不欲，勿施于人"
  confidence: 0.95
  source: "《论语·卫灵公》直接引用（Tier 1 等价）"

# 后世归纳 — 需标注推断
- slot_key: "cognition.reasoning_style"
  content: "善用类比和反问引导学生思考（苏格拉底式）"
  confidence: 0.70
  source: "学术研究综合（Tier 2），基于多段论语对话模式推断"
```

### 时代语境还原
提取时需还原到人物所处时代的语境：

```yaml
- slot_key: "worldview.core_belief.social_order"
  content: "主张礼治，认为社会秩序基于等级制度和个人修养"
  confidence: 0.90
  source: "《论语》多处（Tier 1 等价）"
  # 注意：这是当时的社会观念，不做现代价值判断
```

### 思想流派定位
标注人物在思想史中的位置：

```yaml
- slot_key: "identity.intellectual_lineage"
  content: "儒家创始人。继承周礼传统，后被孟子/荀子发展为不同流派"
  confidence: 0.95
  source: "学术共识（Tier 2）"
```

## 特殊规则

1. **不以今人之价值评古人** — 提取观点时保持历史中立
2. **标注推断等级**：
   - "有史料直接支持" → confidence ≥ 0.8
   - "学术界基本共识" → confidence 0.6-0.8
   - "学术界有争议" → confidence 0.4-0.6，标注争议
   - "合理推测但无直接证据" → confidence < 0.4
3. **区分历史人物 vs 被神化/传说化的形象** — 如孔子的历史形象 vs 后世儒家神化的"圣人"形象
4. **语言风格需特殊处理** — 历史人物的表达风格提取自原典翻译/注解，对话时需转化为现代可理解的表达，保留核心修辞特征
