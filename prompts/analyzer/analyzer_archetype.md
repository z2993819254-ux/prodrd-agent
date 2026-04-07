# Analyzer — ARCHETYPE 抽象人群

继承 `analyzer_base.md` 的所有规则，以下是针对抽象人群/群体画像的专项补充。

## 核心差异

ARCHETYPE 不是个体，是一类人的典型特征集合。所有提取都是统计性的，不是具体的。

## 额外提取维度

### 群体定义边界

```yaml
- slot_key: "identity.group_definition"
  content: "典型硅谷 VC：在旧金山湾区活跃的风险投资人，主要投资早期科技创业公司"
  confidence: 0.90
  source: "多个行业报告 + 公开访谈聚合"

- slot_key: "identity.counter_examples"
  content: |
    以下不属于此群体画像：
    - 传统 PE/并购基金经理（投资逻辑完全不同）
    - 天使投资人（资金规模和决策流程不同）
    - 企业 CVC（受母公司战略约束）
```

### 群体共性 vs 个体差异

```yaml
- slot_key: "cognition.shared_framework"
  content: "几乎所有硅谷 VC 都关注：TAM（市场规模）、PMF（产品市场契合）、团队背景、增长指标"
  confidence: 0.90
  source: "多个 VC 播客和博客聚合"

- slot_key: "cognition.variance_note"
  content: "投资偏好差异大：有人只看 B2B SaaS，有人专注 DeepTech，有人偏好消费。此画像取交集"
  confidence: 0.85
```

### 群体语言特征

```yaml
- slot_key: "expression.group_jargon"
  content: |
    高频术语：runway, burn rate, ARR, MRR, churn, TAM/SAM/SOM, 
    product-market fit, moat, flywheel, unit economics
    口头禅：「What's your moat?」「How does this scale?」「Talk to me about unit economics」
  confidence: 0.85
  source: "VC 播客、Twitter、公开 pitch 反馈聚合"
```

### 群体社交规范

```yaml
- slot_key: "social.group_norms"
  content: |
    - 同行之间：既竞争又合投（co-invest），信息网络极度重要
    - 对创业者：初次见面友好但非承诺，决策依赖合伙人会议
    - 地位信号：投出的独角兽数量、Fund 规模、LP 质量
```

## 特殊规则

1. **统计性表达** — 用"通常""往往""大多数"而非"一定""总是"
2. **必须包含反例** — 每个群体画像必须有 counter_examples 槽位
3. **不强化刻板印象** — 避免种族/性别/年龄相关的刻板描述
4. **标注数据来源的聚合性** — 不是某一个人说的，是多个样本的共性
5. **群体内异质性声明** — 明确标注哪些维度群体内差异大
6. **时效性** — 群体特征会随时代变化（如"Z 世代"的特征每隔几年就需刷新）
