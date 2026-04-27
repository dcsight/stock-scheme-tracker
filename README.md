# stock-scheme-tracker 📈

> 个股投资策略跟踪器 — 自动保存建仓条件、核对市场状态、给出下一步建议。

适用于 OpenClaw / WorkBuddy / Claude Code 等 AI Agent 工具。完成任何个股分析后，一句话保存策略；随时检查市场条件是否触发入场/止损。

## 功能

| 模式 | 触发词 | 功能 |
|------|--------|------|
| **Save** | `保存策略`、`跟踪这个票`、`记录建仓条件` | 从分析报告中提取建仓条件、止损线、目标价、退出信号，追加到 `scheme.md` |
| **Check** | `检查策略`、`核对建仓条件`、`看看哪些票可以买` | 读取已保存策略，获取最新行情，判断条件是否触发，给出建议 |

## 文件结构

```
stock-scheme-tracker/
├── SKILL.md                     # Skill 核心指令
├── references/
│   └── scheme_format.md         # scheme.md 格式规范
└── scripts/
    ├── save_scheme.py           # 保存策略（JSON输入 → 追加到 scheme.md）
    └── check_scheme.py          # 核对策略（解析 scheme.md → 生成报告）
```

## 安装

将整个文件夹放入你使用的 Agent 的 skills 目录：

```bash
# OpenClaw
cp -r stock-scheme-tracker ~/.openclaw/skills/

# WorkBuddy
cp -r stock-scheme-tracker ~/.workbuddy/skills/

# Claude Code (project-level)
cp -r stock-scheme-tracker .claude/skills/
```

## 配置

设置环境变量指向你的策略跟踪文件：

```bash
# 添加到 shell profile (~/.zshrc, ~/.bashrc 等)
export STOCK_SCHEME_PATH="$HOME/Documents/stock/scheme.md"
```

如果未设置，默认路径为 `~/Documents/stock/scheme.md`。

## 使用方式

### 保存策略（分析完成后）

```bash
python3 scripts/save_scheme.py < strategy.json
```

或直接告诉 AI：`保存策略`，AI 会自动从分析报告中提取并保存。

`strategy.json` 示例：

```json
{
  "stock_code": "688XXX",
  "stock_name": "XX办公",
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
      "code": "688XXX",
      "name": "XX办公",
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
## XX办公 (688XXX)

- **分析日期**: 2026-04-22
- **当前状态**: 观察中
- **分析结论**: 建议观察，等待更好入场时机

### 建仓条件

| 条件 | 目标值 | 当前值 | 是否触发 | 备注 |
|------|--------|--------|----------|------|
| PE(TTM) | ≤45-50 | 57 | ❌ 未触发 | Q1扣非增速21-35% |
| 股价 | 180-200元 | ~250元 | ❌ 未触发 | 距目标区间仍有空间 |
```

## 安全声明

- 脚本仅在本地读写 Markdown 文件，不发起网络请求
- Check 模式通过 Agent 的金融数据工具或 web search 获取公开行情，不传输私有策略内容
- 不执行任何交易，不连接券商账户
- 不收集 API Key、密码等凭据

## 依赖

- Python 3.8+
- 无外部依赖，纯标准库

## License

MIT-0
