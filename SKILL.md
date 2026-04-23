---
name: stock-scheme-tracker
description: This skill should be used when the user has completed an investment analysis for a specific stock and wants to save the resulting trading strategy to a tracking file, or when the user wants to check if any saved stock strategies have triggered their entry/exit conditions. Triggers on phrases like "保存策略", "跟踪策略", "检查股票策略", "核对建仓条件", "scheme", "stock strategy", or after completing any stock investment analysis. Supports two modes: save (extract strategy from analysis and append to /Users/dc/DCmacOB/stock/scheme.md) and check (fetch latest prices, evaluate all conditions, output actionable recommendations).
---

# Stock Strategy Tracker

## Overview

Save per-stock investment strategies extracted from analysis reports into a persistent tracking file (`/Users/dc/DCmacOB/stock/scheme.md`), then periodically check if market conditions have triggered any entry, stop-loss, or take-profit conditions. Provide actionable next-step recommendations.

## Workflow Decision Tree

1. **Determine mode**: Did the user just complete an analysis (→ save mode), or do they want to review existing strategies (→ check mode)?
2. **Save mode**: Extract strategy elements from the analysis, format them, and append to scheme.md.
3. **Check mode**: Read all strategies from scheme.md, fetch latest market data for each stock, evaluate every condition, and output a summary with next-step recommendations.

## Save Mode

### When to Use
- Immediately after completing any stock investment analysis (e.g., DCF valuation, Buffett framework, technical analysis, or ljg-invest report).
- The user says "保存策略", "把策略记下来", "跟踪这个票", or any variant.

### Extraction Rules
From the analysis text, extract these fields. If a field is not mentioned, mark it as "未明确" rather than fabricating.

| Field | Description | Example |
|-------|-------------|---------|
| `stock_code` | 股票代码 | 688111 |
| `stock_name` | 股票名称 | 金山办公 |
| `analysis_date` | 分析日期 | 2026-04-22 |
| `recommendation` | 核心建议 | 建议观察，等待更好入场时机 |
| `entry_conditions` | 建仓条件（价格/估值/事件） | PE≤45-50；股价180-200元 |
| `stop_loss` | 止损条件 | 跌破前期低点或技术位 |
| `target_price` | 目标价/止盈区间 | 机构目标价400元 |
| `exit_signals` | 退出信号（假设证伪条件） | 月活环比下滑；市场份额跌破80% |
| `position_size` | 建议仓位 | 轻仓试探 |
| `time_horizon` | 持有周期 | 3年 |
| `key_assumptions` | 核心假设 | AI商业化持续推进 |
| `open_questions` | 未解问题 | WPS AI付费转化率是多少 |
| `current_price` | 分析时股价 | ~250元 |
| `current_pe` | 分析时PE | 62.46 |
| `market_cap` | 分析时市值 | 1147亿 |

### Output Format
Append a new section to `/Users/dc/DCmacOB/stock/scheme.md` using the exact format defined in `references/scheme_format.md`. Use Markdown table for entry conditions so they can be machine-parsed later.

### Important
- If the stock already exists in scheme.md, update the existing section rather than duplicating it. Preserve historical entries by appending a date-stamped update note.
- After saving, briefly confirm to the user what was saved.

## Check Mode

### When to Use
- User says "检查策略", "核对建仓条件", "看看哪些票可以买", "策略跟踪", or any variant.
- Can be invoked standalone without any prior analysis.

### Steps
1. Read `/Users/dc/DCmacOB/stock/scheme.md`.
2. For each tracked stock, fetch the latest market data (price, PE, key metrics). Use neodata-financial-search skill if available; otherwise use web search.
3. Evaluate each `entry_conditions` row: compare current value vs target value.
4. Mark each condition as `✅ 已触发`, `❌ 未触发`, or `⚠️ 接近触发`.
5. Check if any `exit_signals` have occurred.
6. Output a clean summary table and specific next-step recommendations per stock.

### Output Format

```markdown
## 策略核对报告 — 2026-04-23

### 金山办公 (688111)
| 条件类型 | 目标 | 当前 | 状态 |
|----------|------|------|------|
| PE建仓 | ≤45-50 | 62.46 | ❌ 未触发 |
| 股价建仓 | 180-200 | 250 | ❌ 未触发 |

**建议**: 继续观察，等待PE回落至50以下或股价跌破200再考虑建仓。
```

## Resources

- `references/scheme_format.md` — Exact Markdown schema for scheme.md entries.
- `scripts/save_scheme.py` — Script to parse and append strategy data to scheme.md.
- `scripts/check_scheme.py` — Script to read scheme.md, fetch prices, and evaluate conditions.
