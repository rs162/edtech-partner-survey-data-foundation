# 📊 Partner Survey Data Foundation

**Turning messy, inconsistent survey exports from multiple partners into one clean, analysis-ready data foundation — without throwing away anyone's custom fields.**

When an organization collects the *same* survey from several partner institutions, every partner sends their data a little differently: different column names, different formats, an extra custom question here, a duplicate submission there. This project builds the small **data warehouse layer** that makes all of it comparable — so leadership can answer *"how are our partners actually doing?"* with confidence.

> A **data-foundation & metrics design** portfolio project. It's intentionally a clear, reproducible pipeline rather than a heavy production system — the goal is to show *how* I model and reason about data, end to end. *(White-labeled: domain framing is a generic ed-tech partner program; the dataset is synthetic.)*

---

## 🧭 The 30-second version

| | |
|---|---|
| **Problem** | Two partners (A & B) send the same student survey in different shapes, with messy real-world issues. |
| **Goal** | One unified data model + trustworthy partner-level metrics, without losing partner-specific fields. |
| **Approach** | A star-schema-style foundation: dimensions, fact tables, and a flexible long table for custom questions. |
| **Output** | Clean tables + three defined metrics (confidence change, belonging change, lesson-completion rate). |
| **Skills shown** | Data modeling, schema normalization, data-quality validation, metric definition, clear documentation. |

---

## 🎯 What this project demonstrates

- **🏛️ Data modeling** — designing a [star-schema](https://en.wikipedia.org/wiki/Star_schema) foundation (dimension + fact tables) that's the right shape for analytics, with a clear reason for every table.
- **🔗 Schema normalization** — mapping each partner's idiosyncratic columns onto one shared vocabulary, so cross-partner comparison is even *possible*.
- **🧩 Not losing data** — partner-specific questions (e.g. Partner B's custom "peer connection" field) are **preserved** in a flexible long table instead of being dropped to force everyone into the same shape.
- **🔍 Data-quality discipline** — explicit validation checks that surface real problems (out-of-range survey scores, duplicate submissions, "orphan" records with no matching student) instead of silently averaging over them.
- **📐 Metric definition** — every metric is defined with an explicit numerator, denominator, and scope, so the numbers mean exactly what they say.
- **📖 Honest documentation** — assumptions, tradeoffs, and "what I'd do with more time" are written down, not hidden.

---

## 🏗️ The data model

The design separates *who/what* (dimensions) from *what happened* (facts), with a flexible layer for custom survey questions:

| Table | One row per… | Why it exists |
|---|---|---|
| `dim_partner` | partner | Canonical partner lookup — kills inconsistent naming. |
| `dim_student` | student | Demographics/enrollment used to slice metrics. |
| `fact_survey_submission` | student × survey wave | Normalized pre/post survey facts (shared `confidence`, `belonging`). |
| `survey_question_response` | submission × question | **Preserves every answer**, including partner-specific custom questions. |
| `fact_engagement_event` | telemetry event | Activity log (lesson completes, dialogue joins, …). |
| `event_metadata` | event metadata key/value | Scalable home for JSON fields like score/duration. |

**The key design move:** standard questions get mapped into shared fields for comparison, while custom questions are tagged `partner_specific` and kept in the long `survey_question_response` table. New partners or new questions slot in by adding a small mapping — the model doesn't have to change.

---

## 🔍 Real-world messiness it handles

The provided dataset has the kinds of problems real survey data always has — and the pipeline catches them explicitly:

- ⚠️ **Out-of-range values** — survey scores outside the valid 1–5 range are flagged by validation checks.
- 👥 **Duplicate submissions** — 28 duplicate student/partner/wave rows; resolved by a documented rule (keep the latest submission).
- 👻 **Orphan records** — 3 survey rows with no matching student profile; excluded from reporting but surfaced as a data-quality issue rather than hidden.

---

## 📐 The metrics

Three partner-level metrics, each with an explicit definition:

1. **Matched confidence change** — average (post − pre) confidence among students who completed *both* surveys.
2. **Matched belonging change** — same, for sense of belonging.
3. **Lesson-completion rate** — share of *enrolled* students with at least one completed lesson (denominator = all enrolled, so low engagement stays visible).

The notebook reports each as numerator / denominator / value so the meaning is unambiguous.

---

## 🚀 Run it

```bash
git clone https://github.com/rs162/partner-survey-data-foundation.git
cd partner-survey-data-foundation

python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

jupyter notebook survey_data_foundation.ipynb   # or open in VS Code / Colab
```

The provided synthetic dataset lives in [`data/`](data/), so the notebook runs top-to-bottom with no setup.

---

## 🗂️ What's in here

```
partner-survey-data-foundation/
├── survey_data_foundation.ipynb   ← the full pipeline, documented cell by cell
├── data/                          ← provided synthetic survey + engagement data
│   ├── student_profiles.csv
│   ├── survey_partner_a.csv
│   ├── survey_partner_b.csv
│   └── interaction_events.csv
├── requirements.txt
└── README.md
```

---

<sub>Data-engineering portfolio project by Ratna Suthar. White-labeled: presented as a generic ed-tech partner program. The dataset is synthetic. The notebook includes an AI-usage log documenting where assistant tools were used.</sub>
