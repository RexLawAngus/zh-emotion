# 给后续模型与 agent 使用

## 默认交接

答题式分析卡本身就是默认交接格式。调用方先读取原文和依据，再读取综合判断及风险与歧义；不能只截取情绪词去执行行动。没有要求时不生成建议或追问清单。

需要下一步指导时，分别列出：已知事实、尚待确认的推断、与调用方任务有关的建议。建议注明适用条件；不要把“可能失望”升级为“用户确实失望”。后续回应可以确认感受，避免强加情绪标签。

## 可选 JSON

仅在调用方要求 JSON 时使用，不强制替换默认卡片。与卡片保持相同结论及不确定性。字段如下：

- `schema_version`: 固定为 `1.0`；与技能版本分开。
- `context`: 语境、已知背景及缺口，不填虚构事实。
- `units`: 每项包括 `id`、`text`、`technique`、`content`、`effect`、`emotions`、`evidence`、`synthesis`、`uncertainties`。
- `emotions`: 每项包括 `label`、`subject`、`target`、`source_type`（`self_report`／`explicit_description`／`inference`）、`assertion_status`（`affirmed`／`negated`／`hypothetical`／`uncertain`）、`time_reference`（原文支持的时间描述或 `null`）、`support`（`explicit`／`supported`／`undetermined`）、`expression_intensity`、`reported_experience_intensity`。可另记态度和意图，不混入情绪标签。
- 明确否认的情绪若需要记录，标 `negated`，例如“不难过”不能仅输出一个无否定标记的“悲伤”标签。`support` 说明判断依据，不能代替 `assertion_status`；对肯定程度有疑问与原文明确否认是两回事。
- `evidence`: 每项包括 `unit_id`、`quote`、`supports`；引用须能在对应输入单元中找到。只有上下文引用时指向其自身单元，不假称来自目标句。需要字符偏移时注明以 Unicode 码点计数、从 0 开始、左闭右开，逐一核对后提供。
- `trajectory`: 同一主体的变化及证据，无可判断变化时为空数组。
- 未知标量用 `null`，没有记录用空数组；没有明确情绪时 `emotions` 可为空，并在 `uncertainties` 解释原因。不虚构“中性”感受者。
- 强度只在请求编码时填写 0—5，否则为 `null`。自报分数原样另记，未经预先规定不转换量尺；编码含义见 emotion.md。

自拟例句“我有点失望”的输出示意：

```json
{
  "schema_version": "1.0",
  "context": {"genre": "未明确", "known": [], "missing": ["事件背景", "情绪对象"]},
  "units": [{
    "id": "S1",
    "text": "我有点失望。",
    "technique": "第一人称自述，以有点弱化程度",
    "content": "主体报告轻微失望",
    "effect": "直接说明感受并弱化表达程度",
    "emotions": [{
      "label": "失望", "subject": "说话者", "target": null,
      "source_type": "self_report", "assertion_status": "affirmed",
      "time_reference": null, "support": "explicit",
      "expression_intensity": null, "reported_experience_intensity": null
    }],
    "evidence": [{"unit_id": "S1", "quote": "我有点失望", "supports": "主体自述失望，并报告轻微程度"}],
    "synthesis": "以第一人称和弱化程度词直接自述轻微失望，对象与原因未知。",
    "uncertainties": ["不能确认是否为输入者本人体验", "不能判断持续时间"]
  }],
  "trajectory": []
}
```

## 研究与评估用途

固定模型与版本、提示词、技能版本和上下文范围，保存原始输出和人工复核记录。标注者不知道评估目标时可减少预设影响；仍需独立研究设计和验证。不可把本技能的支持程度标记当作经过校准的概率。

接入抑郁相关评估时，只交接文本支持的体验线索、主体、时态、持续信息、功能影响及缺口。保持来源可追溯，不从情绪强度直接换算抑郁分数。角色台词、剧情选择与本人自述分别记录；叙事情景中选择与心理构念的关系需另行验证。

真实用户当前表达安全风险时，由调用方按其规则回应和处理，不等待文学赏析或数字编码完成。本技能不设临床诊断、量表阈值或危机风险等级。
