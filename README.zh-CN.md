# Dunder Mifflin RevOps 案例研究

[![CI](https://github.com/anacartola/dunder-mifflin-revops/actions/workflows/ci.yml/badge.svg)](https://github.com/anacartola/dunder-mifflin-revops/actions/workflows/ci.yml) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

[English](README.md) | [Português](README.pt-BR.md) | [Français](README.fr.md) | **简体中文**

一个自包含的 RevOps 分析练习：一份合成的 B2B SaaS 风格 CRM 数据集、来自 CRO 的三个开放式问题，
以及一份参考解答。

它被设计为**下载并动手尝试**，而不仅仅是阅读。先自己试一遍，再对照参考解答。

---

## 场景

Dunder Mifflin Paper Company 正在重新构建其销售组织的预测方式。你刚以 RevOps 数据分析师的身份加入。
CRO Michael Scott 在周一的会议上提出三个问题：

1. *我们第一季度的签约额（bookings）少了 18%。是什么让预测失准了？*
2. *交易（deals）在哪里停滞，为什么？*
3. *以当前的管道（pipe）来看，我们下个季度能达标吗？*

这份需求是有意含糊的。练习的一部分就是判断这些问题究竟指的是什么，然后明确地说出来。每一份交付物
都应显式地说明其假设与注意事项。

> 三个问题中有一个无法按其原样回答。找出是哪一个，并能够证明它，正是本练习的重点。

**建议时限：90 分钟**完成第一遍。

---

## 数据

`data/` 中有四张表，涵盖 2024 年及 2025 年上半年约 700 笔交易。

| 文件 | 行数 | 粒度 |
|---|---|---|
| `deals_meta.csv` | 700 | 每笔交易一行，当前状态 |
| `deals_snapshot.csv` | 8,321 | 每笔交易每周一行，管道历史 |
| `owners.csv` | 13 | 销售代表、细分、经理、任职时长 |
| `targets.csv` | 104 | 每位销售代表的季度签约目标 |

```
deals_meta      deal_id, owner_id, created_date, close_date, deal_stage,
                forecast_category, record_type, deal_type, deal_order_type,
                account_industry, account_region, account_size, deal_source,
                deal_amount

deals_snapshot  deal_id, snapshot_date, stage, forecast_category, amount,
                close_date, owner_id

owners          owner_id, name, email, role_start_date, role_end_date, role,
                manager, segment

targets         quarter, owner_id, segment, target_amount
```

阶段：`Prospecting → Qualification → Proposal → Negotiation → Closed Won / Closed Lost`

这些数据带有你能从真实 CRM 导出中预期到的粗糙之处。它们不是缺陷，而正是练习本身。

---

## Findings（结论）

*剧透提示 —— 这个练习自己先做一遍更有意思。完整过程见 [`notebooks/revops_analysis.ipynb`](notebooks/revops_analysis.ipynb)；这里是执行摘要。*

**方法。** 标准 RevOps 指标（win rate、按历史校准各类别权重的加权管道、覆盖率、阶段速度），外加一层统计严谨性：Wilson 置信区间、卡方独立性检验（含置换检验复核）、用于停滞阈值的分阶段控制限（SPC）、考虑偏度的汇总统计，以及一张帕累托图 —— 以 DMAIC 框架组织。

**基线。** 700 笔交易，109 笔成交。Win rate **48.7%**（95% 置信区间 42–55%）。典型交易规模是**中位数约 $58.5k**，而非均值约 $70k（交易金额右偏，偏度 2.6）；销售周期约 90 天。Win rate 看起来在 Cross-Sell（59%）最高、New Client（43%）最低 —— 但置信区间相互重叠，因此该排序只是*提示性的，而非已确立的*。

**Q1 —— “第一季度差了 18%，是什么让预测失准？” → 无法按原样回答。**
`targets` 表已损坏：原始季度目标呈指数级膨胀，到 2025-Q4 达到 2024-Q1 基线的约 **93,000 倍**。没有可信的目标，“18% 的差距”就无法验证；从唯一校准季度重建后显示约 **9%** 的差距。这就是需求无法支撑的那个问题。

**Q2 —— “交易在哪里停滞，为什么？” → 是流程问题，而非人或渠道问题 —— 经过检验，而非断言。**
445 笔在进行的交易中有 115 笔停滞。卡方独立性检验**未发现**停滞与经理、渠道来源、客户规模或细分之间存在显著关联（p = 0.30–0.90；期望频数假设满足；置换检验一致）—— 这是*流程*失灵的证据，而非某个销售弱或渠道差。（交易类型处于临界，p ≈ 0.08：New Client 可能略微更容易停滞。）停滞还与阶段有关：停留时间中位数从 **Negotiation 的 1 周到 Prospecting 的 5 周**不等，因此统一的“4 周”会误判交易 —— 分阶段控制限才是正解。至于从哪里入手，帕累托图显示 **8 位经理中的 4 位掌握了约 80% 的停滞金额**。

**Q3 —— “以当前管道来看，下个季度能达标吗？” → 签约说能，管道说不能。**
2025 年 Q2 的签约进度领先（预计 **$1.70M**，目标 **$1.17M**，**+46%**）—— 但那是滞后指标。领先指标已亮红灯：在建管道四周内下跌 **63%**（$8.46M → $3.12M），比 Q1 同期低 57%。Q3 覆盖率为 **1.3×**（健康值为 3×），加权预测为目标的 **49%**。Q3 将继承一个被掏空的管道，需要立即重建。

---

## 运行方式

项目锁定了 Python 版本（`.python-version`）及其依赖（`pyproject.toml` / `poetry.lock`），
因此全新克隆在任何机器上都能一致运行。

**使用 [pyenv](https://github.com/pyenv/pyenv) + [Poetry](https://python-poetry.org/)（推荐）：**

```bash
git clone <this-repo>
cd dunder-mifflin-revops

pyenv install 3.12        # 如果已安装 3.12 可跳过；遵循 .python-version
poetry env use 3.12       # 在锁定的解释器上创建 venv
poetry install            # 安装锁定的依赖（含 JupyterLab）
poetry run jupyter lab notebooks/revops_analysis.ipynb
```

在 Windows 上，使用 [pyenv-win](https://github.com/pyenv-win/pyenv-win) 时 pyenv 命令相同。

**不使用 Poetry（纯 pip）：** 在 Python 3.12 上创建虚拟环境并运行
`pip install -r requirements.txt`，然后启动 JupyterLab。

无需云账户、无需认证、无需 API 密钥。一切都从 `data/` 读取。生成的输出写入 `output/`，
并已被 git 忽略。

---

## 仓库结构

```
data/                    四个 CSV，输入数据
notebooks/
  revops_analysis.ipynb  参考解答
output/                  生成的表格，已被 git 忽略
```

Notebook 遵循 Ask → Prepare → Process → Analyse → Share（提问 → 准备 → 处理 → 分析 → 分享）。
Process 部分是各项假设所在之处，也是最值得推敲的部分。

---

## 数据来源与许可

该数据集为**合成数据**。其中不含任何真实的公司、客户、交易或个人。名称取自《办公室》
（*The Office*，美版）中的角色，用作易于辨识的标签，这样按销售代表得出的结论比 "owner_9"
更便于讨论。电子邮件地址使用 RFC 2606 保留的 `.invalid` 顶级域，永远无法解析到真实邮箱。

《The Office》、Dunder Mifflin 以及角色名称均归其各自权利人所有，此处仅用作一个免费、
非商业教学数据集中的标签。不暗示任何隶属或背书关系。

本仓库中的数据集、notebook 及文本以免费教育用途发布。代码依据 [MIT 许可证](LICENSE) 发布。
