# Architecture Diagram

High-level system architecture for the Multilingual Voice-Based Workforce Management Agent MVP.

## System Architecture Overview

```mermaid
flowchart TB
    subgraph Clients["Client Layer"]
        OP["Owner Portal<br/>(React + Vite)"]
        SP["Supervisor Portal<br/>(React + Vite)"]
        WP["Worker Portal<br/>(React + Vite)"]
    end

    subgraph API["API Layer — FastAPI"]
        AUTH["Authentication Module<br/>JWT / RBAC"]
        ORG["Organization Module"]
        TASK["Task Module"]
        VOICE["Voice Module"]
        DASH["Dashboard Module"]
        WS["WebSocket Server"]
    end

    subgraph Agents["Agent Engine — LangGraph"]
        A1["Agent 1<br/>Speech Processing"]
        A2["Agent 2<br/>Intent Understanding"]
        A3["Agent 3<br/>Task Planning"]
        A4["Agent 4<br/>Worker Assignment"]
        A5["Agent 5<br/>Translation"]
        A6["Agent 6<br/>Monitoring"]
        WF["LangGraph Workflow<br/>Orchestrator"]
    end

    subgraph Data["Data Layer"]
        PG[("PostgreSQL<br/>Organizations, Users,<br/>Tasks, Voice Messages,<br/>Task History")]
        REDIS[("Redis<br/>Cache & Sessions")]
    end

    subgraph External["External Services"]
        SARVAM["Sarvam AI<br/>STT · Translation · TTS"]
        GEMINI["Gemini 2.5 Flash<br/>Intent & Planning"]
    end

    subgraph Deploy["Deployment"]
        CLOUD["Azure / DigitalOcean"]
    end

    OP --> AUTH
    SP --> AUTH
    WP --> AUTH

    OP --> ORG
    OP --> TASK
    OP --> VOICE
    OP --> DASH
    SP --> TASK
    SP --> DASH
    WP --> TASK
    WP --> VOICE

    OP -.->|real-time| WS
    SP -.->|real-time| WS
    WP -.->|real-time| WS

    AUTH --> PG
    ORG --> PG
    TASK --> PG
    VOICE --> PG
    DASH --> PG
    AUTH --> REDIS
    DASH --> REDIS

    VOICE --> WF
    TASK --> WF
    WF --> A1 --> A2 --> A3 --> A4 --> A5 --> A6
    A6 --> TASK
    A6 --> WS

    A1 --> SARVAM
    A5 --> SARVAM
    VOICE --> SARVAM
    A2 --> GEMINI
    A3 --> GEMINI
    A4 --> PG

    API --> CLOUD
    Data --> CLOUD
```

## Monorepo Structure

```mermaid
flowchart LR
    ROOT["ai-staff-broker/"]

    ROOT --> FE["frontend/<br/>React + Vite + Tailwind"]
    ROOT --> BE["backend/<br/>FastAPI + SQLAlchemy"]
    ROOT --> DC["docker-compose.yml<br/>PostgreSQL + Redis"]
    ROOT --> DOCS["docs/<br/>Diagrams & Docs"]

    FE --> FE_SRC["src/<br/>App.jsx, portals, API client"]
    BE --> BE_APP["app/"]
    BE_APP --> MAIN["main.py"]
    BE_APP --> MODELS["models/"]
    BE_APP --> API_MOD["api/"]
    BE_APP --> CORE["core/"]
    BE_APP --> SERVICES["services/"]
    BE_APP --> AGENTS["agents/workflow.py"]
```

## Agent Pipeline Architecture

```mermaid
flowchart LR
    AUDIO["Owner Audio<br/>(Telugu)"] --> STT["Speech Agent<br/>Sarvam STT"]
    STT --> TEXT["Structured Text<br/>+ Language Detection"]
    TEXT --> INTENT["Intent Agent<br/>Gemini"]
    INTENT --> PLAN["Task Planner<br/>Gemini"]
    PLAN --> ASSIGN["Assignment Agent<br/>Skills + Language Match"]
    ASSIGN --> TRANS["Translation Agent<br/>Sarvam"]
    TRANS --> TTS["Text-to-Speech<br/>Sarvam TTS"]
    TTS --> NOTIFY["Worker Notification<br/>WebSocket"]
    NOTIFY --> RESPONSE["Worker Voice Response"]
    RESPONSE --> MONITOR["Monitoring Agent"]
    MONITOR --> DASHBOARD["Dashboard Update"]
```

## Technology Stack

| Layer | Technology |
|-------|------------|
| Frontend | React, Vite, TailwindCSS, React Router, Axios |
| Backend | FastAPI, Uvicorn, SQLAlchemy |
| Agent Framework | LangGraph |
| LLM | Gemini 2.5 Flash |
| Speech / Translation / TTS | Sarvam AI |
| Database | PostgreSQL 15 |
| Cache | Redis 7 |
| Real-time | WebSockets |
| Authentication | JWT + RBAC |
| Deployment | Azure or DigitalOcean |
