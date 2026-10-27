# 题库字段

```json
{
  "id": "CA-001",
  "title": "欧姆定律与功率",
  "chapter": "电路元件与基本定律",
  "type": "计算题",
  "difficulty": 1,
  "source": "自编",
  "problem": "完整题干",
  "given": {"U": "12 V", "R": "1 kΩ"},
  "asks": ["I", "P"],
  "answer": {"I": "12 mA", "P": "144 mW"},
  "solution": ["第一步", "第二步"],
  "netlist": "circuits/ohms-law.cir",
  "simulation": {
    "analysis": "op",
    "expect": {"V(in)": "12 V", "@r1[i]": "12 mA"}
  },
  "common_errors": ["易错点"],
  "created": "2026-10-26"
}
```

## 约定

- `id`：科目前缀 + 三位序号。`CA` 电路分析，`AN` 模电，`DG` 数电。
- `difficulty`：1 基础，2 常规，3 综合，4 压轴。
- `type`：计算题、分析题、设计题、证明题、选择题、填空题。
- `netlist`、`simulation` 可省略；有仿真就必须能通过 `check.ps1`。
- `source` 写「自编」或具体来源，不写模糊出处。
- 新题追加到对应科目文件的 `questions` 数组，不覆盖旧题。
