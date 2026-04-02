<div align="center">

# 📊 quant-trading — 量化交易助手

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://python.org)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

<br>

还在手动翻 K 线图找买卖点？<br>
还在 Excel 里算估值分位算到眼花？<br>
选了一堆股票不知道该买哪只？<br>
分析完的数据转头就忘，下次还得从头来？<br>

**一个指令搞定「分析→决策→归档」，量化交易全自动闭环！**

<br>

触发指令：`量化交易` 或 `/quant`<br>
自动调度两个量化 Skill，5 维度加权打分，输出明确的 **买入 / 观望 / 规避** 结论

[前置依赖](#前置依赖) · [安装](#安装) · [使用](#使用) · [执行规则](#执行规则) · [效果示例](#效果示例) · [决策权重](#决策权重) · [项目结构](#项目结构)

</div>

---

## 前置依赖

本 Skill **不包含独立的量化引擎**，核心能力完全依赖以下两个 Skill，三者必须安装在同一 skills 目录下：

| 依赖 Skill | 用途 | 仓库 |
|-----------|------|------|
| **cn-stock-quant** | 个股全面量化分析 — 估值分位、FCF 分红模型、9因子信号、风险指标、动量、资金面、事件日历、交易回测 | [cn-stock-quant](https://github.com/CroTuyuzhe/cn-stock-quant) |
| **quant-stock-screener** | 量化多因子选股 — 6大策略组合（低估值/成长/质量/动量/低波动/情绪），覆盖 A 股 + 港股 | [quant-stock-screener](https://github.com/CroTuyuzhe/quant-stock-screener) |

> ⚠️ **必须同时安装以上两个 Skill，否则本 Skill 无法运行。**

---

## 安装

### Claude Code

> **重要**：Claude Code 从 **git 仓库根目录** 的 `.claude/skills/` 查找 skill。请在正确的位置执行。

```bash
# 安装到当前项目（在 git 仓库根目录执行）
mkdir -p .claude/skills

git clone https://github.com/CroTuyuzhe/quant-trading-normal.git .claude/skills/quant-trading
git clone https://github.com/CroTuyuzhe/cn-stock-quant.git .claude/skills/cn-stock-quant
git clone https://github.com/CroTuyuzhe/quant-stock-screener.git .claude/skills/quant-stock-screener

# 或安装到全局（所有项目都能用）
git clone https://github.com/CroTuyuzhe/quant-trading-normal.git ~/.claude/skills/quant-trading
git clone https://github.com/CroTuyuzhe/cn-stock-quant.git ~/.claude/skills/cn-stock-quant
git clone https://github.com/CroTuyuzhe/quant-stock-screener.git ~/.claude/skills/quant-stock-screener
```

### OpenClaw

```bash
git clone https://github.com/CroTuyuzhe/quant-trading-normal.git ~/.openclaw/workspace/skills/quant-trading
git clone https://github.com/CroTuyuzhe/cn-stock-quant.git ~/.openclaw/workspace/skills/cn-stock-quant
git clone https://github.com/CroTuyuzhe/quant-stock-screener.git ~/.openclaw/workspace/skills/quant-stock-screener
```

### 依赖

```bash
pip install akshare pandas numpy scipy
```

---

## 使用

在 Claude Code 中输入：

```
/quant
```

或直接说：

```
量化交易
```

按提示输入个股代码和执行步数，即可开始分析。

### 管理命令

| 命令 | 说明 |
|------|------|
| `量化交易` / `/quant` | 启动量化交易助手 |
| `查看量化记录` / `量化历史` | 查询过往分析记录 |

---

## 效果示例

### 个股分析（5步）

```
用户      ❯ /quant

助手      ❯ 📊 量化交易助手已启动
           请输入要分析的个股代码（留空将自动执行全市场选股）：

用户      ❯ 600104

助手      ❯ 请输入量化执行步数（如 15；留空则单次分析）：

用户      ❯ 5

助手      ❯ 开始 600104 量化分析 x5...
           20:12 — 观望
           20:13 — 观望
           20:14 — 观望
           20:15 — 观望
           20:16 — 观望

           📋 量化交易报告 — 2026-04-02 20:16

           一、执行概要
              股票：600104 上汽集团
              步数：5
              最终结论：⏸️ 观望（5/5 一致）

           二、综合评分
              估值 0.55 × 25% + 因子 0.55 × 30% + 风险 0.25 × 20%
              + 动量 0.45 × 15% + 资金 0.65 × 10% = 0.419 → 观望

           三、每步运行汇总
              20:12 — 观望
              20:13 — 观望
              20:14 — 观望
              20:15 — 观望
              20:16 — 观望

           四、风险提示
              以上分析仅供参考，不构成投资建议。
```

### 全市场选股

```
用户      ❯ /quant

助手      ❯ 请输入要分析的个股代码（留空将自动执行全市场选股）：

用户      ❯ [留空]

助手      ❯ 执行全市场选股...
           [选股结果 + Top 5 逐只深度分析 + 独立交易结论]
```

---

## 执行规则

### 核心约束

| 规则 | 说明 |
|------|------|
| **单流程执行** | 每次触发只执行一轮（引导→输入→分析→报告），不自动重复 |
| **步数上限** | 最多 20 步，超出自动截断 |
| **步间间隔** | 多步执行时，每步间等待约 **60 秒**（非最后一步返回极简结论后等待） |
| **超时兜底** | 单只分析 120s，选股 300s |
| **错误处理** | 调用失败记录错误并给出部分结论，不无限重试 |
| **不追问** | 输出报告即结束，等待用户下次触发 |

### 多步执行流程

```
非最后一步 → 返回极简一行: "HH:mm — 结论"
           → 等待 ~60 秒
           → 执行下一步

最后一步   → 返回完整量化分析报告
           → 附所有步的运行汇总表
```

---

## 决策权重

交易结论基于 5 维度加权打分：

| 维度 | 权重 | 买入阈值 |
|------|------|----------|
| 估值分位 | 25% | PE/PB 分位 < 30% |
| 因子信号 | 30% | 9因子综合评分 > 1.5 |
| 风险指标 | 20% | 夏普 > 1 且最大回撤 < 15% |
| 动量信号 | 15% | 动量分组回测收益 > 0 |
| 资金面 | 10% | OBV 趋势向上且筹码集中 |

**综合阈值：**

| 加权得分 | 结论 |
|----------|------|
| > 0.6 | ✅ 建议买入 |
| 0.3 ~ 0.6 | ⏸️ 建议观望 |
| < 0.3 | ⛔ 建议规避 |

---

## 记录与归档

每次执行完成后，分析结果自动写入记录文件：

```
references/records/YYYY-MM-DD.json
```

记录内容：时间戳、股票代码、执行步数、因子数据、加权得分、最终结论。

可通过 `查看量化记录` 或 `量化历史` 查询过往分析。

---

## 项目结构

本项目遵循 [AgentSkills](https://agentskills.io) 开放标准，整个 repo 就是一个 skill 目录：

```
quant-trading/
├── SKILL.md                          # Skill 入口（官方 frontmatter）
├── README.md                         # 本文件
├── LICENSE
└── references/
    └── records/                      # 分析记录（按日期归档）
        ├── 2026-04-02.json
        └── ...
```

---

## 注意事项

- **必须同时安装** cn-stock-quant 和 quant-stock-screener，否则本 Skill 无法运行
- 数据基于 akshare + 腾讯财经 API，A 股覆盖 5000+ 只，港股覆盖恒指成分股
- 财务数据仅在年报/半年报/季报披露后更新
- 结论仅供研究参考，不构成投资建议

---

<div align="center">

MIT License © [CroTuyuzhe](https://github.com/CroTuyuzhe)

</div>
