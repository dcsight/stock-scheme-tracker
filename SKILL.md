---
name: stock-strategy-tracker
description: 个股投资策略跟踪器——将从分析报告中提取的个股投资策略持久化到一个本地 Markdown 文件，然后定期核对市场行情是否触发了建仓、止损或止盈条件，给出可操作的下一步建议。
  Save and track per-stock investment strategies with entry/exit conditions.
  Triggers on 保存策略, 跟踪策略, 检查股票策略, 核对建仓条件, strategy, stock strategy.
  Two modes: save (extract strategy from analysis → append to tracking file)
  and check (fetch latest prices, evaluate all conditions, output actionable recommendations).
version: 2.0.0
requires:
  anyBins:
    - python3
    - python
  env:
    - STOCK_STRATEGY_PATH
primaryEnv: STOCK_STRATEGY_PATH
emoji: 📈
---

# Stock Strategy Tracker 📈

个股投资策略跟踪器——将从分析报告中提取的个股投资策略持久化到一个本地 Markdown 文件，然后定期核对市场行情是否触发了建仓、止损或止盈条件，给出可操作的下一步建议。

两种模式：
- **保存模式（Save）**：完成任意个股分析后（DCF 估值、巴菲特框架、技术分析等），自动从分析文本中提取建仓条件、止损线、目标价、退出信号、核心假设等关键字段，格式化后追加到策略跟踪文件。如果该股票已存在则更新而非重复添加。触发词：`保存策略`、`跟踪这个票`、`记录建仓条件`
- **核对模式（Check）**：读取策略跟踪文件中所有已保存的策略，获取每只股票的最新行情数据（价格、PE 等），逐条比对建仓条件是否已触发（✅ 已触发 / ❌ 未触发 / ⚠️ 接近触发），检查退出信号是否出现，输出清晰的汇总表格和每只股票的具体操作建议。触发词：`检查策略`、`核对建仓条件`、`看看哪些票可以买`

---

## Quick Start

### Install

Copy the skill directory to your agent's skills folder:

```bash
# WorkBuddy
cp -r stock-strategy-tracker ~/.workbuddy/skills/

# Codex / OpenClaw
cp -r stock-strategy-tracker ~/.codex/skills/

# Claude Code (project-level)
cp -r stock-strategy-tracker .claude/skills/
```

### Configure

Set the environment variable to point to your tracking file:

```bash
export STOCK_STRATEGY_PATH="$HOME/Documents/stock/strategy.md"
```

If `STOCK_STRATEGY_PATH` is not set, the skill defaults to `~/Documents/stock/strategy.md`.

---

## Industry Classification（⚠️ 核心规则：行业决定宏观敏感度）

**不同行业的股票对宏观事件的敏感度完全不同。** 策略分析时必须按行业分类判断宏观事件的影响范围，不能一锅端。

### 行业分组与宏观敏感度矩阵

| 行业组 | 包含板块 | 受影响的关键宏观事件 | 不受影响的事件 |
|--------|---------|-------------------|-------------|
| **科技/半导体** | AI算力、芯片、消费电子、智能驾驶、通信 | 英伟达财报、Google I/O、OpenAI发布会、苹果WWDC、美债利率（估值折现率）、半导体周期 | 油价、厄尔尼诺、农产品价格 |
| **新能源/电力** | 光伏、储能、风电、锂电、电网 | 美债利率、IRA政策、铜价、欧盟碳关税 | 英伟达财报、AI产品发布 |
| **化工/石化** | 化纤、煤化工、烯烃、化塑 | 油价（WTI/布伦特）、伊朗地缘、OPEC+产量决策、人民币汇率 | 英伟达财报、Google I/O、AI产品发布 |
| **军工/航天** | 商业航天、碳纤维、航空发动机 | 国防预算、星网/铱星招标、中美关系、台海局势 | 油价短期波动、AI产品发布 |
| **消费/医药** | 白酒、食品、中药、创新药 | CPI/PPI、消费刺激政策、医保集采、利率（估值） | 英伟达财报、OPEC+决策 |
| **互联网/平台** | 腾讯、阿里、美团、小米 | 美债利率（港股估值压制）、监管政策、AI商业化进展、中美关系 | 油价、OPEC+ |
| **金融** | 券商、银行、保险 | 央行利率、LPR、社融、M2、PMI | 英伟达财报、AI产品发布 |

### 判断规则

1. **英伟达财报** → 仅直接影响科技/半导体+互联网（AI投资预期），**不影响化工/消费/金融**
2. **美债利率** → 影响所有港股（估值折现率）+ 高估值成长股，对低估值A股影响小
3. **油价** → 影响化工/石化，但**必须区分方向**（见下方化工子类型规则），**不影响科技**
4. **AI产品发布（Google/OpenAI/Anthropic）** → 仅影响科技/半导体+互联网，**不影响化工/消费/金融**
5. **OPEC+决策** → 仅影响化工/石化+交运，**不影响科技**

### ⚠️ 化工/石化子类型：油价影响方向相反

**化工/石化不是铁板一块**，必须区分油头下游和煤头竞品：

| 子类型 | 标的示例 | 油价↑影响 | 油价↓影响 | 传导路径 |
|--------|---------|----------|----------|---------|
| **油头下游**（PTA/涤纶） | 桐昆股份 | 🔴 成本压力 | ✅ 利好 | 油价↓→PTA成本↓→价差扩大→利润改善 |
| **煤头竞品**（煤制烯烃） | 宝丰能源 | ✅ 相对改善 | 🔴 利空 | 油价↓→油头烯烃成本↓→煤头竞争力削弱 |

**判断规则**：不能因为"都叫化工"就认为油价影响方向一致。看成本结构——是吃油（油头下游）还是跟油竞争（煤头）。

### 宏观数据缺失处理规则

- **Tier 1 指标获取失败** → 标注"获取失败，不作为本次判断依据"，并建议补用前一日数据
- **港股实时价缺失** → 标注"港股实时缺失"，浮亏列写"待获取"，不以估算值参与判断
- **禁止行为**：用美股 ADR 昨收推算港股实时价、用前一日浮亏冒充当日数据、无声略过数据缺失

---

## Macro Context Data Sources（⚠️ 策略检查前必须获取宏观上下文）

策略检查时，不能只看个股行情，**必须先获取当日宏观上下文**。

### Tier 1：必须获取（每次策略检查）

| 数据 | 数据源 | 获取方式 | 说明 |
|------|--------|---------|------|
| **美债10Y利率** | 英为财情 | curl+正则解析 | 港股估值锚点，>4.5%压制港股科技估值 |
| **中国10Y国债利率** | akshare | `ak.bond_china_yield()` | A股利率锚点 |
| **油价WTI** | 英为财情 | curl+正则解析 | 化工/石化板块核心变量 |
| **今日财经日历** | 东方财富datacenter | ak/eastmoney API | 获取当日宏观事件 |

### Tier 2：按需获取（持仓涉及该行业时获取）

| 数据 | 数据源 | 获取方式 | 适用行业 |
|------|--------|---------|---------|
| **个股公告（A股）** | 东方财富 | `np-anotice-stock.eastmoney.com` API | 所有A股持仓 |
| **持仓财报披露日** | 东方财富datacenter | `ak.stock_report_disclosure()` | 所有持仓 |
| **美股科技财报日历** | WebSearch | 搜索"earnings calendar" | 科技/半导体持仓 |
| **AI/科技行业事件** | WebSearch | 搜索"tech events 2026" | 科技/半导体+互联网持仓 |
| **化工行业事件** | WebSearch | 搜索"OPEC+ oil geopolitics" | 化工/石化持仓 |
| **军工/航天事件** | WebSearch | 搜索"商业航天 发射 招标" | 军工/航天持仓 |
| **铜价/大宗商品** | 英为财情 | curl+正则解析 | 新能源/电力持仓 |

### Tier 3：每周获取一次

| 数据 | 数据源 | 获取方式 | 说明 |
|------|--------|---------|------|
| **美联储利率预期** | akshare | `ak.macro_bank_usa_interest_rate()` | 下次议息日期+市场预期 |
| **美国CPI** | akshare | `ak.macro_usa_cpi_yoy()` | 通胀趋势 |
| **中国PMI** | akshare | `ak.macro_china_pmi()` | 制造业景气度 |

### 财经日历获取代码（Python 示例）

```python
import urllib.request, ssl, json, re

ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

def fetch_us_treasury_10y():
    url = 'https://cn.investing.com/rates-bonds/u.s.-10-year-bond-yield'
    req = urllib.request.Request(url, headers={
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36'
    })
    resp = urllib.request.urlopen(req, context=ctx, timeout=10)
    html = resp.read().decode('utf-8')
    match = re.search(r'data-test="instrument-price-last"[^>]*>([^<]+)<', html)
    return float(match.group(1).strip()) if match else None

def fetch_stock_notices(code):
    url = f'https://np-anotice-stock.eastmoney.com/api/security/ann?cb=&sr=-1&page_size=5&page_index=1&ann_type=SHA&stock_list={code}&f_node=0&s_node=0'
    req = urllib.request.Request(url, headers={'Referer': 'https://data.eastmoney.com/', 'User-Agent': 'Mozilla/5.0'})
    resp = urllib.request.urlopen(req, context=ctx, timeout=10)
    data = json.loads(resp.read().decode('utf-8'))
    return data.get('data', {}).get('list', [])[:5] if data.get('data') else []
```

---

## Save Mode

### When to Use
- Immediately after completing any stock investment analysis (e.g., DCF valuation, Buffett framework, technical analysis, or any investment report).
- The user says "保存策略", "把策略记下来", "跟踪这个票", "save strategy", "track this stock", or any variant.

### Extraction Rules
From the analysis text, extract these fields. If a field is not mentioned, mark it as "未明确" rather than fabricating.

| Field | Description | Example |
|-------|-------------|---------|
| `stock_code` | Stock code | 688111 |
| `stock_name` | Stock name | 金山办公 |
| `industry_group` | 行业分组（见上方行业映射表） | 科技/半导体 |
| `analysis_date` | Analysis date | 2026-04-22 |
| `recommendation` | Core recommendation | 建议观察，等待更好入场时机 |
| `entry_conditions` | Entry conditions (price/valuation/events) | PE≤45-50；股价180-200元 |
| `stop_loss` | Stop-loss condition | 跌破前期低点或技术位 |
| `target_price` | Target price / take-profit range | 机构目标价400元 |
| `exit_signals` | Exit signals (assumption invalidation) | 月活环比下滑；市场份额跌破80% |
| `position_size` | Suggested position size | 轻仓试探 |
| `time_horizon` | Holding period | 3年 |
| `key_assumptions` | Key assumptions | AI商业化持续推进 |
| `open_questions` | Open questions | 付费转化率是多少 |
| `current_price` | Price at analysis | ~250元 |
| `current_pe` | PE at analysis | 62.46 |
| `market_cap` | Market cap at analysis | 1147亿 |

### Output Format
Append a new section to the strategy file (path from `STOCK_STRATEGY_PATH` env var) using the exact format defined in `references/strategy_format.md`. Use Markdown table for entry conditions so they can be machine-parsed later.

### Important
- If the stock already exists in the strategy file, update the existing section rather than duplicating it. Preserve historical entries by appending a date-stamped update note.
- **必须标注行业分组**——在个股条目头部增加 `行业分组: 科技/半导体` 字段
- After saving, briefly confirm to the user what was saved.

## Check Mode

### When to Use
- User says "检查策略", "核对建仓条件", "看看哪些票可以买", "策略跟踪", "check strategy", or any variant.
- Can be invoked standalone without any prior analysis.

### Steps

**Step 0：获取宏观上下文（⚠️ 必须先于个股检查完成）**

在检查任何个股之前，必须先获取当日宏观上下文。按行业分组判断哪些宏观事件需要关注。

1. **获取Tier 1数据**：美债10Y利率、中国10Y国债利率、油价WTI
2. **按持仓行业确定需要获取的Tier 2数据**：
   - 有科技/半导体持仓 → 搜索本周AI/科技行业事件
   - 有化工/石化持仓 → 搜索本周OPEC+/油价/地缘事件
   - 有军工/航天持仓 → 搜索本周航天发射/招标/政策事件
   - 有新能源持仓 → 获取铜价、储能政策
3. **获取持仓股票公告**（A股用东方财富API，港股用WebSearch）
4. **汇总当日宏观事件日历**

**⚠️ 关键原则：宏观事件仅影响同行业组的持仓股票！**
- 英伟达财报 → 只影响科技/半导体+互联网持仓，**不影响化工/石化**
- OPEC+决策 → 只影响化工/石化持仓，**不影响科技**
- 美债利率 → 影响港股，对A股低估值股影响小

**Step 1：读取前置文件**

读取策略文件（`STOCK_STRATEGY_PATH`）和当日相关记忆文件。

**Step 2：按行业分组获取个股行情**

按行业分组批量展示个股行情和关键信号。

**Step 3：评估建仓/止损条件**

1. Evaluate each `entry_conditions` row: compare current value vs target value.
2. Mark each condition as `✅ 已触发`, `❌ 未触发`, or `⚠️ 接近触发`.
3. **按需读取投资研究报告**：条件已触发或用户关注的股票读取，条件未触发且无用户提及的不读。
4. Check if any `exit_signals` have occurred.

**Step 4：结合宏观上下文给出操作建议**

1. 将宏观事件的影响范围与持仓行业匹配
2. 仅对受影响的股票给出"等XX事件后再操作"的建议
3. 不受影响的股票按原策略执行，不受外部事件约束
4. 输出操作建议时明确标注"受XX事件影响"或"不受XX事件影响"

### Output Format

```markdown
## 策略核对报告 — YYYY-MM-DD

### 当日宏观上下文

| 宏观指标 | 数值 | 变化 | 影响行业 |
|---------|------|------|---------|
| 美债10Y | 4.65% | +5bp | 港股/高估值成长 |
| 中国10Y国债 | 1.73% | -1bp | A股利率锚 |
| WTI原油 | $62.5 | -1.2% | 化工/石化 |

### 本周关键宏观/行业事件

| 日期 | 事件 | 影响行业 | 受影响持仓 |
|------|------|---------|----------|

### 科技/半导体组
| 股票 | 当前价 | 涨跌 | 关键信号 | 受宏观影响 |
|------|--------|------|---------|----------|

### 化工/石化组
| 股票 | 当前价 | 涨跌 | 关键信号 | 受宏观影响 |
|------|--------|------|---------|----------|

### 操作建议（按优先级）
1. 🔴 XX股票：...
2. ✅ XX股票：...

### ⚠️ 操作状态三态标注规范

**所有报告中涉及操作的文字，必须严格区分三种状态，绝不可混用：**

| 状态 | 含义 | 使用场景 | 示例表述 |
|------|------|---------|---------|
| **`已触发`** | 价格/条件到了 | 股价触及目标线、止损线、均线破位等 | "28目标已触发" |
| **`建议执行`** | 系统建议动作 | 需要用户决策并下单的动作 | "建议冲28.5-29减仓" |
| **`已执行`** | 用户确认成交后 | ⚠️ **只有用户亲口确认成交后才能写** | "已减仓200股@28.70" |

**禁止行为**：
- ❌ 写"减仓执行中" → 你以为自己下单了，其实没有
- ❌ 写"✅ 执行" → 除非用户确认过成交
- ❌ 写"止损上移 ✅ 执行" → 你没权限下单，只能写"建议"
- ❌ 在方案中暗示动作已经完成

**正确做法**：
- ✅ 写"已触发，建议执行"（明确界限）
- ✅ 写"待减仓后执行止损上移"（动作依赖关系清晰）
- ✅ 在触发评估表里标注"⏳ 待执行"而非"✅ 已执行"
```

### strategy.md 更新规则

**日常（每次策略检查）**：只更新顶部「当日盘中跟踪」汇总表 → **1 次 Edit**

- ⚠️ **禁止逐只股票追加历史跟踪**：多次 Edit 浪费 Token，个股区保持静态（核心策略 + ≤2 条历史跟踪）
- 策略检查完 → 写顶部汇总表（1 Edit）→ 完成

**周五归档**（仅周五）：
1. 删除超过 2 天的顶部汇总表 → 追加到 backup 文件
2. 更新 header 日期 → 总计 2-3 次 Edit

**调仓时**：建仓/减仓/清仓 → 更新顶部汇总 + 个股状态字段（1-2 Edit）

### 低吸失效→右侧切换规则

**场景**：第一笔买入后股价快速上涨，原定低位加仓档（第二、三档）永远到不了。

**核心原则**：不能把所有低吸计划无脑上移追着买；必须切换成右侧加仓方案。

#### 规则1：第一笔必须明确仓位性质

建仓方案必须标注：
- **「第一档标准仓」**：按计划买够原定仓位（如10%）
- **「半仓试错」**：只买原定仓位的50%（如5%），事后底仓太轻无法补救

#### 规则2：上涨8-12%后，原低位档自动失效

建仓后股价累计上涨 **8%-12%**：
- 原第二、三档低位加仓 **自动失效**，禁止追补
- 失效≠放弃 → 自动触发规则3（右侧补仓条件）
- 在策略检查报告中标注：`⚠️ 低位档已失效（涨幅xx%），等右侧确认`

#### 规则3：右侧补仓条件 = 突破关键压力后回踩不破

当低位档失效后，新增补仓只有一种方式：

| 步骤 | 条件 | 验证方法 |
|------|------|---------|
| 1. 突破关键压力 | 放量突破整数关口/前高/均线压力 | 量比>1.5 + K线实体收盘突破 |
| 2. 回踩确认 | 自然回落至突破位附近，缩量不破 | 回撤不超过突破价的2%，量比<1 |
| 3. 补仓触发 | 回踩位企稳后右侧加仓 | 15分钟K线在回踩位收阳或十字星 |

#### 规则4：做T回补单未成交 → 禁止当天追回

- 做T卖出后的回补挂单未成交 → **当天禁止追回**
- 允许次日：等15分钟回踩支撑位再挂，或开盘重新评估
- 策略检查报告中标注：`⚠️ 回补未成交，放弃日内追回`

#### 与正/倒金字塔的关系

| 原策略 | 涨幅检查 | 切换动作 |
|--------|---------|---------|
| 正金字塔（低吸） | >8% → 低位档失效 | 等右侧补仓条件（规则3），不追 |
| 倒金字塔（追趋势） | 加速中 | 本来就是右侧，用规则3加仓 |
| T操作回补 | 回补未成交 | 放弃当日（规则4） |

## Resources

- `references/strategy_format.md` — Exact Markdown schema for strategy.md entries.
- `scripts/save_strategy.py` — Script to parse and append strategy data to strategy.md.
- `scripts/check_strategy.py` — Script to read strategy.md, fetch prices, and evaluate conditions.

## Security & Guardrails

1. **Local file only**: Scripts only read/write a local Markdown file specified by `STOCK_STRATEGY_PATH`. No network calls by `save_strategy.py` or `check_strategy.py`.
2. **Market data source**: During check mode, market data is fetched via the agent's available financial data tools or web search. No private strategy content or position details are transmitted.
3. **No trading execution**: This skill provides analysis and recommendations only. It does NOT execute any trades or connect to brokerage accounts.
4. **No credential collection**: This skill only requires `STOCK_STRATEGY_PATH` (a file path). It never asks for API keys, tokens, passwords, or any other credentials.
5. **Data integrity**: The save script uses atomic write patterns and preserves historical entries.
6. **Path validation**: The scripts validate that `STOCK_STRATEGY_PATH` points to a `.md` file and will create parent directories if needed.

## Environment Variable Contract

| Variable | Purpose | Required | Set Via |
|----------|---------|----------|---------|
| `STOCK_STRATEGY_PATH` | Path to the strategy tracking Markdown file | No (defaults to `~/Documents/stock/strategy.md`) | Shell profile or agent config |

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| "No such file or directory" | `STOCK_STRATEGY_PATH` not set or points to invalid path | Set env var or rely on default |
| Strategy not found during check | Strategy file is empty or format is corrupted | Verify the file uses the format in `references/strategy_format.md` |
| Duplicate entries for same stock | Save mode ran twice without updating | The script auto-detects duplicates |
| Chinese characters garbled | Terminal encoding mismatch | Ensure UTF-8 (`export LANG=en_US.UTF-8` or `zh_CN.UTF-8`) |

## Release Notes

- **2.0.0** — Major update: Added industry classification system, macro context data sources, three-state action labeling, low-absorb→right-side switch rules, strategy.md update optimization rules, and chemical/petrochemical sub-type analysis.
- **1.1.0** — Fixed metadata consistency, added credential collection disclaimer, clarified market data query scope.
- **1.0.0** — Initial release. Save and check modes with Markdown-based tracking file.

## Links

- [Scheme Format Reference](references/strategy_format.md)
- [Save Script](scripts/save_strategy.py)
- [Check Script](scripts/check_strategy.py)

## Publisher

* **Publisher:** @dcsight
* **License:** MIT-0
* **Source:** https://github.com/dcsight/stock-strategy-tracker
