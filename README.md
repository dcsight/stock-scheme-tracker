# Stock Strategy Tracker 📈

[![Version](https://img.shields.io/badge/version-2.0.0-blue)](https://github.com/dcsight/stock-strategy-tracker)
[![License](https://img.shields.io/badge/license-MIT--0-green)](LICENSE)

个股投资策略跟踪器 — 将分析报告中提取的投资策略持久化到本地 Markdown 文件，定期核对市场行情是否触发了建仓、止损或止盈条件。

**Save and track per-stock investment strategies with entry/exit conditions.** Extract strategies from analysis reports → persist to a local tracking file → periodically check against live market data.

---

## 功能

- **保存模式（Save）**：完成个股分析后，自动提取建仓条件、止损线、目标价、退出信号等关键字段，格式化追加到策略跟踪文件
- **核对模式（Check）**：读取所有已保存策略，获取最新行情数据，逐条比对条件触发状态，输出可操作建议

## 核心特性（v2.0.0）

### 行业分组系统
按 7 大行业组对持仓分类，不同行业对宏观事件的敏感度完全不同：

| 行业组 | 示例 |
|--------|------|
| 科技/半导体 | AI算力、芯片、消费电子 |
| 新能源/电力 | 光伏、储能、锂电 |
| 化工/石化 | 化纤、煤化工、烯烃 |
| 军工/航天 | 商业航天、碳纤维 |
| 消费/医药 | 白酒、中药、创新药 |
| 互联网/平台 | 港股科技龙头 |
| 金融 | 券商、银行、保险 |

### 宏观上下文感知
策略检查前自动获取当日宏观数据（美债利率、油价、汇率等），按行业匹配影响范围，避免"一个宏观事件影响所有持仓"的错误假设。

### 三态标注规范
所有操作状态严格区分三层：
- `已触发` — 价格/条件到达
- `建议执行` — 系统建议动作（需用户决策）
- `已执行` — ⚠️ 仅用户确认后可写

### 低吸→右侧切换规则
建仓后股价上涨 8-12%，原低位加仓档自动失效 → 切换为右侧突破回踩确认加仓模式。

### 做T回补规则
回补挂单未成交 → 当天禁止追回，等次日再评估。

---

## 快速开始

### 安装

```bash
# WorkBuddy
cp -r stock-strategy-tracker ~/.workbuddy/skills/

# Codex / OpenClaw
cp -r stock-strategy-tracker ~/.codex/skills/

# Claude Code
cp -r stock-strategy-tracker .claude/skills/
```

### 配置

```bash
# 设置策略文件路径
export STOCK_STRATEGY_PATH="$HOME/Documents/stock/strategy.md"
```

未设置时默认使用 `~/Documents/stock/strategy.md`。

### 使用

**保存策略**（分析完成后）：
```
"保存策略"、"跟踪这个票"、"记录建仓条件"
```

**核对策略**（日常检查）：
```
"检查策略"、"核对建仓条件"、"看看哪些票可以买"
```

---

## 兼容性

兼容 WorkBuddy / Codex (OpenClaw) / Claude Code 等 AI Agent 工具。仅需 Python 3 运行时，脚本无网络请求。

## 安全

- 仅读写本地 Markdown 文件
- 不连接券商账户
- 不收集凭据
- 行情数据由 Agent 通过其金融数据工具获取

## 文件结构

```
stock-strategy-tracker/
├── SKILL.md              # 技能定义（完整工作流）
├── README.md             # 本文件
├── references/
│   └── strategy_format.md  # 策略文件格式规范
└── scripts/
    ├── save_strategy.py    # 保存模式脚本
    └── check_strategy.py   # 核对模式脚本
```

## 版本

- **2.0.0** — 行业分组系统、宏观上下文、三态标注、低吸→右侧切换规则、化工子类型分析
- **1.1.0** — 元数据一致性修复、安全声明完善
- **1.0.0** — 初始版本

## License

MIT-0

## Publisher

[@dcsight](https://github.com/dcsight)
