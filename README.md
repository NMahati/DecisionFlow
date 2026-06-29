## Team Details

Team Name - TriAInity

Project Title - DecisionFlow AI - Agentic Decision Intelligence for Customer Success(Hackathon Project 2: Intelligent Next Best
Action Platform)

Members:

Chepyala Sahasra Reddy - 23071A67E4 (CSE - DataScience)

Mahati N - 23071A67F9 (CSE - DataScience)

Pilli Amrutha Varshini - 23071A67G9(CSE - DataScience)


# DecisionFlow AI

**An Agentic Decision Intelligence Platform for Customer Success.**

DecisionFlow AI is *not* a chatbot. It is a reusable, explainable, multi-agent
platform that ingests customer interactions (meeting transcripts, CRM data,
emails, support tickets, notes), reasons over them with a team of specialised
agents, and recommends the **Next Best Actions** a Customer Success Manager
should take before a renewal — with full evidence for every recommendation and
a human-in-the-loop approval step.

> **Domain:** Customer Success for SaaS · **Example account:** ABC Technologies · **User:** Customer Success Manager

---

## Project Overview

When a CSM uploads everything they know about an account, a **Planner Agent**
decides which specialist agents to run. Each agent does one job well —
summarising meetings, reading CRM data, retrieving enterprise knowledge,
scoring risk, finding expansion opportunities, generating recommendations, and
explaining them. A **Memory Agent** persists every analysis and every human
decision, so future recommendations are informed by past history.

Key properties:

- **Modular** — each agent lives in its own file behind a common interface.
- **Reusable** — swap the domain by changing prompts + knowledge docs.
- **Explainable** — every recommendation carries transcript, CRM, knowledge and business-rule evidence.
- **Human-in-the-loop** — approve / reject / modify each Next Best Action.
- **Memory-aware** — approved and rejected decisions feed future runs.
- **Demo-safe** — runs fully offline in MOCK mode when no API key is present.

---
## GitHub Repository Link



## Setup

### 1. Backend

```bash
cd decisionflow-ai/backend
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env         # optional: add GOOGLE_API_KEY for real LLM calls
uvicorn main:app --reload --port 8000
```

> **No API key?** Leave `GOOGLE_API_KEY` blank — the platform runs in **MOCK
> mode** and every agent returns realistic, deterministic output, so the full
> workflow and demo work offline.

### 2. Frontend

```bash
cd decisionflow-ai/frontend
npm install
npm run dev
```

## Architecture

<img width="1331" height="802" alt="image" src="https://github.com/user-attachments/assets/879c1b61-55a3-428b-98bf-cf9a6f7cedad" />

**Stack**

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 14, TailwindCSS, shadcn-style UI, lucide-react |
| Backend | FastAPI, Uvicorn, Pydantic |
| AI orchestration | LangGraph + LangChain |
| LLM | Google Gemini (`gemini-1.5-flash`) with deterministic mock fallback |
| Vector DB | ChromaDB (keyword fallback if unavailable) |
| Database / Memory | SQLite |

---

## Folder Structure

```
decisionflow-ai/
├── backend/
│   ├── main.py              # FastAPI entrypoint
│   ├── config.py            # central config (no hardcoded business logic)
│   ├── llm.py               # Gemini wrapper + MOCK fallback
│   ├── logging_config.py    # shared logger
│   ├── agents/
│   │   ├── base.py          # reusable BaseAgent + REGISTRY
│   │   ├── planner.py
│   │   ├── conversation_agent.py
│   │   ├── crm_agent.py
│   │   ├── knowledge_agent.py
│   │   ├── risk_agent.py
│   │   ├── opportunity_agent.py
│   │   ├── recommendation_agent.py
│   │   ├── explanation_agent.py
│   │   └── memory_agent.py
│   ├── tools/
│   │   ├── rag.py
│   │   ├── vector_store.py   # ChromaDB
│   │   ├── crm_tool.py
│   │   └── memory_tool.py
│   ├── workflow/
│   │   ├── graph.py          # LangGraph orchestration
│   │   └── state.py
│   ├── api/routes.py         # REST endpoints
│   ├── db/database.py        # SQLite persistence
│   ├── knowledge/            # enterprise docs (RAG corpus)
│   └── sample_data/          # transcript, crm.json, email, ticket, notes
├── frontend/                 # Next.js dashboard
└── README.md
```

---



---

## How it works (the workflow)

```
Upload → Planner → Conversation → CRM → Knowledge → Risk → Opportunity
       → Recommendation → Explanation → Dashboard → Approve/Reject/Modify → Memory
       → future interactions reuse stored memory
```

The **Planner** inspects which inputs were supplied and emits an execution plan;
the LangGraph workflow runs only the planned agents. State accumulates as each
agent contributes its slice, and every execution is logged with timing.

---

## Agents

| Agent | Responsibility | Key output |
|-------|----------------|-----------|
| **Planner** | Pure orchestration — decides which agents run. No business reasoning. | execution plan |
| **Conversation** | Analyse meeting transcript | summary, pain points, goals, sentiment, competitors, urgency, gaps |
| **CRM** | Normalise CRM data | name, renewal_days, ARR, health, opportunities |
| **Knowledge** | RAG over `knowledge/` via ChromaDB | relevant playbook / docs snippets |
| **Risk** | Detect renewal & account risks | risk level, risks, confidence |
| **Opportunity** | Find upsell / cross-sell / adoption | opportunity list, priority, confidence |
| **Recommendation** | Generate top 5 Next Best Actions | action, priority, confidence, impact, reason, outcome |
| **Explanation** | Attach evidence to each recommendation | transcript + CRM + knowledge + business rules |
| **Memory** | Recall prior history; persist runs & decisions | customer history timeline |

**Adding a new agent** = create one file, subclass `BaseAgent`, decorate with
`@register`, and add its key to `DEFAULT_PIPELINE` in `config.py`. No
orchestration code changes.

---

## API

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/upload` | Stage customer inputs |
| POST | `/api/analyze` | Run the full agent workflow |
| GET | `/api/recommendations` | List recommendations |
| POST | `/api/approve` | Approve a recommendation (stored to memory) |
| POST | `/api/reject` | Reject a recommendation |
| POST | `/api/modify` | Modify a recommendation's action |
| GET | `/api/memory` | Retrieve customer memory timeline |
| GET | `/api/customer` | Customer record + memory |

---

## User Interface

- **Sidebar** — Dashboard / Memory navigation.
- **Customer Overview** — name, health score, renewal date, sentiment, risk level.
- **Agent Execution Timeline** — live status + timing for all nine agents.
- **Recommendation Cards** — action, priority, confidence, reason, impact, outcome, with **Approve / Reject / Modify**.
- **Evidence Panel** — transcript, CRM, knowledge and business-rule evidence per recommendation.
- **Memory Page** — previous interactions, approved/rejected decisions, customer history.

---

## Screenshots

Dashboard with customer overview + agent timeline.

<img width="958" height="499" alt="image" src="https://github.com/user-attachments/assets/6affabd4-14e9-4ab6-90a1-2e7c726e93f4" />

Recommendation cards with the evidence panel open.

<img width="938" height="490" alt="image" src="https://github.com/user-attachments/assets/84c8fa52-3f41-4c80-906b-64a15570ec39" />

Memory timeline after approving a few actions.

<img width="937" height="488" alt="image" src="https://github.com/user-attachments/assets/28e952ec-97ff-4f9d-8252-b69d26f402a7" />


---
## Project Links

Frontend → Vercel: https://decisionflow-ai-tau.vercel.app/

Backend → Render: https://decisionflow-backend.onrender.com

Project Documentation: https://drive.google.com/file/d/1Wfakt_3W3zBO-Gh0ldfPTQ2koryGOeBe/view?usp=sharing


## Future Improvements

- Streaming agent execution (Server-Sent Events) for real-time timeline updates.
- Per-customer ChromaDB namespaces and semantic memory recall.
- Confidence calibration and A/B testing of recommendation quality.
- Connector ingestion (Salesforce, Zendesk, Gong) instead of manual upload.
- Role-based access control and audit logging for enterprise deployment.
- Outcome tracking loop: feed realised results back to improve recommendations.
