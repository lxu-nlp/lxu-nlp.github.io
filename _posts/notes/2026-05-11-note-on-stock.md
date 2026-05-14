---
layout: post
title: "Notes on Stock Options"
date: 2026-05-11
categories: [Notes]
tags: [economics, finance, stock]
math: true
hidden: true
---

# 金融策略 Study Notes：从期权基础（Options Basics）到市场场景玩法（Market Regime Playbook）

---

## 如何阅读这份 notes

建议按这个顺序读：

1. **先理解风险形状**：做多、做空、期权、margin 到底分别改变什么。
2. **再理解基础工具**：现金、短债、债券久期、权益 ETF、因子 ETF。
3. **再看期权策略**：先看买 call/put，再看 spread、covered call、collar、cash-secured put。
4. **再看 ETF 包装**：JEPI/JEPQ/QYLD、PUTW、buffer ETF、tail-risk ETF、managed futures 等。
5. **最后按市场场景选择策略**：牛市、熊市、横盘、高波动、通胀、衰退、高估值 AI 市场。

**重点：不要先问“哪个工具收益最高”，而要先问“我想要什么收益曲线（payoff profile）以及我最多能亏多少”。工具只是实现风险形状的方式。**

---

# Part I. 基础概念（Foundations）

## 1. 总框架（Framework）：金融工具到底在解决什么问题？

金融策略不是简单的“看涨”或“看跌”。一个策略至少包含四件事：

1. **方向判断（Directional view）**：看涨、看跌、横盘、震荡、危机、通胀、降息、衰退等。
2. **收益曲线（Payoff profile）**：无限上涨、上涨封顶、下跌保护、收权利金、凸性暴跌保护等。
3. **成本（Cost）**：权利金、机会成本、融资成本、跟踪误差、税务、滑点、bid-ask spread。
4. **失败模式（Failure mode）**：策略在什么情况下亏钱、跑输或被迫平仓。

一句话：

> 股票给你方向暴露；margin 放大方向暴露；期权改变收益曲线形状；ETF 把某类策略产品化。

---

## 2. 三个最基础概念（Core Concepts）

| 概念 | 回答的问题 | 例子 |
|---|---|---|
| 做多（Long）/ 做空（Short） | 你押哪个方向？ | 做多=希望涨；做空=希望跌 |
| 期权（Options）：Call / Put | 你用什么合约表达方向或转移风险？ | 买 call 看涨；买 put 防跌/看空 |
| Margin / 保证金（Margin）/ 杠杆（Leverage） | 你用多少钱控制多大仓位？亏损时如何保证履约？ | 融资买股票、融券做空、卖期权保证金 |

核心区别：

> 多空是方向；期权是权利和义务；margin 是资金杠杆和履约保证机制。

---

## 3. 做多（Long）与做空（Short）

### 3.1 做多（Long）

做多就是买入资产，希望价格上涨。

例子：

- 100 元买入股票；
- 涨到 120，赚 20；
- 跌到 80，亏 20。

| 项目 | 做多股票（Long stock） |
|---|---|
| 方向 | 看涨（Bullish） |
| 最大亏损 | 理论上最多亏到 0 |
| 最大收益 | 理论上很大 |
| 是否有到期日 | 没有 |
| 是否强平 | 现金买入通常不会；融资买入可能会 |

### 3.2 做空（Short）

传统股票做空流程：

1. 向券商借股票；
2. 现在卖出；
3. 未来低价买回来还给券商。

例子：

- 借股票，在 100 元卖出；
- 后来跌到 70；
- 以 70 买回来还掉；
- 赚 30。

但如果股票涨到 150，你需要以 150 买回来还，亏 50。

**重点：股票理论上可以无限上涨，所以裸做空（naked short selling）理论亏损无限，并且可能遇到逼空（short squeeze）和强平（forced liquidation）。**

---

## 4. 期权基础（Options Basics）：Call、Put、买方和卖方

期权是一张合同。

> 买期权的人付权利金，获得一个“选择权”；卖期权的人收权利金，但承担相应义务。

### 4.1 基础术语（Basic Terminology）

| 词 | English term | 含义 |
|---|---|---|
| 标的资产 | Underlying asset | 股票、ETF、指数、期货等 |
| 看涨期权 | Call option | 买方有权按行权价买入标的 |
| 看跌期权 | Put option | 买方有权按行权价卖出标的 |
| 行权价 | Strike price | 约定买入或卖出的价格 |
| 到期日 | Expiration date | 期权有效期结束日 |
| 权利金 | Premium | 买方付给卖方的钱 |
| 行权 | Exercise | 买方使用权利 |
| 实值 | In the money / ITM | 期权已有内在价值 |
| 平值 | At the money / ATM | 标的价格接近行权价 |
| 虚值 | Out of the money / OTM | 期权暂无内在价值 |
| 隐含波动率 | Implied volatility / IV | 期权价格反推的未来波动率预期 |

### 4.2 Call：看涨期权（Call Option）

Call 给买方一个权利：

> 未来可以按约定价格买入某个资产。

例子：

- 股票现在 100；
- 买入一个行权价 110 的 call；
- 如果到期股票涨到 130；
- 买方可以按 110 买入；
- 这个权利就有价值。

直觉：

> Call 像上涨门票。你花一笔小钱，买一个未来上涨的参与权。

买 Call 的动机：

| 动机 | 解释 |
|---|---|
| 杠杆看涨 | 用较少资金博取上涨收益 |
| 锁定未来买入价格 | 例如企业担心原材料价格上涨 |
| 控制最大亏损 | 最多亏掉权利金 |
| 事件交易 | 财报、政策、并购等短期催化 |

卖 Call 的动机：

| 动机 | 解释 |
|---|---|
| 认为不会大涨 | 没涨过行权价，call 作废，卖方赚权利金 |
| 持有股票想收现金流 | Covered call，即持有底仓同时卖 call |
| 赚取波动率溢价 | 像保险公司一样收保费 |

### 4.3 Put：看跌期权（Put Option）

Put 给买方一个权利：

> 未来可以按约定价格卖出某个资产。

例子：

- 股票现在 100；
- 买入一个行权价 90 的 put；
- 如果股票跌到 60；
- 你仍然可以按 90 卖出；
- 这就像下跌保险。

买 Put 的动机：

| 动机 | 解释 |
|---|---|
| 给持仓买保险 | 持有股票但担心大跌 |
| 看空但不想做空股票 | 最大亏损限于权利金 |
| 对冲组合风险 | 机构常用指数 put 对冲市场暴跌 |

卖 Put 的动机：

| 动机 | 解释 |
|---|---|
| 认为不会大跌 | 没跌破行权价就赚权利金 |
| 愿意低价买入 | 本来就想 90 买，卖 put 等于收钱挂低价买单 |
| 收保险费 | 承担别人不想承担的下跌风险 |

例子：

- 股票现在 100；
- 卖出 90 put，收 2；
- 如果股票跌到 60；
- 买方会让你按 90 接盘；
- 你相当于高价接盘。

---

## 5. 期权 Greeks：为什么方向对了也可能亏钱

| Greek | 中文理解 | 说明 |
|---|---|---|
| Delta | 方向敏感度 | 标的价格变动 1 单位时期权价格大约变动多少 |
| Gamma | Delta 的变化速度 | 标的价格变化时，delta 如何加速变化 |
| Theta | 时间损耗 | 时间流逝导致期权价值衰减 |
| Vega | 波动率敏感度 | IV 变化对期权价格的影响 |

例子：

- 你买 call 看涨；
- 股票确实涨了；
- 但涨得不够快，theta 消耗很多；
- 同时财报后 IV 下跌，vega 受损；
- 最后可能仍然亏钱。

**重点：期权不是“方向对就赚钱”。期权价格同时受到方向、时间、波动率和行权价影响。方向只是其中一个变量。**

---

## 6. 买期权为什么天然有杠杆（Embedded Leverage）

买期权通常不用借钱，但天然有杠杆。

原因是：

> 你用较小的权利金，控制一个较大的标的资产风险敞口。

例子：

- 股票 100；
- 直接买股票需要 100；
- 买 call 可能只需要 5；
- 如果股票涨到 120，call 可能从 5 涨到 20 甚至更多。

股票涨 20%，call 可能涨 300%。

但反过来：

- 如果股票没涨到足够高；
- 或涨得太慢；
- 或到期时间太近；
- call 可能归零。

| 工具 | 敏感方式 |
|---|---|
| Margin | 线性放大，比如 2 倍、3 倍 |
| 买期权 | 非线性放大，受价格、时间、波动率、行权价共同影响 |

一句话：

> Margin 是线性放大器；期权是非线性放大器。

**重点：期权不是“更高级的融资买股票”。买期权的亏损通常限定在权利金，但方向判断对了，也可能因为涨得不够快或 IV 下降而亏钱。**

---

## 7. Margin、保证金（Margin）与强平（Forced Liquidation）

### 7.1 Margin 是什么

Margin 可以理解成两件事：

1. 借钱或借证券交易；
2. 交保证金，证明你亏损时赔得起。

常见场景：

- 融资买股票；
- 融券做空股票；
- 期货交易；
- 卖期权；
- 杠杆 ETF 或衍生品交易。

### 7.2 Margin 做多例子

你有 10 万本金，借 10 万，买 20 万股票。

如果股票涨 10%：

- 仓位赚 2 万；
- 你的本金收益约 20%。

如果股票跌 10%：

- 仓位亏 2 万；
- 你的本金亏约 20%。

Margin 的核心：

> 放大收益，也放大亏损。

### 7.3 什么是强平

强平 = 强制平仓。

它发生在保证金不足时。

逻辑是：

> 你用了借来的钱、借来的证券，或承担了未来义务。券商/交易所担心你亏到还不起，于是要求你补保证金；如果你不补或行情太快，它会强制平掉你的仓位。

强平最麻烦的地方：

- 可能在最差的位置被迫卖出；
- 不能继续等待反弹；
- 极端情况下还可能需要补钱；
- 它是别人替你提前结束仓位。

### 7.4 强平和期权归零的区别

| 项目 | 强平 | 买入期权归零 |
|---|---|---|
| 发生机制 | 保证金不足，被券商强制平仓 | 到期时合约没有价值 |
| 是否借钱/承担义务 | 通常是 | 买入期权不是 |
| 是否可能被提前出局 | 是 | 买方可持有到期 |
| 是否可能补钱 | 可能 | 买入期权通常不会 |
| 最大亏损 | 可能超过初始本金 | 通常限于权利金 |
| 核心风险 | 被迫在不利价格出局 | 时间到了，权利失效 |

直觉：

- 期权归零：你买的电影票过期没用，票钱没了；
- 强平：你用贷款买资产，抵押物不足，券商强制处置。

**重点：买入期权归零通常是“预先知道最大亏损”；margin 强平则可能是在最差时点被迫退出，甚至出现补保证金压力。二者的心理和资金风险完全不同。**

---

## 8. 期权原始用途（Original Use Cases）

期权最原始的用途是风险转移，不是一开始为赌博设计的。

| 场景 | 风险 | 可用期权 |
|---|---|---|
| 农民未来要卖小麦 | 怕价格下跌 | 买 put，锁定最低卖价 |
| 航空公司未来要买燃油 | 怕油价上涨 | 买 call，锁定最高买入价 |
| 基金持有股票组合 | 怕市场暴跌 | 买指数 put 做组合保险 |
| 持有股票的人想收现金流 | 短期不认为会大涨 | 卖 call，形成 covered call |

---

# Part II. 基础资产配置工具（Asset Allocation Building Blocks）

## 9. 现金、货币基金、短债（Cash / Money Market / Short-duration Bonds）

**目标**：保本、流动性、等待补偿、保留弹药。

**机制**：持有现金、货币基金、Treasury bills、浮动利率国债或超短债基金。

| ETF | 策略类型 | 说明 |
|---|---|---|
| SGOV | 0-3 个月美国国债 | 久期很短，接近现金 |
| BIL | 1-3 个月美国国债 | T-bill 曝露 |
| SHV | 短期美国国债 | 短久期 |
| USFR | 浮动利率美国国债 | 短端利率高时较有吸引力 |
| TFLO | 浮动利率美国国债 | 类似 USFR |
| MINT | 增强超短债 | 比纯国债多一点信用/主动管理风险 |
| JPST | 超短收益 | 主动管理，非纯国债 |

例子：

如果你有 100,000 美元，觉得 AI 股票太贵：

- 80,000 放 T-bills / SGOV 类工具；
- 20,000 留作分批买入；
- 等回调 10%-20% 或财报确认后再加。

适合：

- 市场估值高；
- 想保留买入权；
- 短端利率有吸引力；
- 不想 FOMO 追高。

风险：

- 强牛市中可能跑输；
- 利率下降后再投资收益降低；
- 非纯国债超短债仍有信用风险。

---

## 10. 债券久期选择（Bond Duration Positioning）

| 策略 | ETF 例子 | 适合场景 | 主要风险 |
|---|---|---|---|
| 短债 | SGOV, BIL, SHV | 高短端利率、不确定市场 | 降息时上行弹性有限 |
| 中期国债 | IEF, VGIT | 衰退风险、降息周期 | 利率上升时亏损 |
| 长期国债 | TLT, EDV, ZROZ | 通缩衰退、避险、利率大跌 | 久期风险很高 |
| TIPS | TIP, SCHP | 通胀超预期 | 实际利率上升会压价格 |
| 综合债券 | AGG, BND | 平衡配置 | 久期 + 信用利差风险 |

核心：

- 短债 = 价格稳定、凸性弱；
- 长债 = 降息时弹性强、加息时伤害大。

---

## 11. 普通权益 ETF 与因子 ETF（Equity Beta and Factor ETFs）

### 11.1 普通权益 ETF（Plain Equity Beta）

| 曝露 | ETF 例子 |
|---|---|
| 美国全市场 | VTI, ITOT |
| S&P 500 | SPY, IVV, VOO |
| Nasdaq 100 | QQQ, QQQM |
| 半导体 | SMH, SOXX |
| 全球 ex-U.S. | VXUS, IXUS |
| 发达市场 | EFA, IEFA |
| 新兴市场 | EEM, VWO |

适合：

- 长期持有；
- 不想处理期权复杂性；
- 想吃市场 beta。

风险：

- 完整下跌风险；
- 估值压缩；
- 市值加权 ETF 可能高度集中。

### 11.2 因子 ETF（Factor ETFs）

| 因子 | ETF 例子 | 典型适用场景 |
|---|---|---|
| Quality | QUAL, SPHQ | 后周期、不确定环境 |
| Low volatility | USMV, SPLV | 防守、震荡市 |
| Value | VTV, IWD, VLUE | 通胀、周期复苏 |
| Momentum | MTUM | 趋势延续 |
| Dividend growth | VIG, DGRO, SCHD | 收益 + 质量 |
| Small cap | IWM, VB | 早周期复苏、国内经济走强 |

例子：

如果不想全买高 PE AI 股，可以用 QUAL 或 USMV 降低组合估值和集中度。

---

# Part III. 核心期权策略（Core Option Strategies）

## 12. 方向和凸性策略（Directional / Convex Strategies）

### 12.1 Long call：买入看涨期权（Buying Calls）

**观点**：看涨，希望用有限资金参与上涨。

**结构**：买 call。

例子：

- 股票价格：100；
- 买入 6 个月 110 call，付 5；
- 最大亏损：5；
- 到期盈亏平衡：115。

| 到期价格 | 盈亏 |
|---:|---:|
| 90 | -5 |
| 110 | -5 |
| 120 | +5 |
| 140 | +25 |

适合：

- 预期强上涨；
- IV 不太贵；
- 想控制最大亏损。

风险：

- 时间损耗；
- IV crush；
- 到期前必须涨得足够多。

ETF/产品思路：

- SWAN 这类产品可粗略理解为债券底仓 + 长期期权式权益上行曝露，虽然不是简单 long call ETF。

### 12.2 Long put：买入看跌期权（Buying Puts）

**观点**：看空，或想给持仓买保险。

**结构**：买 put。

例子：

- 股票价格：100；
- 买入 3 个月 90 put，付 3；
- 如果股票跌到 70，put 有较大价值；
- 如果股票不跌，最多亏掉 3。

适合：

- 看空但不想裸做空；
- 持仓保护；
- 事件风险对冲。

风险：

- 保护成本高；
- 时间损耗；
- 跌得不够多也可能亏。

### 12.3 Call spread：看涨价差（Bull Call Spread）

**观点**：温和到中等看涨，希望降低买 call 成本。

**结构**：买较低行权价 call，卖较高行权价 call。

例子：

- 股票 100；
- 买 105 call，花 6；
- 卖 130 call，收 2；
- 净成本 4；
- 最大亏损 4；
- 最大收益 = 130 - 105 - 4 = 21。

适合：

- 认为有上涨，但不是无限上涨；
- 单买 call 太贵；
- 想定义最大亏损。

风险：

- 上涨封顶；
- 仍可能亏掉全部权利金；
- 行权价选择很重要。

高估值 AI 市场用法：

- 90%-95% 放 SGOV/BIL/USFR；
- 5%-10% 用 QQQ/SMH/SOXX 做 3-6 个月 call spread；
- 目标是“保本金 + 吃一段上行”。

### 12.4 Long straddle / strangle：跨式 / 宽跨式（Volatility Buying）

**观点**：预期大波动，但方向不确定。

**结构**：

- Straddle：买同一行权价 call + put；
- Strangle：买 OTM call + OTM put。

例子：

- 股票 100；
- 买 100 call，花 6；
- 买 100 put，花 5；
- 总成本 11；
- 到期需要高于 111 或低于 89 才盈利。

适合：

- 预期有大行情；
- IV 不贵；
- 事件波动被低估。

风险：

- 成本高；
- 双边 theta 损耗；
- 财报前 IV crush 风险很大。

---

## 13. 防守和保护策略（Protection Strategies）

### 13.1 Protective put：保护性看跌（Portfolio Insurance）

**观点**：已有持仓，想买保险。

**结构**：持有股票/ETF + 买 put。

例子：

- 持有股票，价格 100；
- 买 90 put，花 3；
- 如果股票跌到 70，90 以下损失由 put 对冲；
- 保护成本为 3。

适合：

- 已有较大浮盈；
- 不想卖出但怕大跌；
- 财报、宏观事件或高估值阶段。

风险：

- Put 很贵时保护成本高；
- 频繁买保护会拖累长期收益；
- 保护只覆盖到期日前。

ETF 例子：

| ETF | 风格 |
|---|---|
| TAIL | 尾部风险保护，常结合 Treasuries 和 put 曝露 |
| QTR | Nasdaq 100 tail-risk 风格例子 |
| CAOS | 凸性/尾部风险取向 ETF 例子 |

### 13.2 Collar：领口策略（Protective Collar）

**观点**：已有持仓；想防跌，也愿意放弃部分上涨。

**结构**：持有股票 + 买 put + 卖 call。

例子：

- 持有 ETF，价格 100；
- 买入 95 put，跌破 95 有保护；
- 卖出 110 call，涨破 110 收益封顶；
- 卖 call 的收入补贴买 put 成本。

本质：

> 我不想亏太多，也愿意放弃赚太多。

| 策略 | 下跌保护 | 上涨空间 | 成本/收入 |
|---|---|---|---|
| 直接买 ETF | 无 | 完整 | 无额外成本 |
| Covered call | 无，只是权利金缓冲 | 被封顶 | 收权利金 |
| Protective put | 有 | 完整 | 付保险费 |
| Collar | 有 | 被封顶 | put 成本被 call 收入抵消一部分 |

ETF 例子：

- NUSI：历史上使用类似 Nasdaq 100 collar / 风险管理收入策略。
- QRMI / XRMI：风险管理收入策略，使用 option overlay。
- Innovator / FT Vest defined outcome ETF：通过期权包定义下跌 buffer 和上涨 cap。

**重点：collar 不是免费午餐。你用放弃上方极端收益，换取下方保护和更低保险成本。**

---

## 14. 收益增强和区间策略（Income and Range Strategies）

### 14.1 Covered call：备兑看涨（Long Underlying + Short Call）

**观点**：温和看多或横盘，想收权利金。

**结构**：持有股票/ETF + 卖 call。

例子：

- 持有 ETF，价格 100；
- 卖出行权价 105 的 call；
- 收到 2 元权利金。

到期情况：

| 到期价格 | 结果 |
|---|---|
| 跌到 90 | 期权不执行；底仓亏 10；权利金缓冲 2；净亏约 8 |
| 涨到 103 | 期权不执行；底仓赚 3；权利金赚 2；总赚约 5 |
| 涨到 120 | 期权被执行；105 以上收益让给别人；赚约 5 + 2 |

本质：

> 把未来一部分上涨空间卖掉，换取当期权利金现金流。

适合：

- 横盘；
- 小涨；
- 小跌；
- 波动率较高但趋势不强。

不适合：

- 单边大牛市；
- 持续熊市；
- 下跌后快速 V 型反弹。

ETF 例子：

| ETF | 策略风格 |
|---|---|
| JEPI | S&P 500 风格权益组合 + option overlay / ELNs 收益增强 |
| JEPQ | Nasdaq 100 / 科技成长风格权益 + option overlay |
| QYLD | Nasdaq 100 covered call，常系统性卖 ATM 附近 call |
| XYLD | S&P 500 covered call |
| RYLD | Russell 2000 covered call |
| PBP | S&P 500 buy-write 风格 |
| DIVO | 红利股票 + 战术 covered call |

**重点：covered call 的收入来自“卖掉一部分未来上涨空间”。它适合横盘或慢牛，不是熊市防跌工具，也不是无风险收息工具。**

### 14.2 JEPQ / QDTE 的原理（Option Income ETFs）

这类产品可以粗略理解为：

> 股票或指数底仓 + 卖出看涨期权或类似结构 + 分派权利金收益。

JEPQ 大致是：

- 持有偏纳斯达克科技股的组合；
- 通过卖出看涨期权或类似结构获取权利金；
- 将部分收益定期分派。

QDTE 更强调短期期权收益，分派频率更高，策略更依赖短周期/接近 0DTE 的期权逻辑。

核心交换：

| 你得到 | 你放弃 |
|---|---|
| 权利金现金流 | 一部分大涨收益 |
| 横盘市场收益增强 | 单边牛市跑输指数 |
| 下跌时一点缓冲 | 熊市仍然会亏 |

**重点：高分派率（distribution yield）不等于高总回报（total return）。如果净值长期下滑，分派看起来很高，也可能只是把波动和本金风险包装成现金流。**

### 14.3 下行市场里 Covered Call 的问题

如果市场持续下跌：

- 底仓会跟着跌；
- 卖 call 收到的权利金只能缓冲一小部分；
- 如果市场安静地下跌，call 权利金可能不高；
- 如果基金在低位继续卖更低行权价的 call，之后反弹可能被封顶。

所以 covered call 不是熊市保护工具。

更准确定位：

> 震荡市收益增强产品，而不是熊市防跌产品。

### 14.4 Cash-secured put：现金担保卖 Put

**观点**：愿意低价买入，等待时收权利金。

**结构**：卖 put，同时保留足够现金接盘。

例子：

- 股票 100；
- 你想 85 买入；
- 卖出 3 个月 85 put，收 3；
- 如果股票高于 85，put 作废，你赚 3；
- 如果股票低于 85，你可能按 85 接盘，实际成本约 82。

适合：

- 你真的愿意在该价格买入；
- 有足够现金；
- IV 较高。

风险：

- 股票可能远低于行权价；
- 可能在坏市场被迫接盘；
- 权利金收入会掩盖权益下跌风险。

ETF 例子：

| ETF | 策略风格 |
|---|---|
| PUTW | 系统性 put-write，大盘 put 卖方策略 |

**重点：卖 put 的真实含义是“收保险费，承诺在下跌时接盘”。只有当你本来就愿意以该价格买入，并且有现金承接时，cash-secured put 才是等待策略；否则它只是伪装成收入的下跌风险。**

### 14.5 Put spread：看跌价差 / 定义风险卖 Put

**观点**：中性到温和看涨，想收权利金但限制尾部风险。

**结构**：卖较高行权价 put，买较低行权价 put。

例子：

- 股票 100；
- 卖 90 put，收 4；
- 买 75 put，花 1；
- 净收入 3；
- 最大收益 3；
- 最大亏损 = 90 - 75 - 3 = 12。

适合：

- 想收权利金；
- 认为标的不会跌破关键支撑；
- 希望最大亏损可控。

风险：

- 收益有限、亏损可能更大；
- 跳空下跌时亏损很快；
- 需要保证金和风控。

### 14.6 Iron condor：铁鹰策略（Range-bound Premium Strategy）

**观点**：预期区间震荡。

**结构**：卖 OTM call spread + 卖 OTM put spread。

例子：

- 股票 100；
- 卖 90 put，买 80 put；
- 卖 110 call，买 120 call；
- 收净权利金；
- 如果到期价格在 90-110 之间，盈利。

适合：

- 横盘市场；
- IV 高，但预计实际波动低；
- 没有重大催化剂。

风险：

- 单边突破会亏；
- 需要主动管理；
- 不适合财报/重大事件前无脑做。

### 14.7 Calendar spread：日历价差（Term Structure Strategy）

**观点**：短期平静，中期可能有波动；或想利用期限结构。

**结构**：卖短期期权，买长期期权，行权价相同或接近。

例子：

- 股票 100；
- 卖 1 个月 100 call；
- 买 3 个月 100 call。

适合：

- 近月 IV 偏高；
- 预期近月时间价值衰减；
- 想保留更长期方向期权。

风险：

- Greeks 复杂；
- 短期大波动可能伤害；
- 需要理解 IV term structure。

---

# Part IV. ETF 包装与另类工具（ETF Wrappers and Alternatives）

## 15. 期权 ETF 与结构化收益 ETF

### 15.1 Covered call ETFs

**目标**：把部分上涨空间转换成当前收入。

例子：JEPI、JEPQ、QYLD、XYLD、RYLD、PBP、DIVO。

适合：

- 横盘到慢牛；
- 重视现金流多于完整上涨；
- 能接受强牛市跑输指数。

注意：

- 高分派率不等于高总回报；
- 熊市仍会跌；
- 单边牛市会明显跑输普通指数 ETF。

### 15.2 Put-write ETFs

**目标**：系统性卖 put 收权利金。

例子：PUTW。

适合：

- 波动率较高；
- 市场横盘、温和上涨或跌后修复；
- 想通过承担下跌风险换取收入。

注意：

- 下跌风险类似权益；
- 不是现金或短债替代品。

### 15.3 Buffer / Defined outcome ETFs

**目标**：在一个 outcome period 内定义下跌 buffer 和上涨 cap。

发行商和例子：

- Innovator Defined Outcome ETFs：BJAN/BJUL 风格 buffer series，PJAN/PJUL 风格 power buffer series，UJAN/UJUL 风格 ultra buffer series。
- FT Vest Buffer ETFs：按月份/季度滚动的 defined outcome ETF 系列。

机制：

- 用 SPY、QQQ 等指数 ETF 的期权包构造收益曲线。

适合：

- 想参与权益上涨；
- 想要明确下跌缓冲；
- 能接受上涨封顶；
- 能理解 outcome period。

注意：

- buffer 和 cap 取决于买入日期和剩余期限；
- 中途买入的实际保护和上限可能不同于产品初始宣传；
- 费用和价差重要。

### 15.4 Tail-risk ETFs

**目标**：保护极端下跌。

| ETF | 风格 |
|---|---|
| TAIL | 尾部风险保护，常结合 Treasuries 和 put 曝露 |
| QTR | Nasdaq 100 tail-risk 风格例子 |
| CAOS | 凸性/尾部风险 ETF 例子 |

适合：

- 害怕崩盘；
- 波动率便宜；
- 想要组合保险。

注意：

- 尾部保护长期可能持续流血；
- 择时很难；
- 它是保险，不是主要收益来源。

---

## 16. 杠杆、反向、波动率与趋势工具

### 16.1 杠杆和反向 ETF（Leveraged and Inverse ETFs）

| ETF | 曝露 |
|---|---|
| TQQQ | 3x Nasdaq 100 多头 |
| SQQQ | 3x Nasdaq 100 反向 |
| SOXL | 3x 半导体多头 |
| SOXS | 3x 半导体反向 |
| SPXL | 3x S&P 500 多头 |
| SPXU | 3x S&P 500 反向 |
| QLD | 2x Nasdaq 100 多头 |
| SDS | 2x S&P 500 反向 |

适合：

- 短线战术；
- 强趋势、低反复；
- 小仓位。

风险：

- 每日重置；
- 路径依赖；
- 震荡市 volatility decay；
- 逆向走势中可能快速大亏。

**重点：杠杆 ETF 的倍数通常是“每日收益倍数”，不是长期收益倍数。震荡路径会产生波动率损耗（volatility decay），所以横盘大波动也可能亏钱。**

### 16.2 Volatility ETPs

**目标**：交易 VIX 期货曝露。

| ETF/ETN | 方向 |
|---|---|
| VIXY | 多头 VIX 期货 |
| UVXY | 杠杆多头 VIX 期货 |
| SVXY | 空头 VIX 期货 |

适合：

- 短期战术避险或投机；
- 对波动率期限结构有理解。

风险：

- VIX ETP 通常不适合长期持有；
- 期货展期损耗明显；
- 杠杆波动率产品亏损速度极快。

### 16.3 Managed futures / trend following

**目标**：跨商品、货币、利率、股指期货做趋势跟随，提供非传统相关性。

| ETF | 策略风格 |
|---|---|
| DBMF | Managed futures replication / active strategy |
| KMLM | 商品、货币、全球债券等趋势跟随 |
| CTA | Managed futures-style ETF |

适合：

- 通胀冲击；
- 商品趋势；
- 债券熊市；
- 股票危机但跨资产趋势明显。

风险：

- 横盘反复市场容易 whipsaw；
- 策略透明度有限；
- 表现依赖趋势持续性。

---

## 17. 另类、事件驱动和多资产策略

### 17.1 Merger arbitrage（并购套利）

**目标**：捕捉并购交易价与当前价格之间的 spread。

ETF 例子：MNA。

适合：并购活跃、spread 有吸引力、监管风险可控。

风险：交易失败会大跌；spread 太窄时收益有限；法律/监管/个案风险。

### 17.2 Long/short 或市场中性（Market Neutral）

**目标**：降低市场 beta，从相对价值中赚钱。

| ETF | 风格 |
|---|---|
| BTAL | Anti-beta / market-neutral 风格 |
| QAI | Multi-strategy liquid alternatives 风格 |

适合：市场贵、想降低相关性、想减少纯股票 beta。

风险：强牛市跑输；因子和持仓逻辑可能不直观。

### 17.3 Risk parity / Efficient core

**目标**：用股票、债券和期货提高资本效率或分散风险。

| ETF | 风格 |
|---|---|
| NTSX | 90/60 风格，美国股票 + 国债期货 |
| RPAR | Risk parity 多资产风格 |

适合：股票债券负相关有效、利率稳定或下降、想要平衡型组合。

风险：2022 式股债双杀会受伤；内含杠杆/期货曝露。

---

## 18. 商品、黄金和真实资产

### 18.1 黄金（Gold）

**目标**：对冲货币不稳定、危机、实际利率下降、信用风险。

ETF 例子：GLD、IAU、SGOL。

适合：实际利率下降、美元走弱、避险需求上升。

风险：没有现金流；实际利率上升时可能表现差。

### 18.2 大宗商品（Broad Commodities）

**目标**：通胀对冲和趋势曝露。

ETF 例子：DBC、PDBC、COMT。

适合：通胀超预期、商品供给冲击、美元走弱。

风险：futures roll yield、商品周期波动大、税务结构可能复杂。

### 18.3 REITs 和基础设施（Infrastructure）

| 曝露 | ETF 例子 |
|---|---|
| 美国 REITs | VNQ, SCHH, XLRE |
| 基础设施 | IFRA, PAVE, GRID |
| 公用事业 | XLU, VPU |

适合：利率压力缓解、想要稳定现金流、基建 capex 周期增强。

风险：利率敏感；行业监管和债务风险。

---

# Part V. 场景化策略选择（Market Regime Playbook）

## 19. 市场场景快速地图

| 市场场景 | 更适合的策略 | 通常要小心 |
|---|---|---|
| 强牛市 | 长期持有股票/ETF、成长 ETF、call spread、短期战术杠杆 ETF | Covered call 作为主仓、过重现金、过紧 collar |
| 慢牛 / 温和上涨 | 股票 + covered call、红利/质量、call spread | 深虚值 call 彩票、过度避险 |
| 横盘 | Covered call、put-write、iron condor、短债、区间交易 | 买入高 theta 损耗的裸期权 |
| 温和熊市 | Protective put、collar、低波 ETF、短债、目标价 cash-secured put | 裸多高 beta、杠杆多头 ETF |
| 急跌 / 危机 | 现金/T-bills、long put、put spread、tail-risk ETF、managed futures | 裸卖 put/call、流动性差的小盘票 |
| 高波动但无趋势 | Covered call、put-write、iron condor、期权收益 ETF | 无催化买贵期权 |
| 低波动 / 过度平静 | 小比例 cheap convexity、tail hedge、长期 put/call | 为一点权利金过度卖波动 |
| 通胀 / 商品冲击 | 商品、能源、managed futures、TIPS、价值/周期 | 长久期债券、无盈利支撑高估值成长 |
| 衰退 / 降息冲击 | 高质量债券、防御股票、黄金、现金、质量因子 | 低质量周期股、高杠杆公司 |
| 高估值 AI 市场 | 部分参与 + 短债 + call spread；避免全仓追涨 | 满仓高 PE 拥挤交易 |

---

## 20. 高估值 AI 市场 playbook

AI 相关资产基本面可能很强，但价格已经贵。很多资产处于：

> 贵，但可能继续贵。

**重点：高估值资产上涨需要连续兑现高预期；下跌却只需要一个裂缝。现金、短债和 call spread 的价值在于避免坏赔率，同时保留部分上行参与权。**

### 20.1 结构 A：短债 + Call spread

- 90%-95% 放 SGOV/BIL/USFR 类短债；
- 5%-10% 做 QQQ/SMH/SOXX call spread。

适合：长期看多，但怕估值压缩；想保留部分上行；不想满仓承担回撤。

### 20.2 结构 B：核心权益 + 现金储备

- 50%-70% broad ETF / quality equity；
- 20%-40% 短债/现金；
- 5%-10% 高 conviction AI 二阶标的。

适合：想参与市场；不想彻底空仓；希望下跌时有子弹。

### 20.3 结构 C：已有 AI 持仓 + Collar

- 保留已有赢家；
- 买 10%-15% OTM put；
- 卖 15%-25% OTM call。

适合：已经有较大浮盈；想防回撤；愿意放弃一部分极端上涨。

### 20.4 结构 D：Cash-secured put 等目标价

- 只挑你真的想长期持有的标的；
- 只在你愿意买入的价格卖 put；
- 必须准备现金接货。

适合：想低价买入；不想空等；能接受被行权持有底仓。

---

## 21. 按投资目标选择策略

### 21.1 保本和等待

可用：SGOV / BIL / SHV；USFR / TFLO；小比例 call spread 防踏空。

避免：没有明确降息判断时重仓长债；高估值高 beta 资产满仓追涨。

### 21.2 收益增强

可用：JEPI、JEPQ、QYLD、XYLD、RYLD、DIVO、PUTW、SGOV、JPST、MINT。

避免：把高分派当成无风险收益；忽略 NAV erosion 和上涨封顶。

### 21.3 保护已有浮盈

可用：protective put、collar、tail-risk ETF、部分仓位转现金/T-bills。

避免：每个月无脑买很贵的 put；用保护工具把全部上涨空间锁死。

### 21.4 参与上涨但控制风险

可用：call spread、短债 + call spread、defined outcome ETF、部分股票 + 现金储备。

避免：深虚值 call 彩票；把杠杆 ETF 当长期核心仓。

### 21.5 横盘市场赚钱

可用：covered call、put-write、iron condor、短债。

避免：没有催化剂时买入高时间损耗期权。

### 21.6 对冲通胀或宏观 regime shift

可用：GLD、IAU、DBC、PDBC、COMT、DBMF、KMLM、CTA、TIP、SCHP。

避免：通胀上行时重仓长久期债券。

---

# Part VI. 风险管理、决策树与术语表（Risk Management and Glossary）

## 22. 常见错误（Common Mistakes）

1. **把高分派率当成高总回报**。
2. **卖 put 在自己不想长期持有的股票上**。
3. **买深虚值 call，因为它看起来便宜**。
4. **把杠杆 ETF 当长期核心配置**。
5. **在波动率已经爆炸后才买保护**。
6. **忽略税务、分派和产品结构**。
7. **忽略期权 bid-ask spread 和流动性**。
8. **以为 buffer ETF 不管什么时候买都有同样保护**。
9. **策略很复杂，但仓位管理很粗糙**。
10. **交易前没有定义最大亏损**。

---

## 23. 仓位和风险预算经验值

| 策略 | 保守仓位思路 |
|---|---|
| 单只高波动股票 | 单票 2%-5% |
| 行业 ETF | 5%-15%，视 conviction 而定 |
| 买入期权权利金 | 每个 idea 通常 0.5%-3% 组合资金 |
| Call spread 篮子 | 2%-8% 权利金预算 |
| Tail hedge | 1%-5% 年化保险成本预算 |
| Cash-secured put | 只卖你愿意且有现金接盘的规模 |
| 杠杆 ETF | 小仓位战术工具，提前定退出 |
| Covered call ETF | 当权益收益产品，不当债券替代 |

---

## 24. 简单决策树（Decision Tree）

1. **你想保本等待吗？**
   - 现金、T-bills、短债。

2. **你想参与上涨，但最大亏损有限吗？**
   - Call spread 或短债 + call spread。

3. **你已有浮盈股票吗？**
   - Protective put 或 collar。

4. **你想在横盘市场收收入吗？**
   - Covered call 或 put-write，但要理解上涨封顶和下跌风险。

5. **你怕崩盘吗？**
   - Tail-risk hedge、long put、部分转现金。

6. **你担心通胀或宏观冲击吗？**
   - 商品、managed futures、黄金、TIPS。

7. **你想广泛分散吗？**
   - Equity beta + 短债 + managed futures + 黄金 + quality/low-vol factors。

---

## 25. 快速记忆表（Cheat Sheet）

| 操作 | 方向 | 最大亏损 | 主要风险 | 常见用途 |
|---|---|---|---|---|
| 买股票（Long stock） | 看多（Bullish） | 跌到 0 | 资产下跌 | 长期投资 |
| 融资买股票（Margin buying） | 看多（Bullish） | 可能很大 | 强平、利息 | 放大看多 |
| 做空股票（Short selling） | 看空（Bearish） | 理论无限 | 逼空、强平 | 做空/对冲 |
| 买 call（Long call） | 看多（Bullish） | 权利金 | 到期归零、时间损耗 | 杠杆看涨 |
| 买 put（Long put） | 看空/保险（Bearish / hedge） | 权利金 | 到期归零、买贵 | 做空/保护 |
| 卖 call（Short call） | 不看大涨（Neutral to bearish） | 裸卖很大 | 大涨亏损 | 收权利金 |
| Covered call | 温和看多/震荡 | 底仓下跌风险 | 牛市跑输、熊市仍跌 | 收入增强 |
| 卖 put（Short put） | 不看大跌/愿意接盘（Neutral to bullish） | 标的大跌风险 | 高价接盘 | 收权利金/低价买入 |
| Put spread | 温和看多/中性 | 定义亏损 | 跳空下跌 | 定义风险收权利金 |
| Protective put | 防守 | 权利金 | 保险成本高 | 持仓保护 |
| Collar | 区间防守 | 下跌有限保护 | 上涨封顶 | 控制回撤 |
| Iron condor | 横盘 | 定义亏损 | 单边突破 | 区间收益 |
| Long straddle | 大波动 | 权利金 | IV crush、theta | 事件波动 |

---

## 26. 紧凑策略总表（Compact Strategy Table）

| 策略 | 收益曲线 | 最佳场景 | ETF/产品例子 | 主要成本 |
|---|---|---|---|---|
| T-bills / 短债 | 稳定收益 | 不确定 / 高短端利率 | SGOV, BIL, SHV, USFR | 机会成本 |
| 普通指数 ETF | 完整上涨/下跌 | 长期牛市 | SPY, VOO, QQQ | 完整回撤 |
| Quality / Low-vol | 低 beta 权益 | 震荡 / 后周期 | QUAL, USMV, SPLV | 牛市跑输 |
| Covered call | 收入，上涨封顶 | 横盘 / 慢牛 | JEPI, JEPQ, QYLD, XYLD | 放弃上涨 |
| Put-write | 收权利金，承担下跌 | 横盘 / 慢牛 | PUTW | 暴跌风险 |
| Protective put | 下跌地板 | 危机风险 | TAIL-style hedge, direct puts | 保险费 |
| Collar | 下有保护，上有封顶 | 保护浮盈 | NUSI/QRMI/XRMI 风格 | 上涨封顶 |
| Call spread | 定义风险上涨 | 温和牛市 | Direct options | 上涨封顶，权利金损失 |
| Buffer ETF | buffer + cap | 不确定但想持股 | Innovator, FT Vest series | cap、买入时点敏感 |
| Managed futures | 趋势分散 | 危机/通胀趋势 | DBMF, KMLM, CTA | whipsaw |
| Gold | 危机/实际利率对冲 | 实际利率下行 | GLD, IAU | 无现金流 |
| Commodities | 通胀/趋势对冲 | 通胀冲击 | DBC, PDBC, COMT | 展期/周期风险 |
| Leveraged ETF | 战术放大 beta | 强趋势 | TQQQ, SOXL, SQQQ | 路径损耗 |
| Merger arb | 事件 spread | 并购活跃 | MNA | 交易失败 |
| Market neutral | 低 beta 另类 | 贵/震荡权益市场 | BTAL, QAI | 牛市跑输 |

---

## 27. 核心术语速查（Glossary）

| 中文术语 | English term | 简要说明 |
|---|---|---|
| 做多 | Long | 买入资产，希望价格上涨 |
| 做空 | Short | 借入并卖出资产，希望未来低价买回 |
| 看涨 | Bullish | 预期价格上涨 |
| 看跌 | Bearish | 预期价格下跌 |
| 横盘 | Range-bound / Sideways | 价格在区间内震荡 |
| 波动率 | Volatility | 价格波动幅度 |
| 隐含波动率 | Implied volatility / IV | 期权价格反推的未来波动率预期 |
| 已实现波动率 | Realized volatility | 事后观察到的实际波动率 |
| 期权 | Option | 买方拥有权利、卖方承担义务的合约 |
| 看涨期权 | Call option | 买方有权按行权价买入标的 |
| 看跌期权 | Put option | 买方有权按行权价卖出标的 |
| 标的资产 | Underlying asset | 期权对应的股票、ETF、指数、期货等 |
| 行权价 | Strike price | 期权约定买入或卖出的价格 |
| 到期日 | Expiration date | 期权有效期结束日 |
| 权利金 | Premium | 买方支付、卖方收取的期权价格 |
| 行权 | Exercise | 买方使用期权权利 |
| 实值 | In the money / ITM | 期权已有内在价值 |
| 平值 | At the money / ATM | 标的价格接近行权价 |
| 虚值 | Out of the money / OTM | 期权暂无内在价值 |
| 内在价值 | Intrinsic value | 期权立即行权可获得的价值 |
| 时间价值 | Time value | 期权因剩余时间和不确定性产生的价值 |
| 时间损耗 | Time decay / Theta decay | 期权随时间流逝而损失价值 |
| Delta | Delta | 标的价格变化对期权价格的影响 |
| Gamma | Gamma | Delta 的变化速度 |
| Theta | Theta | 时间流逝对期权价格的影响 |
| Vega | Vega | 隐含波动率变化对期权价格的影响 |
| 保证金 | Margin | 借钱/承担义务交易时需要的履约保证 |
| 维持保证金 | Maintenance margin | 持仓期间必须维持的最低保证金要求 |
| 追加保证金 | Margin call | 保证金不足时券商要求补充资金 |
| 强制平仓 | Forced liquidation | 保证金不足时被券商强制平仓 |
| 融资买入 | Margin buying | 借钱买入资产做多 |
| 融券做空 | Short selling | 借入证券卖出做空 |
| 收益曲线 | Payoff profile | 不同价格结果下的盈亏形状 |
| 盈亏平衡点 | Break-even point | 策略到期不赚不亏的价格 |
| 最大亏损 | Maximum loss | 策略最坏情况下亏损上限 |
| 最大收益 | Maximum profit | 策略最好情况下收益上限 |
| 看涨价差 | Call spread | 买低行权价 call、卖高行权价 call |
| 看跌价差 | Put spread | 买卖不同行权价 put 构造定义风险 |
| 保护性看跌 | Protective put | 持有底仓同时买 put 防跌 |
| 领口策略 | Collar | 持有底仓 + 买 put + 卖 call |
| 备兑看涨 | Covered call | 持有底仓同时卖 call 收权利金 |
| 现金担保卖 put | Cash-secured put | 持有现金并卖 put，准备被行权接盘 |
| 跨式 | Straddle | 同行权价同时买 call 和 put |
| 宽跨式 | Strangle | 买 OTM call 和 OTM put |
| 铁鹰策略 | Iron condor | 同时卖 put spread 和 call spread，押区间震荡 |
| 日历价差 | Calendar spread | 卖短期期权、买长期期权 |
| 尾部风险 | Tail risk | 小概率但大幅亏损的极端风险 |
| 尾部对冲 | Tail hedge | 用 put、尾部风险 ETF 等对冲极端下跌 |
| 路径依赖 | Path dependency | 最终收益取决于价格运动路径而不只是终点 |
| 波动率损耗 | Volatility decay | 杠杆 ETF 或期权策略在震荡中因路径损耗亏损 |
| 久期 | Duration | 债券价格对利率变化的敏感度 |
| 凸性 | Convexity | 价格对利率或标的变动的非线性敏感度 |
| 信用利差 | Credit spread | 信用债相对无风险利率的额外收益率 |
| 商品展期收益 | Roll yield | 期货换月时产生的收益或损耗 |
| 系统性卖权利金 | Systematic option writing | 规则化卖 call 或 put 收取权利金 |
| 定义结果 ETF | Defined outcome ETF | 用期权包预设 buffer 和 cap 的 ETF |
| 缓冲 | Buffer | 在一定跌幅内提供下跌吸收 |
| 上涨封顶 | Cap | 策略最大上涨收益被限制 |
| 管理期货 | Managed futures | 跨资产期货趋势跟随策略 |
| 市场中性 | Market neutral | 降低市场 beta，追求相对价值收益 |
| 风险平价 | Risk parity | 按风险而非金额分配多资产仓位 |
| 并购套利 | Merger arbitrage | 捕捉并购交易价和现价之间的 spread |
| 因子 | Factor | 质量、价值、动量、低波等系统性风格暴露 |
| 贝塔 | Beta | 对整体市场波动的敏感度 |
| 阿尔法 | Alpha | 扣除市场和风险因子后的超额收益 |

---

## 28. 最终原则（Final Framework）

大多数实用金融策略可以归为五类：

1. **安全等待**：现金、T-bills、短债。
2. **持有 beta**：宽基 ETF、行业 ETF、因子 ETF。
3. **定义风险**：put、collar、call spread、buffer ETF。
4. **赚取权利金**：covered call、put-write、iron condor。
5. **分散 regime**：managed futures、黄金、商品、市场中性、risk parity。

最重要的不是找到最复杂的结构，而是匹配：

- 市场环境；
- 你的观点；
- 你的最大可承受亏损；
- 你的持有周期；
- 你的仓位大小。

最后一句：

> 默认使用简单工具；只有当期权或结构化产品能明确解决“现金、债券、普通 ETF 无法解决的问题”时，才值得使用复杂工具。

**重点：复杂策略的目标不是显得高级，而是把最大亏损、上涨空间、现金流、波动风险和持有周期变得更可控。看不懂最大亏损和失败模式的结构，默认不做。**
