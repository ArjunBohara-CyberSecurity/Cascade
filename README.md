<div align="center">

![Cascade](https://img.shields.io/badge/CASCADE-000000?style=for-the-badge&logo=cascade&logoColor=white)
![Churn Intelligence](https://img.shields.io/badge/CHURN%20INTELLIGENCE%20SYSTEM-e11d48?style=for-the-badge)

![ML](https://img.shields.io/badge/Machine%20Learning-Pipeline-blue?style=flat-square)
![LLM](https://img.shields.io/badge/LLM-Reasoning%20Agent-brightgreen?style=flat-square)
![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python&logoColor=white)
![Stages](https://img.shields.io/badge/Stages-3-lightgrey?style=flat-square)

![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)

</div>

---

## 🌊 Cascade

> Two-stage churn intelligence — statistical risk scoring cascades into LLM-driven reasoning and retention action.

Cascade predicts **which accounts are about to churn**, **explains why** using real feature attribution (not vibes), and **recommends a grounded next action** by retrieving from a playbook knowledge base. It's built to answer the question every stakeholder actually asks: *"why did the model flag this account, and what do we do about it?"*

Every score, explanation, and recommendation is logged end-to-end — nothing is a black box.

---

## 🧠 Core Capabilities

- ⚡ Two-stage architecture — statistical grounding, then LLM reasoning
- 🎯 Calibrated risk scores (PR-AUC + Platt/isotonic calibration)
- 🔍 Per-account SHAP explainability
- 🤖 LLM agent constrained to structured, schema-enforced output
- 📚 RAG-based retention playbook retrieval — no hardcoded action logic
- 🧪 Hand-labeled hallucination-rate eval for the LLM stage
- 🗂️ Full audit trail — every stage logged to Postgres
- 📊 React dashboard with drill-down reasoning chain

---

## 🏗️ Tech Stack

| Layer | Technology |
|---|---|
| 🗄️ **Database** | PostgreSQL + pgvector |
| 🐍 **Backend** | Python 3.11+, FastAPI |
| 📈 **Feature Engineering** | pandas / polars, YAML feature registry |
| 🌲 **Risk Model** | scikit-learn (baseline), LightGBM (main) |
| 🎛️ **Tuning** | Optuna |
| 🔎 **Explainability** | SHAP |
| 🎯 **Calibration** | Platt scaling / isotonic regression |
| 🧠 **LLM Reasoning** | Anthropic / OpenAI API + Pydantic schema enforcement |
| 📚 **RAG Layer** | sentence-transformers + pgvector |
| 🖥️ **Frontend** | React + TypeScript, Vite, Tailwind, Recharts, TanStack Query |
| 🐳 **Deployment** | Docker Compose, Cloud Run / Azure Container Apps |
| 📋 **Experiment Tracking** | MLflow *(optional)* |

---

## 🔬 Coverage Areas

| Stage | Description |
|---|---|
| 📊 **Stage 1 — Risk Scoring** | LightGBM model, time-based split, calibrated probabilities |
| 🧩 **Stage 2 — Reasoning** | LLM interprets SHAP output into a structured risk narrative |
| 📖 **Stage 3 — Action** | RAG retrieval over retention playbooks, personalized per account |
| 🗃️ **Audit Layer** | Full pipeline output logged to Postgres for traceability |

---

## ⚙️ Installation

```bash
git clone https://github.com/ArjunBohara-CyberSecurity/Cascade.git
cd Cascade

python -m venv venv

# Windows
venv\scripts\activate

# Linux / MacOS
source venv/bin/activate

pip install -r requirements.txt
```

Set up environment variables:

```bash
cp .env.example .env
# Add DATABASE_URL, LLM_API_KEY, etc.
```

Spin up Postgres + services:

```bash
docker compose up -d
```

---

## 🚀 Launch

Run the pipeline API:

```bash
uvicorn app.main:app --reload
```

Run the frontend:

```bash
cd frontend
npm install
npm run dev
```

---

## 🧪 Usage

1. Load account data into Postgres (`accounts`, `usage_events`, `support_tickets`, `billing_events`)
2. Run the feature engineering module to build the feature set
3. Train / load Stage 1 risk model → generates calibrated scores + SHAP values
4. Stage 2 LLM agent converts SHAP output into a structured risk narrative
5. Stage 3 retrieves the most relevant retention playbook via RAG
6. View ranked at-risk accounts and drill into the full reasoning chain in the dashboard
7. Export full audit logs for any account from Postgres

---

## 📐 Implementation Details

**Data**
IBM Telco Churn dataset (real) + documented synthetic augmentation (support tickets, sentiment, usage decay) generated from defined archetypes — price-sensitive, feature-gap, support-friction, silent-disengagement.

**Stage 1 — Risk Model**
Logistic regression baseline → LightGBM main model. Time-based train/val/test split (no shuffling — prevents leakage). Optuna-tuned. Evaluated on PR-AUC, not accuracy. Probabilities calibrated via Platt/isotonic. SHAP values computed per account.

**Stage 2 — LLM Reasoning Agent**
Input: SHAP top features + ticket text + account metadata. Output: enforced Pydantic schema (`risk_narrative`, `primary_driver_category`, `confidence`, `recommended_action_id`). Few-shot prompted per archetype. Validated against a 30–50 example hand-labeled eval set checking SHAP-to-narrative consistency (hallucination rate reported).

**Stage 3 — Action Engine**
Retention playbooks embedded and stored in pgvector. LLM retrieves the closest-matching playbook to the driver category and personalizes it — adding a new playbook requires no code changes.

**Orchestration**
Simple sequential pipeline class (no LangGraph — unnecessary for 3 fixed stages). Every stage's output logged to Postgres for full auditability.

---

## 🏛️ Architecture

### Design Philosophy

Two stages, not one, and not more:

- **Stage 1 (statistical)** answers *"how risky is this account, and which factors drove that score?"* — grounded, calibrated, explainable by construction (SHAP).
- **Stage 2 (LLM)** answers *"what does that mean in plain language, and what should we actually do about it?"* — the model reasons over Stage 1's structured output, not raw uncertainty.

The LLM never scores risk itself. It's constrained to interpret and act on numbers a statistical model already produced. This keeps the system auditable — a risk score has a PR-AUC and a calibration curve behind it, not a hallucination risk — while still giving analysts natural-language reasoning and retrieved, relevant next steps.

No multi-agent framework, no LangGraph. Three sequential stages don't need one; adding framework complexity here would be complexity without payoff.

### System Diagram

```
┌─────────────┐     ┌──────────────────┐     ┌────────────────────┐     ┌──────────────────┐
│   Postgres   │────▶│  Feature Engine   │────▶│   Stage 1: Risk     │────▶│  Stage 2: LLM      │
│ (raw tables) │     │ (registry-driven) │     │  LightGBM + SHAP    │     │  Reasoning Agent    │
└─────────────┘     └──────────────────┘     └────────────────────┘     └─────────┬──────────┘
                                                                                     │
                                                          ┌──────────────────────────▼──────────────────────────┐
                                                          │  Stage 3: Action Engine (RAG over retention playbooks)│
                                                          └──────────────────────────┬──────────────────────────┘
                                                                                     │
                                                          ┌──────────────────────────▼──────────────────────────┐
                                                          │        Postgres audit log (score, SHAP, LLM out)     │
                                                          └──────────────────────────┬──────────────────────────┘
                                                                                     │
                                                          ┌──────────────────────────▼──────────────────────────┐
                                                          │     FastAPI  ──▶  React Dashboard (drill-down UI)    │
                                                          └───────────────────────────────────────────────────┘
```

### Data Layer

**Source data:** IBM Telco Customer Churn dataset (real, labeled) for account-level features — tenure, contract type, monthly charges, service usage. Synthetic augmentation (support tickets, sentiment, usage-decay series, NPS) is generated from a documented archetype process: price-sensitive, feature-gap, support-friction, silent-disengagement. Synthetic ≠ arbitrary — the generative logic is versioned and defensible.

**Schema (Postgres):**

| Table | Purpose |
|---|---|
| `accounts` | Core account metadata |
| `usage_events` | Time-series usage/engagement signal |
| `support_tickets` | Ticket text, sentiment, escalation flags |
| `billing_events` | Payment, downgrade, renewal history |
| `churn_label` | Ground-truth / simulated churn outcome |

### Feature Engineering

A versioned feature registry (YAML) defines every feature with rationale, not a kitchen-sink dump:

- **Usage trend** — slope over rolling windows (not point-in-time)
- **Support friction score** — ticket count, escalation rate, sentiment trajectory
- **Billing/contract risk** — renewal proximity, downgrade history, payment failures
- **Engagement breadth** — % of licensed features actually used

Feature definitions are unit-tested and versioned to guard against drift and to demonstrate reproducibility.

### Stage 1 — Risk Model (Deep Dive)

- **Baseline:** logistic regression (sanity floor — if the main model can't beat this, something's wrong)
- **Main model:** LightGBM
- **Splitting:** time-based, not random shuffle (prevents future-leak into past)
- **Tuning:** Optuna
- **Metric:** PR-AUC (accuracy is misleading under class imbalance)
- **Calibration:** Platt scaling / isotonic regression — so a "72% risk" score is a real probability, not an arbitrary rank
- **Explainability:** SHAP per-account, feeding directly into Stage 2

### Stage 2 — LLM Reasoning Agent (Deep Dive)

**Input:** SHAP top features + raw ticket text + account metadata
**Output:** enforced Pydantic/JSON schema:

```json
{
  "risk_narrative": "string",
  "primary_driver_category": "enum",
  "confidence": "float",
  "recommended_action_id": "string"
}
```

Structured output prevents hallucinated free-text and keeps downstream logic reliable. Prompts are few-shot (3–4 examples per archetype) for consistency.

**Eval set:** 30–50 hand-labeled examples checking whether the LLM's stated reasoning actually matches the SHAP top feature. This hallucination-rate check is reported as a headline number.

### Stage 3 — Action Engine (Deep Dive)

The archetype → action mapping is not hardcoded. A small retrieval layer (sentence-transformers embeddings + pgvector) sits over a set of written retention playbooks (10–15 docs). The LLM retrieves the most relevant playbook for the driver category and personalizes it. New playbooks can be added without code changes.

### Evaluation Summary

| Layer | Metric | Why it matters |
|---|---|---|
| Stage 1 | PR-AUC, calibration curve | Class imbalance makes accuracy meaningless; calibration makes probabilities trustworthy |
| Stage 2 | Hallucination rate (SHAP-match %) | Verifies the LLM's stated reasoning is grounded, not invented |
| System | Simulated churn-weighted revenue impact | Ties the pipeline back to the actual business question |

### Explicit Non-Goals

To keep scope defensible and interview-probeable rather than sprawling:

- No multi-agent framework
- No LLM calls beyond the two-stage design "because we can"
- No auth/RBAC beyond basic API auth

Time is concentrated on Stage 1 model rigor and the Stage 2 hallucination-rate eval — the parts a technical interviewer will actually probe.

---

## ⚠️ Disclaimer

> This project is built for **educational and portfolio purposes**.

- 🔺 Synthetic data is documented, not fabricated arbitrarily
- ✅ Real base dataset (IBM Telco) used for credibility
- 🧾 All metrics reported are from actual evaluation runs, not estimates

---

## 👤 Authors

- **Arjun Bohara**
- **Vatsal Garg**

---

## 🌟 Support the Project

If you find Cascade useful:

- ⭐ Star the repo
- 🍴 Fork it
- 🤝 Contribute

---

<div align="center">

![Built for](https://img.shields.io/badge/BUILT%20FOR-ML%20SYSTEMS-lightgrey?style=for-the-badge)
![Powered by](https://img.shields.io/badge/POWERED%20BY-LightGBM%20%2B%20LLM-brightgreen?style=for-the-badge)

*Cascade — Score. Explain. Act.*

</div>
