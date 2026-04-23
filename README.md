# stock-scheme-tracker

> 个股投资策略跟踪器 — 自动保存建仓条件、核对市场状态、给出下一步建议。

配合 [WorkBuddy](https://www.codebuddy.cn) AI 助手使用。完成任何个股分析后，一句话保存策略；随时检查市场条件是否触发入场/止损。

## 功能

| 模式 | 触发词 | 功能 |
|------|--------|------|
| **Save** | `保存策略`、`跟踪这个票`、`记录建仓条件` | 从分析报告中提取建仓条件、止损线、目标价、退出信号，追加到 `scheme.md` |
| **Check** | `检查策略`、`核对建仓条件`、`看看哪些票可以买` | 读取已保存策略，获取最新行情，判断条件是否触发，给出建议 |

## 文件结构

```
stock-scheme-tracker/
├── SKILL.md                     # WorkBuddy skill 核心指令
├── references/
│   └── scheme_format.md         # scheme.md 格式规范
└── scripts/
    ├── save_scheme.py          # 保存策略（JSON输入 → 追加到 scheme.md）
    └── check_scheme.py          # 核对策略（解析 scheme.md → 生成报告）
```

## 安装

将整个文件夹放入 WorkBuddy 的 skills 目录：

```bash
cp -r stock-scheme-tracker ~/.workbuddy/skills/
```

## 使用方式

### 保存策略（分析完成后）

```bash
python3 scripts/save_scheme.py < strategy.json
```

或直接在工作Buddy中告诉AI：`保存策略`，AI会自动从分析报告中提取并保存。

`strategy.json` 示例：

```json
{
  "stock_code": "688111",
  "stock_name": "金山办公",
  "analysis_date": "2026-04-22",
  "status": "观察中",
  "entry_conditions": [
    {"name": "PE(TTM)", "target": "≤45-50", "note": "3年分位点低于20%"},
    {"name": "股价", "target": "180-200元", "note": "等待大盘配合"}
  ],
  "stop_loss": "跌破前期低点",
  "target_price": "机构目标价400元",
  "exit_signals": ["月活环比下滑", "政府IT支出削减"],
  "open_questions": ["付费转化率？", "海外扩张进度？"]
}
```

### 核对策略（定期检查）

```bash
# 解析并显示当前状态
python3 scripts/check_scheme.py --parse

# 生成完整核对报告
python3 scripts/check_scheme.py --report < stocks.json
```

`stocks.json` 示例：

```json
{
  "stocks": [
    {
      "code": "688111",
      "name": "金山办公",
      "status": "观察中",
      "conditions": [
        {"name": "PE(TTM)", "target": "≤45-50", "current": "57", "triggered": "❌ 未触发", "note": "Q1扣非增速21-35%"}
      ],
      "recommendation": "耐心等待180-200元区间建仓"
    }
  ]
}
```

## scheme.md 格式

策略保存到 `scheme.md`，格式如下：

```markdown
## 金山办公 (688111)

- **分析日期**: 2026-04-22
- **当前状态**: 观察中
- **分析结论**: 建议观察，等待更好入场时机

### 建仓条件

| 条件 | 目标值 | 当前值 | 是否触发 | 备注 |
|------|--------|--------|----------|------|
| PE(TTM) | ≤45-50 | 57 | ❌ 未触发 | Q1扣非增速21-35% |
| 股价 | 180-200元 | ~250元 | ❌ 未触发 | 距目标区间仍有空间 |
```

## 依赖

- Python 3.8+
- 无外部依赖，纯标准库

## License

MIT
