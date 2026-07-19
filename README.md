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

The dataset, notebook, and text in this repository are released for free
educational use. The code is licensed under the [MIT License](LICENSE).
