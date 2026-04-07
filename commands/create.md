# /anyone create — 详细编排流程

当用户执行 `/anyone create <name>` 时，按以下步骤顺序执行。每一步都必须完成才能进入下一步。

---

## 变量约定

整个流程中使用以下变量传递数据：

- `{user_input}` — 用户输入的名字或描述
- `{intake_result}` — Step 1 的输出
- `{search_plan}` — Step 2 的输出
- `{raw_corpus}` — Step 3 的输出
- `{extracted_slots}` — Step 4 的输出
- `{persona_yaml}` — Step 5 的输出
- `{base_prompt}` — Step 6 的输出
- `{quality_report}` — Step 7 的输出
- `{PERSONA_DIR}` — `~/.anyone-skill/personas/{id}/`

---

## Step 1: Intake — 对象识别

**读取** `prompts/intake.md`，按其中的规则分析 `{user_input}`。

**执行逻辑：**
1. 判断对象类型（REAL_PUBLIC / HISTORICAL / FICTIONAL / ARCHETYPE）
2. 判断子类型
3. 评估语料丰富度
4. 确定主要语言
5. 如果对象模糊（如"孙悟空"可能是神话或影视），**询问用户**偏好哪个版本
6. 生成 `id`：中文名用拼音（如 `kong-zi`），英文名用小写连字符（如 `elon-musk`）

**输出 `{intake_result}`：**
```yaml
intake_result:
  name: "{显示名}"
  id: "{kebab-case-id}"
  type: "{REAL_PUBLIC|HISTORICAL|FICTIONAL|ARCHETYPE}"
  subtype: "{子类型}"
  era: "{contemporary|historical:XXX|fictional:XXX}"
  language_primary: "{en|zh|ja|ko}"
  source_richness: "{abundant|moderate|scarce|none}"
  worldview_id: null          # 仅 FICTIONAL 填写
  user_focus: null             # 用户指定的侧重点
  supplementary_files: []
```

**向用户展示识别结果，确认后继续。**

---

## Step 2: Search — 搜索策略生成

**读取** `prompts/searcher.md`，根据 `{intake_result}` 生成搜索策略。

**执行逻辑：**
1. 根据 `type` 选择对应的搜索模板（REAL_PUBLIC / HISTORICAL / FICTIONAL / ARCHETYPE）
2. 生成 6-8 个搜索查询，确保覆盖 6 类记忆维度
3. 查询语言匹配 `language_primary`（中文人物加中文查询，英文加英文，混合两种都加）
4. 如果 `source_richness` 为 scarce/none，降低 `fetch_limit`

**输出 `{search_plan}`：**
```yaml
search_plan:
  queries:
    - query: "具体搜索词"
      target_tier: 1
      target_layers: [expression, cognition]
    # ... 共 6-8 个
  fetch_limit: 3
  language_preference: "{语言}"
```

---

## Step 3: Fetch — 语料采集

**使用工具：** WebSearch + WebFetch

**执行逻辑：**

1. **并行执行所有搜索查询：**
   对 `{search_plan}.queries` 中的每个 query，调用 `WebSearch`。使用 Agent 工具并行执行多个搜索以加速。

2. **筛选搜索结果：**
   每个查询选取前 `{search_plan}.fetch_limit` 个最相关的结果 URL。
   总计不超过 **20 个页面**。

3. **抓取页面内容：**
   对选中的 URL 调用 `WebFetch`，使用 Agent 工具并行抓取。

4. **组装语料列表：**
   每条语料记录来源信息。

**输出 `{raw_corpus}`：**
```yaml
raw_corpus:
  - url: "https://..."
    title: "来源标题"
    tier: 1                    # 根据内容判断：1=本人言论, 2=权威传记, 3=一般文章, 4=用户提供
    target_layers: [expression, cognition]
    content: "抓取到的文本内容（截取关键部分，每篇不超过 2000 字）"
    
  # ... 最多 20 条
```

**如果语料太少（< 3 条有效结果）：**
- 提示用户："语料较少，画像可能不够丰富。是否继续？或提供补充材料？"
- 用户可选择继续或补充

---

## Step 4: Analyze — 语料分析

**读取** `prompts/analyzer/analyzer_base.md` + 对应类型的 `analyzer_{type}.md`。

根据 `{intake_result}.type`，选择类型补丁文件：
- REAL_PUBLIC → `prompts/analyzer/analyzer_real.md`
- HISTORICAL → `prompts/analyzer/analyzer_historical.md`
- FICTIONAL → `prompts/analyzer/analyzer_fictional.md`
- ARCHETYPE → `prompts/analyzer/analyzer_archetype.md`

**执行逻辑：**

1. 先读取 `analyzer_base.md` 作为基础分析框架
2. 再读取 `analyzer_{type}.md` 作为类型补充规则
3. 将 `{intake_result}` 和 `{raw_corpus}` 作为输入
4. 按 6 个维度提取记忆槽位（identity / worldview / expression / cognition / emotion / social）
5. 每个槽位必须标注来源和置信度

**输出 `{extracted_slots}`：**
```yaml
extracted_slots:
  - slot_key: "identity.birthplace"
    category: identity
    content: "具体、可执行的描述"
    confidence: 0.95
    source: "来源说明（Tier N）"
    evidence: "支撑证据描述"
  # ... 尽可能多的槽位
```

**分析完成后检查覆盖度：**
- identity ≥ 5、worldview ≥ 5、expression ≥ 8、cognition ≥ 5、emotion ≥ 3、social ≥ 3
- 不足的维度提示用户："{类别} 类记忆不足（当前 N 个，建议 ≥ M），可补充语料或跳过"
- 用户可选择跳过（接受当前结果）或补充

---

## Step 5: Build — 画像组装

**读取** `prompts/builder/builder_base.md` + 对应类型的 `builder_{type}.md`。

根据 `{intake_result}.type`，选择类型补丁文件：
- REAL_PUBLIC → `prompts/builder/builder_real.md`
- HISTORICAL → `prompts/builder/builder_historical.md`
- FICTIONAL → `prompts/builder/builder_fictional.md`
- ARCHETYPE → `prompts/builder/builder_archetype.md`

**执行逻辑：**

1. 先读取 `builder_base.md` 作为构建框架
2. 再读取 `builder_{type}.md` 作为类型补充规则
3. 将 `{intake_result}` 和 `{extracted_slots}` 作为输入
4. 整理槽位：去重、合并相似内容、统一 slot_key 命名
5. 标记核心记忆（`is_core: true`）：5-10 个最能定义此人格的高置信度槽位
6. 计算成熟度阶段
7. 每个槽位初始化 `reinforced_count: 0`、`last_activated: null`

**输出 `{persona_yaml}`：完整的 persona.yaml 内容**

格式严格遵循 `schemas/persona.schema.yaml`：
```yaml
slots:
  {slot_key}:
    content: "具体描述"
    confidence: 0.85
    source: "来源（Tier N）"
    reinforced_count: 0
    last_activated: null
    is_core: false
```

---

## Step 6: SysGen — System Prompt 生成

**读取** 对应类型的 `prompts/system_prompt_generator/sysgen_{type}.md`。

根据 `{intake_result}.type`：
- REAL_PUBLIC → `sysgen_real.md`
- HISTORICAL → `sysgen_historical.md`
- FICTIONAL → `sysgen_fictional.md`
- ARCHETYPE → `sysgen_archetype.md`

**执行逻辑：**

1. 读取对应的 sysgen 模板
2. 从 `{persona_yaml}` 的 identity 槽位综合生成 one_paragraph_bio（~100 词）
3. 按模板生成 base_prompt.md
4. 控制在 500-700 tokens

**输出 `{base_prompt}`：一段完整的 Markdown 文本**

包含：
- Layer 0: 安全规则（AI 模拟声明）
- Layer 1: 身份核心（一段话简介）
- Context Rules: 条件处理规则
- Memory-Driven Response Protocol: 动态记忆使用规则

---

## Step 7: Quality — 质量检查

**读取** `prompts/quality_checker.md`。

**执行逻辑：**

将 `{persona_yaml}` 作为输入，按 quality_checker.md 的 5 项检查逐一执行：

1. **槽位覆盖率** — 6 类是否达到最低要求
2. **置信度分布** — 是否健康（30-50% high / 30-50% medium / <20% low）
3. **核心记忆** — 5-15 个 is_core:true，覆盖 ≥ 3 类
4. **一致性** — 是否有逻辑矛盾
5. **安全** — Layer 0 是否完整，是否有隐私问题

**输出 `{quality_report}`：**
```yaml
quality_report:
  overall_score: "B+"
  coverage: { ... }
  confidence_distribution: { ... }
  core_memories: { ... }
  consistency: { ... }
  safety: { ... }
  maturity: { stage, slot_count, effective_slots }
  recommendations: [...]
```

---

## Step 8: Save — 保存文件

**使用工具：** Write

**执行逻辑：**

1. 创建目录结构 `{PERSONA_DIR}`：
   ```bash
   mkdir -p ~/.anyone-skill/personas/{id}/sources
   mkdir -p ~/.anyone-skill/personas/{id}/corrections
   mkdir -p ~/.anyone-skill/personas/{id}/flywheel
   mkdir -p ~/.anyone-skill/personas/{id}/versions/v1.0.0
   ```

2. **写入 meta.yaml**（遵循 `schemas/meta.schema.yaml`）：
   ```yaml
   id: "{id}"
   name: "{name}"
   type: "{type}"
   subtype: "{subtype}"
   era: "{era}"
   language_primary: "{language_primary}"
   source_richness: "{source_richness}"
   worldview_id: {worldview_id}
   confidence_level: {从 quality_report 计算的平均置信度}
   maturity:
     stage: "{从 quality_report 获取}"
     slot_count: {总槽位数}
     correction_count: 0
     reinforcement_count: 0
   created_at: "{ISO 时间戳}"
   updated_at: "{ISO 时间戳}"
   version: "1.0.0"
   ```

3. **写入 persona.yaml** — 直接写入 `{persona_yaml}`

4. **写入 base_prompt.md** — 直接写入 `{base_prompt}`

5. **写入 sources/manifest.yaml**（遵循 `schemas/source_manifest.schema.yaml`）：
   从 `{raw_corpus}` 转换生成。

6. **写入 corrections/corrections.yaml** — 空数组初始化：
   ```yaml
   corrections: []
   ```

7. **写入 flywheel/reinforcement_log.yaml** — 空初始化：
   ```yaml
   reinforcement_log: []
   last_session: null
   total_sessions: 0
   ```

8. **版本存档**：将 persona.yaml 和 meta.yaml 复制到 `versions/v1.0.0/`

---

## Step 9: Present — 展示结果

向用户展示画像摘要：

```
✅ 化身创建完成！

📋 基本信息
- 名称: {name}
- ID: {id}
- 类型: {type} / {subtype}
- 时代: {era}

📊 质量报告
- 总体评分: {overall_score}
- 成熟度: {maturity_emoji} {stage}（{slot_count} 个槽位）
- 覆盖率:
  identity: {count} ✓/△
  worldview: {count} ✓/△
  expression: {count} ✓/△
  cognition: {count} ✓/△
  emotion: {count} ✓/△
  social: {count} ✓/△

🧠 核心记忆 (Top 5):
1. {slot_key}: {content 摘要}
2. ...

💡 建议:
{recommendations}

📁 保存位置: ~/.anyone-skill/personas/{id}/

输入 `/anyone chat {id}` 开始对话
```

**用户确认完成。流程结束。**

---

## 异常处理

| 场景 | 处理 |
|------|------|
| WebSearch 全部失败 | 提示用户手动提供语料或稍后重试 |
| 语料太少（< 3 条） | 警告并询问是否继续或补充 |
| 覆盖度严重不足（≥ 3 类不达标） | 建议用户补充语料后用 `/anyone evolve` 增量进化 |
| persona 目录已存在同 id | 提示用户已有同名化身，是否覆盖或使用新 id |
| 用户中途取消 | 清理临时数据，不保存 |
