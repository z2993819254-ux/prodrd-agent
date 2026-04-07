# Analyzer — REAL_PUBLIC 真实公众人物

继承 `analyzer_base.md` 的所有规则，以下是针对真实公众人物的专项补充。

## 额外提取维度

### 多媒介表达差异
公众人物在不同场景下表现不同，需分别提取：

```yaml
# 社交媒体风格
- slot_key: "expression.medium.twitter"
  content: "短句、挑衅性、频繁使用 emoji 和 meme"
  
# 访谈风格
- slot_key: "expression.medium.interview"
  content: "更深思熟虑、会引用数据、偶尔停顿思考"

# 演讲风格
- slot_key: "expression.medium.speech"
  content: "重复关键句、节奏感强、善用停顿制造效果"
```

### 公开立场追踪
对已知的公开立场，需记录来源和时间：

```yaml
- slot_key: "worldview.position.ai_regulation"
  content: "反对过度监管 AI，认为会扼杀创新。支持行业自律而非政府管控"
  confidence: 0.85
  source: "国会听证会 2023 + Twitter 多条发言（Tier 1）"
```

### 公众形象 vs 可能的私下形象
- 只提取公开可验证的信息
- 如果多个来源描述了"人前人后不一致"，可作为一个槽位记录，但标注为 Tier 3 推断
- 不主动猜测私下行为

## 特殊规则

1. **争议性内容**：如果人物在某议题上有争议，提取所有主要立场，标注时间线变化
2. **立场演变**：如果发现观点随时间变化，记录演变轨迹
   ```yaml
   - slot_key: "worldview.position.crypto.evolution"
     content: "2020年持怀疑态度 → 2021年公开支持 BTC → 2022年对 Doge 表现出兴趣"
   ```
3. **团队 vs 个人**：区分"公司官方表态"和"个人观点"
4. **语料时效性**：优先最近 2 年的语料，标注日期
