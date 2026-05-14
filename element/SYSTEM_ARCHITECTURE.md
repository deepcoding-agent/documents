# PrepPilot — System Architecture

> Companion to `docs/system_architecture.svg` (the rendered image diagram).
> This file holds the editable Mermaid source so the diagram can be regenerated.

---

## Visual diagram

![PrepPilot System Architecture](./system_architecture.svg)

The SVG above mirrors the reference style: grouped boxes for **Services** (LLMs),
**Web** (Next.js), **Backend** (FastAPI + agents), **Storage**, **CI/CD** and
**Hosting**, with a 7-step user flow strip across the bottom.

---

## High-level flow (Mermaid)

```mermaid
flowchart LR
    %% ─── Services ─────────────────────────────
    subgraph S["SERVICES (LLM Providers)"]
        OAI["OpenAI GPT<br/>gpt-4o-mini"]
        ANT["Anthropic Claude<br/>claude-sonnet-4-6"]
    end

    %% ─── User ─────────────────────────────────
    U(("Data scientist<br/>(browser)"))

    %% ─── Web ──────────────────────────────────
    subgraph W["WEB · Next.js 16"]
        FE["React 19 UI<br/>ChatPanel · DatasetPicker · ModelsPanel<br/>TrainConfigPanel · PredictionResultCard"]
        AR["API routes (thin proxy)<br/>/api/chat /api/train /api/predict<br/>/api/datasets /api/conversations"]
        AUTH["NextAuth v5<br/>Google OAuth"]
        PR["Prisma ORM"]
    end

    %% ─── Backend ──────────────────────────────
    subgraph B["BACKEND · FastAPI"]
        API["uvicorn :8000<br/>/chat /train /predict /insights …"]
        ROUTE["Two-stage AI routing<br/>category router → focused planner"]
        subgraph AG["Agents"]
            DS["DS-Agent (orchestrator)"]
            TA["TrainAgent /train"]
            PA["PredictAgent /predict"]
            AC["AutoClean /cleaning"]
            AP["AutoPrepare /ml-prepare"]
            INS["Insights /insights"]
            DOC["Document /report"]
        end
        HR["HANDLER_REGISTRY · 417 handlers<br/>stats · clean · transform · viz<br/>feature · nlp · analysis"]
        SBX["Sandboxed exec()<br/>pandas · sklearn · plotly · xgboost · optuna"]
        MR["Model registry<br/>27 algorithms · CV · Optuna tuning"]
    end

    %% ─── Storage ──────────────────────────────
    subgraph ST["STORAGE"]
        DB[("Database<br/>SQLite (dev) · MongoDB (prod)<br/>User · Conversation · Dataset")]
        MOD[("models/{uuid}.joblib<br/>+ {uuid}.json metadata")]
    end

    %% ─── CI/CD + Hosting ─────────────────────
    GH["GitHub Actions<br/>tests · type-check · lint"]
    HOST["Hosting<br/>Docker · Vercel (FE) · Cloud Run (BE)"]

    %% ─── Edges ────────────────────────────────
    U -- HTTPS --> FE
    FE --> AR
    AR -- thin proxy --> API
    AUTH --> PR
    AR --> PR
    PR --> DB
    API --> ROUTE
    ROUTE --> AG
    DS --> HR
    TA --> MR
    TA --> SBX
    PA --> MR
    AC --> HR
    AP --> HR
    INS --> HR
    DOC --> HR
    HR --> SBX
    AG -. LLM call .-> OAI
    AG -. LLM call .-> ANT
    TA -- save --> MOD
    PA -- load --> MOD
    AR -- dataset rows --> DB
    GH -- deploy --> HOST
    HOST -. serves .-> U
```

## Model lifecycle (Mermaid)

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant FE as Next.js UI
    participant AR as /api/* (proxy)
    participant API as FastAPI
    participant TA as TrainAgent
    participant PA as PredictAgent
    participant FS as models/ on disk
    participant DB as Prisma DB

    U->>FE: Upload dataset
    FE->>AR: POST /api/datasets
    AR->>DB: insert rows + meta
    U->>FE: /train (active dataset)
    FE->>AR: POST /api/train { datasetId }
    AR->>DB: read dataset rows
    AR->>API: POST /train { rows, columns, target }
    API->>TA: run_training(...)
    TA->>TA: 27 algos · CV · Optuna · evaluate
    TA->>FS: save {uuid}.joblib + .json
    TA-->>API: { model_id, metrics, charts }
    API-->>AR: TrainResponse
    AR-->>FE: TrainResultArtifact
    FE-->>U: TrainResultCard (download · "Use for /predict")

    U->>FE: Open Models tab → "Use for /predict"
    FE->>AR: POST /api/predict { datasetId, modelId }
    AR->>DB: read dataset rows
    AR->>API: POST /predict { model_id, rows, columns }
    API->>PA: run_prediction(...)
    PA->>FS: load {uuid}.joblib
    PA->>PA: re-apply scaler/encoders/onehot
    PA->>PA: model.predict(X)
    PA-->>API: { rows + predicted_target [+ probability] }
    API-->>AR: PredictResponse
    AR-->>FE: PredictionResultArtifact
    FE-->>U: PredictionResultCard (table · distribution · export CSV/XLSX)
```

## Repo / runtime layout

```mermaid
flowchart TB
    subgraph Repo["seniorproject/ (4 independent repos)"]
        WA["web-app/<br/>Next.js 16 · React 19"]
        ML["ml-datascience/<br/>FastAPI · agents · 417 handlers · models/"]
        IC["installation-core/<br/>run.py · docker-compose"]
        DOC["docs/<br/>SPRINT_PLAN.md · system_architecture.svg"]
    end

    IC --start--> WA
    IC --start--> ML
    WA --HTTP :3000--> Browser
    ML --HTTP :8000--> WA
    Browser((Browser))
```

---

## Architectural rules (from CLAUDE.md)

1. **Thin proxy pattern** — `/api/*` routes only auth + forward; zero ML logic in TypeScript
2. **Sandboxed `exec()`** — user-generated Python only runs in `api/sandbox.py` namespace
3. **Database-backed sessions** — NextAuth v5 + PrismaAdapter (never JWT)
4. **Multi-provider LLM** — OpenAI default, Anthropic optional; chosen per request
5. **Two-stage routing** — category router → focused planner (no keyword maps)
6. **Auto-retry** — failed sandbox code is sent back to LLM once before erroring
7. **Plotly-first** — all DS-Agent charts use Plotly; matplotlib only for missingno fallback

## Sprint state

- ✅ Sprints 1 → 4 (data prep, charts, AI routing, 417 handlers, AutoML `/train`)
- ✅ Sprint 5 — Models tab + `/predict` (this commit)
- 🔲 Sprint 6 — automated tests (pytest + vitest, 80% coverage)
- 🔲 Sprint 7 — production hardening (rate limit, sandbox lockdown, CORS)
- 🔲 Sprint 8 — production deployment (MongoDB Atlas, S3/GCS, multi-stage Docker)

---

## How to regenerate the SVG

The SVG was hand-authored to match the reference style and lives at
`docs/system_architecture.svg`. Edit it directly with any text editor or
vector tool, or open it in the browser to view at full resolution.

For a quick auto-generated PNG from the Mermaid blocks above, paste any
diagram into <https://mermaid.live> and export.
