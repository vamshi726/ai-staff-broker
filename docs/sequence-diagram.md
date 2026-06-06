# Sequence Diagram

Interaction flows between actors, API modules, agents, and external services.

## 1. End-to-End Voice Instruction Flow

Primary scenario: Owner speaks in Telugu; workers receive tasks in Tamil and Kannada.

```mermaid
sequenceDiagram
    autonumber
    actor Owner
    participant OP as Owner Portal
    participant API as FastAPI Backend
    participant LG as LangGraph Workflow
    participant STT as Speech Agent<br/>(Sarvam STT)
    participant INT as Intent Agent<br/>(Gemini)
    participant PLN as Task Planner<br/>(Gemini)
    participant ASN as Assignment Agent
    participant TRN as Translation Agent<br/>(Sarvam)
    participant TTS as TTS Service<br/>(Sarvam)
    participant DB as PostgreSQL
    participant WS as WebSocket
    actor W1 as Worker 1 (Tamil)
    actor W2 as Worker 2 (Kannada)

    Owner->>OP: Record voice instruction<br/>"Move rice bags & restock Shelf 5"
    OP->>API: POST /voice/upload (audio + JWT)
    API->>LG: Trigger workflow

    LG->>STT: Convert audio to text
    STT-->>LG: Text (Telugu) + detected language

    LG->>INT: Extract intent & actions
    INT-->>LG: {task, priority}

    LG->>PLN: Break into sub-tasks
    PLN-->>LG: Task 1: Unload rice bags<br/>Task 2: Restock Shelf 5

    LG->>ASN: Find suitable workers
    ASN->>DB: Query users (skills, language, availability)
    DB-->>ASN: Worker 1 (inventory, Tamil)<br/>Worker 2 (stocking, Kannada)
    ASN-->>LG: Assignment mapping

    LG->>TRN: Translate Task 1 → Tamil
    TRN-->>LG: Translated text (Tamil)
    LG->>TTS: Generate Tamil audio
    TTS-->>LG: Audio URL

    LG->>TRN: Translate Task 2 → Kannada
    TRN-->>LG: Translated text (Kannada)
    LG->>TTS: Generate Kannada audio
    TTS-->>LG: Audio URL

    LG->>DB: Persist tasks, assignments, voice messages
    LG->>WS: Broadcast new tasks

    WS-->>W1: Task 1 notification + Tamil audio
    WS-->>W2: Task 2 notification + Kannada audio
    WS-->>OP: Dashboard update (pending tasks)

    W1->>OP: Accept task & complete work
    W1->>API: POST /tasks/{id}/complete (voice update)
    API->>LG: Monitoring Agent processes update
    LG->>DB: Update task status → completed
    LG->>WS: Broadcast completion
    WS-->>Owner: Real-time dashboard update
```

## 2. Authentication & Organization Setup

```mermaid
sequenceDiagram
    autonumber
    actor Owner
    participant FE as Frontend Portal
    participant AUTH as Auth Module
    participant ORG as Organization Module
    participant DB as PostgreSQL
    participant REDIS as Redis

    Owner->>FE: Register (name, email, password)
    FE->>AUTH: POST /auth/register
    AUTH->>DB: Create user record
    AUTH-->>FE: JWT token

    Owner->>FE: Create organization
    FE->>ORG: POST /organizations (JWT)
    ORG->>DB: Insert organization
    ORG-->>FE: Organization created

    Owner->>FE: Add supervisor & workers
    FE->>ORG: POST /users (role, language, skills)
    ORG->>DB: Insert users with RBAC roles
    ORG-->>FE: Users added

    Owner->>FE: Login
    FE->>AUTH: POST /auth/login
    AUTH->>DB: Validate credentials
    AUTH->>REDIS: Cache session
    AUTH-->>FE: JWT token + role
    FE-->>Owner: Redirect to role-based portal
```

## 3. Worker Task Acceptance & Voice Response

```mermaid
sequenceDiagram
    autonumber
    actor Worker
    participant WP as Worker Portal
    participant API as FastAPI Backend
    participant VOICE as Voice Module
    participant STT as Sarvam STT
    participant MON as Monitoring Agent
    participant DB as PostgreSQL
    participant WS as WebSocket
    actor Supervisor

    Worker->>WP: Open My Tasks
    WP->>API: GET /tasks/assigned (JWT)
    API->>DB: Fetch worker tasks
    DB-->>API: Task list with audio URLs
    API-->>WP: Translated tasks

    Worker->>WP: Play voice instruction (TTS audio)
    Worker->>WP: Accept task
    WP->>API: PATCH /tasks/{id} status=in_progress
    API->>DB: Update task status
    API->>WS: Broadcast status change

    Worker->>WP: Record voice update
    WP->>API: POST /voice/upload (audio)
    API->>VOICE: Process worker response
    VOICE->>STT: Speech to text
    STT-->>VOICE: Transcribed update
    VOICE->>MON: Log progress
    MON->>DB: Insert voice message + task history
    MON->>WS: Push update to dashboard

    alt Task completed
        Worker->>WP: Mark task completed
        WP->>API: PATCH /tasks/{id} status=completed
        API->>DB: Update task + history
        API->>WS: Broadcast completion
    else Task delayed — escalation
        MON->>WS: Alert supervisor
        WS-->>Supervisor: Escalation notification
        Supervisor->>API: Reassign task
        API->>DB: Update assignment
    end
```

## 4. Supervisor Task Reassignment

```mermaid
sequenceDiagram
    autonumber
    actor Supervisor
    participant SP as Supervisor Portal
    participant API as Task Module
    participant ASN as Assignment Agent
    participant TRN as Translation Agent
    participant DB as PostgreSQL
    participant WS as WebSocket
    actor NewWorker as New Worker

    Supervisor->>SP: View escalated task
    SP->>API: GET /tasks?status=escalated
    API->>DB: Query escalated tasks
    DB-->>API: Task details
    API-->>SP: Display task

    Supervisor->>SP: Select new worker
    SP->>API: PATCH /tasks/{id}/assign
    API->>ASN: Validate worker (skill, language, availability)
    ASN->>DB: Check worker profile
    DB-->>ASN: Worker available

    API->>TRN: Translate task to new worker language
    TRN-->>API: Translated instruction
    API->>DB: Update assignment + task history
    API->>WS: Notify new worker
    WS-->>NewWorker: New task assignment
    WS-->>Supervisor: Reassignment confirmed
```
