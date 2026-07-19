# Dunder Mifflin RevOps Case Study

[![CI](https://github.com/anacartola/dunder-mifflin-revops/actions/workflows/ci.yml/badge.svg)](https://github.com/anacartola/dunder-mifflin-revops/actions/workflows/ci.yml) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**English** | [Português](README.pt-BR.md) | [Français](README.fr.md) | [简体中文](README.zh-CN.md)

A self-contained RevOps analytics exercise: a synthetic B2B SaaS-style CRM dataset,
three open-ended questions from a CRO, and a worked solution.

Built to be **downloaded and attempted**, not just read. Try it first, then compare.

---

## The scenario

Dunder Mifflin Paper Company is rebuilding how its sales organisation forecasts.
You have just joined as a RevOps data analyst. The CRO, Michael Scott, asks three
questions in a Monday meeting:

1. *We missed Q1 bookings by 18%. What blew the forecast?*
2. *Where are deals stalling, and why?*
3. *Given today's pipe, are we on track for next quarter?*

The brief is deliberately ambiguous. Part of the exercise is deciding what the
questions actually mean, then saying so out loud. Every deliverable should state
its assumptions and caveats explicitly.

> One of the three questions cannot be answered as asked. Finding out which one,
> and being able to prove it, is the point.

**Suggested cap: 90 minutes** for a first pass.

---

## The data

Four tables in `data/`, ~700 deals across 2024 and the first half of 2025.

| File | Rows | Grain |
|---|---|---|
| `deals_meta.csv` | 700 | one row per deal, current state |
| `deals_snapshot.csv` | 8,321 | one row per deal per week, pipeline history |
| `owners.csv` | 13 | sales reps, segments, managers, tenure |
| `targets.csv` | 104 | quarterly booking target per rep |

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

Stages: `Prospecting → Qualification → Proposal → Negotiation → Closed Won / Closed Lost`

The data has the rough edges you would expect from a real CRM export. They are
not bugs, they are the exercise.

---

## Findings

*The exercise is more fun attempted first — full working in [`notebooks/revops_analysis.ipynb`](notebooks/revops_analysis.ipynb).*

<details>
<summary><b>Executive summary — click to expand</b></summary>

<br>

**Method.** Standard RevOps metrics (win rate, weighted pipeline with win-rate-calibrated category weights, coverage, stage velocity) plus a rigor layer: Wilson confidence intervals, a chi-square test of independence (with a permutation check), per-stage control limits (SPC) for the stall threshold, skew-aware summaries, and a Pareto — framed with DMAIC.

**Baseline.** 700 deals, 109 closed won. Win rate **48.7%** (95% CI 42–55%). The typical deal is the **median ~$58.5k**, not the mean ~$70k (deal value is right-skewed, skew 2.6); sales cycle ~90 days. Win rate looks highest on Cross-Sell (59%) and lowest on New Client (43%) — but the confidence intervals overlap, so the ranking is *suggestive, not established*.

**Q1 — "We missed Q1 by 18%. What blew the forecast?" → Can't be answered as asked.**
The `targets` table is corrupted: raw quarterly targets inflate exponentially, reaching ~**93,000×** the 2024-Q1 baseline by 2025-Q4. With no trustworthy target, the "18% miss" can't be verified; a reconstruction from the one calibrated quarter shows a ~**9%** gap. This is the question the brief can't support.

**Q2 — "Where are deals stalling, and why?" → A process problem, not people or channels — tested, not asserted.**
115 of 445 open deals are stalled. A chi-square test of independence finds **no significant association** between stalling and manager, lead source, account size, or segment (p = 0.30–0.90; expected-count assumptions met; a permutation test agrees) — evidence of a broken *process*, not a weak rep or bad channel. (Deal type is borderline, p ≈ 0.08: New Client may stall slightly more.) Stalling is stage-dependent too: median dwell runs from **1 week in Negotiation to 5 in Prospecting**, so a flat "4 weeks" mislabels deals — a per-stage control limit is the fix. For where to act, a Pareto shows **4 of 8 managers hold ~80% of the stalled value**.

**Q3 — "Given today's pipe, are we on track for next quarter?" → Bookings look fine; the pipeline signal can't be trusted in this export.**
Q2 2025 bookings are ahead of pace (projecting ~**$1.70M** vs the **reconstructed** **$1.17M** target (see Q1), **+46%**). The open pipeline *appears* to fall **63%** in the final four weeks ($8.46M → $3.12M) — but that coincides exactly with the export window closing: snapshot coverage drops from ~130 to **95 deals/week** and **no new deals enter in the last ~3 weeks**. The "collapse" is largely an **artifact of the truncated data**, not a proven decline. Honest verdict: bookings are on pace; the leading indicator is unreadable near the cutoff — a second signal the data can't fully support.

</details>

---

## Running it

The project pins its Python version (`.python-version`) and its dependencies
(`pyproject.toml` / `poetry.lock`), so a fresh clone runs the same on any machine.

**With [pyenv](https://github.com/pyenv/pyenv) + [Poetry](https://python-poetry.org/) (recommended):**

```bash
git clone <this-repo>
cd dunder-mifflin-revops

pyenv install 3.12        # skip if 3.12 is already installed; obeys .python-version
poetry env use 3.12       # create the venv on the pinned interpreter
poetry install            # install locked dependencies (incl. JupyterLab)
poetry run jupyter lab notebooks/revops_analysis.ipynb
```

On Windows, the pyenv commands are the same under [pyenv-win](https://github.com/pyenv-win/pyenv-win).

**Without Poetry (plain pip):** create a virtualenv on Python 3.12 and
`pip install -r requirements.txt`, then launch JupyterLab.

No cloud account, no auth, no API keys. Everything reads from `data/`.
Derived outputs are written to `output/` and are gitignored.

Every push runs the notebook end-to-end on a clean CI runner (the **CI** badge
above), so "downloads and runs anywhere" is checked automatically, not just claimed.

---

## Repo layout

```
data/                    four CSVs, the input
notebooks/
  revops_analysis.ipynb  worked solution
output/                  generated tables, gitignored
```

The notebook follows Ask → Prepare → Process → Analyse → Share. The Process
section is where the assumptions live, and it is the section worth arguing with.

---

## Data provenance and licence

The dataset is **synthetic**. No real company, customer, deal, or person appears
in it. Names are characters from *The Office* (US), used as recognisable labels
so that a rep-level finding is easier to talk about than "owner_9". Email
addresses use the `.invalid` TLD reserved by RFC 2606 and can never resolve to a
real mailbox.

*The Office*, Dunder Mifflin, and the character names are the property of their
respective rights holders and are used here only as labels in a free,
non-commercial teaching dataset. No affiliation or endorsement is implied.

The **code** in this repository is licensed under the [MIT License](LICENSE). The
**dataset and written content** are licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — free to
share and adapt with attribution.
