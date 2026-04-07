# /anyone correct — 纠正处理编排流程

当用户在对话中表达纠正意图，或执行 `/anyone correct <id>` 时，按以下流程处理。

---

## 变量约定

- `{id}` — 化身 ID
- `{PERSONA_DIR}` — `~/.anyone-skill/personas/{id}/`
- `{user_correction}` — 用户的纠正描述
- `{original_message}` — 触发纠正的 AI 回复（对话中触发时）
- `{correction_result}` — 纠正处理结果

---

## 触发方式

### 方式 A: 对话中隐式触发

在 `/anyone chat` 对话循环的 Step 2.4（隐式强化）中检测到纠正意图：
- 用户说 "这不像 TA"、"TA 不会这样说"、"不对"、"说话语气不对" 等

**检测到后：**
1. 暂停正常对话循环
2. 询问用户具体哪里不对、应该怎样
3. 进入下面的 Step 1

### 方式 B: 命令直接触发

用户执行 `/anyone correct <id>`：
1. 加载 `{PERSONA_DIR}/persona.yaml` 和 `corrections.yaml`
2. 询问用户要纠正什么
3. 进入 Step 1

---

## Step 1: 理解纠正意图

**读取** `prompts/correction_handler.md`，按其 Step 1 执行。

**输入：**
- `user_correction`：用户的纠正描述
- `original_message`：触发纠正的 AI 回复（方式 B 时为 null）
- `current_slots`：persona.yaml 全部槽位
- `conversation_context`：当前对话上下文（方式 B 时为 null）

**执行：**
从用户描述中提取纠正结构：

```yaml
correction:
  category: "{identity|worldview|expression|cognition|emotion|social}"
  what_was_wrong: "哪里不对的描述"
  what_should_be: "应该怎样的描述"
  trigger_scenario: "适用场景（null 表示通用）"
```

**类型判断参考：**

| 用户说 | category | 目标槽位类别 |
|--------|----------|------------|
| "说话语气不对"、"不会这样讲" | expression | expression.* |
| "TA 不会这样想"、"观点不对" | worldview | worldview.* |
| "TA 不懂这个"、"不是 TA 的专长" | cognition | cognition.* |
| "没这么情绪化"、"反应不对" | emotion | emotion.* |
| "对人不会这样" | social | social.* |
| "这个信息不对" | identity | identity.* |

如果无法判断 → 询问用户属于哪个方面。

---

## Step 2: 定位目标槽位

在 `{all_slots}` 中查找匹配的槽位：

1. **直接匹配**：根据 category + what_was_wrong 内容，找到 slot_key
2. **模糊匹配**：如果没有精确匹配，找该 category 下内容最相关的槽位
3. **无匹配**：创建新槽位（生成合理的 slot_key）

**输出：**
```yaml
target_slot:
  slot_key: "expression.formality"
  current_content: "表达正式、用词考究"
  current_confidence: 0.75
  exists: true                    # false 表示需要新建
```

---

## Step 3: 判断纠正策略

根据 `trigger_scenario` 判断：

### 策略 A: 通用纠正（trigger_scenario = null）

直接覆盖目标槽位的 content。

### 策略 B: 场景化纠正（trigger_scenario ≠ null）

**不覆盖**原槽位，创建新的场景子槽位：

```yaml
# 保留原槽位
expression.formality:
  content: "正式场合用词考究"              # 不变

# 新建场景槽位
expression.formality.casual:
  content: "闲聊场景下说话随意口语化"       # 新建
  confidence: 0.70
  source: "用户纠正 ({日期})"
```

**判断规则：**
- 用户描述中包含 "在...的时候"、"...场景下"、"只有..." → 策略 B
- 用户描述是绝对性的（"从来不"、"总是"、"一直"） → 策略 A
- 不确定 → 询问用户："这个纠正是只在特定场景下适用，还是 TA 一直都这样？"

---

## Step 4: 执行更新

### 更新 persona.yaml

**策略 A（通用纠正）：**
```yaml
{target_slot.slot_key}:
  content: "{what_should_be 转化为槽位格式的具体描述}"
  confidence: 0.70              # 重置为 0.7（非 1.0），待后续对话强化验证
  source: "用户纠正 ({日期})"
  reinforced_count: 0           # 重置为 0，新内容从零开始积累强化（与 correction_handler.md 对齐）
  last_activated: "{当前时间}"
  is_core: {保持原值}
```

**策略 B（场景化纠正）：**
- 原槽位保持不变
- 新增场景子槽位（如上 Step 3 示例）

### 写入 corrections.yaml

追加纠正记录（遵循 `schemas/correction.schema.yaml`）：

```yaml
- id: "corr_{YYYYMMDD}_{序号}"
  timestamp: "{ISO 时间戳}"
  target_slot: "{slot_key}"
  category: "{category}"
  old_value: "{原 content}"
  new_value: "{新 content}"
  user_description: "{用户原始纠正描述}"
  trigger_message_id: "{触发的消息 ID，命令触发时为 null}"
  applied: true
```

---

## Step 5: 重新生成回复

**仅在对话中触发时执行（方式 A）。**

1. 用更新后的槽位重新运行 `prompts/memory_selector.md`
2. 重新拼装上下文（纠正后的槽位会带 `[correction]` 标记）
3. 重新生成回复
4. 展示给用户：
   ```
   ✏️ 已纠正 {slot_key}
   旧: {old_content 摘要}
   新: {new_content 摘要}
   
   重新生成的回复：
   {新回复}
   ```

**命令触发时（方式 B）：**
展示纠正结果即可，无需重新生成：
```
✏️ 纠正完成

目标槽位: {slot_key}
旧值: {old_content}
新值: {new_content}
置信度: 0.70（待对话验证）

纠正记录已保存到 corrections.yaml
```

---

## Step 6: 成熟度更新

更新 `{PERSONA_DIR}/meta.yaml`：
```yaml
maturity:
  correction_count: {+1}
updated_at: "{当前 ISO 时间戳}"
```

---

## 异常处理

| 场景 | 处理 |
|------|------|
| 用户纠正描述模糊 | 追问："能具体说说哪里不对吗？是说话方式、观点、还是其他方面？" |
| 纠正内容与现有槽位矛盾 | 展示矛盾，让用户确认是覆盖还是场景化 |
| 纠正后 confidence 低于正常分布 | 正常现象（0.7），后续对话中验证会自动提升 |
| 多次纠正同一槽位 | 正常，corrections.yaml 保留完整历史 |
