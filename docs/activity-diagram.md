# Activity Diagram

Business process workflows for the Multilingual Voice-Based Workforce Management Agent.

## 1. Voice Instruction to Task Completion (Main Flow)

```mermaid
flowchart TD
    START((Start)) --> RECORD["Owner records or uploads<br/>voice instruction"]
    RECORD --> UPLOAD["Upload audio to Voice Module"]
    UPLOAD --> STT["Speech Agent: STT + language detection"]
    STT --> INTENT["Intent Agent: extract actions & priority"]
    INTENT --> PLAN{"Multiple actions<br/>detected?"}

    PLAN -->|Yes| SPLIT["Task Planner: break into sub-tasks"]
    PLAN -->|No| SINGLE["Create single task"]
    SPLIT --> PRIORITIZE["Prioritize tasks"]
    SINGLE --> PRIORITIZE

    PRIORITIZE --> ASSIGN["Assignment Agent: match workers<br/>(skills + language + availability)"]
    ASSIGN --> FOUND{"Suitable worker<br/>found?"}

    FOUND -->|No| ESCALATE["Flag for supervisor review"]
    ESCALATE --> MANUAL["Supervisor manually assigns"]
    MANUAL --> TRANSLATE

    FOUND -->|Yes| TRANSLATE["Translation Agent: translate<br/>to each worker's language"]
    TRANSLATE --> TTS["Generate TTS audio per worker"]
    TTS --> NOTIFY["Deliver tasks via WebSocket"]
    NOTIFY --> DASH_PENDING["Update dashboard: Pending"]

    NOTIFY --> WORKER_RECV["Worker receives translated task"]
    WORKER_RECV --> LISTEN["Worker listens to voice instruction"]
    LISTEN --> ACCEPT{"Worker accepts<br/>task?"}

    ACCEPT -->|No| REASSIGN["Return to assignment pool"]
    REASSIGN --> ASSIGN

    ACCEPT -->|Yes| INPROG["Status → In Progress"]
    INPROG --> WORK["Worker performs task"]
    WORK --> UPDATE{"Worker sends<br/>voice update?"}

    UPDATE -->|Yes| LOG["Monitoring Agent logs progress"]
    LOG --> WORK

    UPDATE -->|No| COMPLETE{"Task<br/>completed?"}
    COMPLETE -->|No| DELAY{"Delay<br/>detected?"}
    DELAY -->|Yes| ESCALATE2["Supervisor escalation"]
    ESCALATE2 --> MANUAL
    DELAY -->|No| WORK

    COMPLETE -->|Yes| DONE["Worker marks task completed"]
    DONE --> MONITOR["Monitoring Agent updates status"]
    MONITOR --> DASH_DONE["Update dashboard: Completed"]
    DASH_DONE --> END((End))
```

## 2. Organization & User Setup

```mermaid
flowchart TD
    START((Start)) --> REGISTER["Owner registers account"]
    REGISTER --> LOGIN["Owner logs in"]
    LOGIN --> CREATE_ORG["Create organization"]
    CREATE_ORG --> ADD_SUP["Add supervisor(s)"]
    ADD_SUP --> ADD_WORK["Add worker(s)"]
    ADD_WORK --> SET_LANG["Configure preferred languages<br/>for each user"]
    SET_LANG --> SET_SKILL["Set skills & availability"]
    SET_SKILL --> RBAC["Assign roles: Owner / Supervisor / Worker"]
    RBAC --> VERIFY{"All users<br/>configured?"}
    VERIFY -->|No| ADD_WORK
    VERIFY -->|Yes| READY["Organization ready for<br/>voice instructions"]
    READY --> END((End))
```

## 3. LangGraph Agent Workflow

```mermaid
flowchart TD
    START((Owner Voice)) --> A1["Agent 1: Speech Processing<br/>Audio → Structured Text"]
    A1 --> A2["Agent 2: Intent Understanding<br/>Extract actions & priority"]
    A2 --> A3["Agent 3: Task Planning<br/>Break into sub-tasks"]
    A3 --> A4["Agent 4: Worker Assignment<br/>Match skills, language, availability"]
    A4 --> A5["Agent 5: Translation<br/>Telugu → Tamil / Hindi / Kannada"]
    A5 --> NOTIFY["Worker Notification<br/>TTS + WebSocket delivery"]
    NOTIFY --> RESPONSE["Worker Response<br/>Voice update or completion"]
    RESPONSE --> A6["Agent 6: Monitoring<br/>Track progress & delays"]
    A6 --> UPDATE["Task Status Update"]
    UPDATE --> DASH["Dashboard<br/>Real-time broadcast"]
    DASH --> END((End))
```

## 4. Supervisor Escalation Handling

```mermaid
flowchart TD
    START((Escalation Triggered)) --> REASON{"Escalation<br/>reason?"}

    REASON -->|No worker available| VIEW_POOL["View available workers"]
    REASON -->|Task delayed| VIEW_STATUS["Review task history & delays"]
    REASON -->|Worker rejected task| VIEW_REJECT["Review rejection details"]

    VIEW_POOL --> DECIDE["Supervisor decides action"]
    VIEW_STATUS --> DECIDE
    VIEW_REJECT --> DECIDE

    DECIDE --> ACTION{"Action?"}
    ACTION -->|Reassign| SELECT["Select new worker"]
    ACTION -->|Reprioritize| PRIORITY["Change task priority"]
    ACTION -->|Cancel| CANCEL["Cancel task"]

    SELECT --> TRANSLATE["Re-translate for new worker"]
    TRANSLATE --> NOTIFY["Notify worker via WebSocket"]
    PRIORITY --> NOTIFY
    CANCEL --> LOG["Log cancellation in task history"]

    NOTIFY --> MONITOR["Monitoring Agent resumes tracking"]
    LOG --> MONITOR
    MONITOR --> END((End))
```

## 5. Dashboard Data Flow

```mermaid
flowchart LR
    TASKS["Task Module"] --> AGG["Aggregate task counts"]
    USERS["User Module"] --> STATUS["Worker availability status"]
    HISTORY["Task History"] --> REPORTS["Generate reports"]

    AGG --> DASH["Dashboard API"]
    STATUS --> DASH
    REPORTS --> DASH

    DASH --> WS["WebSocket broadcast"]
    WS --> OWNER_DASH["Owner Dashboard"]
    WS --> SUP_DASH["Supervisor Dashboard"]

    DASH --> PENDING["Pending Tasks"]
    DASH --> INPROG["In-Progress Tasks"]
    DASH --> COMPLETED["Completed Tasks"]
    DASH --> WORKERS["Assigned Workers"]
```
