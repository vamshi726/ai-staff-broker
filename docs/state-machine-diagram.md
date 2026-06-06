# State Machine Diagram

Lifecycle states for tasks, workers, and the LangGraph agent workflow.

## 1. Task Lifecycle State Machine

```mermaid
stateDiagram-v2
    [*] --> Pending: AI creates task from<br/>voice instruction

    Pending --> Assigned: Worker matched &<br/>task translated
    Pending --> Escalated: No suitable worker found

    Assigned --> InProgress: Worker accepts task
    Assigned --> Pending: Worker rejects task
    Assigned --> Escalated: Assignment timeout

    InProgress --> Completed: Worker marks complete
    InProgress --> Escalated: Delay detected by<br/>Monitoring Agent
    InProgress --> InProgress: Worker sends<br/>voice progress update

    Escalated --> Assigned: Supervisor reassigns
    Escalated --> Cancelled: Supervisor cancels task

    Completed --> [*]
    Cancelled --> [*]

    note right of Pending
        Task created by Task Planner.
        Awaiting worker assignment.
    end note

    note right of InProgress
        Monitoring Agent tracks
        delays and progress updates.
    end note

    note right of Escalated
        Supervisor notified via
        WebSocket for manual action.
    end note
```

## 2. Task Status Transitions (Detailed)

```mermaid
stateDiagram-v2
    direction LR

    state "pending" as Pending
    state "assigned" as Assigned
    state "in_progress" as InProgress
    state "completed" as Completed
    state "escalated" as Escalated
    state "cancelled" as Cancelled

    [*] --> Pending

    Pending --> Assigned: assign_worker()
    Pending --> Escalated: no_worker_available()

    Assigned --> InProgress: worker_accept()
    Assigned --> Pending: worker_reject()
    Assigned --> Escalated: assignment_timeout()

    InProgress --> Completed: worker_complete()
    InProgress --> Escalated: delay_detected()
    InProgress --> InProgress: voice_update()

    Escalated --> Assigned: supervisor_reassign()
    Escalated --> Cancelled: supervisor_cancel()

    Completed --> [*]
    Cancelled --> [*]
```

## 3. LangGraph Agent Workflow State Machine

```mermaid
stateDiagram-v2
    [*] --> SpeechProcessing: Audio received

    SpeechProcessing --> IntentUnderstanding: STT complete<br/>(text + language)
    SpeechProcessing --> Failed: STT error

    IntentUnderstanding --> TaskPlanning: Intent extracted
    IntentUnderstanding --> Failed: Unrecognized intent

    TaskPlanning --> WorkerAssignment: Sub-tasks created
    TaskPlanning --> Failed: Planning error

    WorkerAssignment --> Translation: Workers matched
    WorkerAssignment --> Escalation: No match found

    Translation --> Notification: All languages translated
    Translation --> Failed: Translation error

    Notification --> AwaitingResponse: Tasks delivered<br/>via WebSocket + TTS

    AwaitingResponse --> Monitoring: Worker responds
    AwaitingResponse --> Escalation: Response timeout

    Monitoring --> AwaitingResponse: Progress update logged
    Monitoring --> Completed: All tasks completed
    Monitoring --> Escalation: Delay detected

    Escalation --> WorkerAssignment: Supervisor triggers reassignment
    Escalation --> Cancelled: Supervisor cancels

    Completed --> DashboardUpdate: Push final status
    DashboardUpdate --> [*]

    Failed --> [*]
    Cancelled --> [*]
```

## 4. Worker Availability State Machine

```mermaid
stateDiagram-v2
    [*] --> Available: Worker onboarded

    Available --> Busy: Task accepted<br/>(in_progress)
    Available --> Offline: Worker goes offline

    Busy --> Available: Task completed
    Busy --> Busy: Additional task<br/>assigned (if capacity)
    Busy --> Offline: Worker goes offline<br/>mid-task

    Offline --> Available: Worker comes online
    Offline --> Busy: Resume in-progress task

    note right of Available
        Assignment Agent queries
        available workers only.
    end note

    note right of Busy
        Worker may have multiple
        tasks based on capacity.
    end note
```

## 5. Voice Message Processing State Machine

```mermaid
stateDiagram-v2
    [*] --> Uploaded: Audio file received

    Uploaded --> Transcribing: STT processing
    Transcribing --> Transcribed: Text extracted
    Transcribing --> Failed: STT error

    Transcribed --> Processing: Route to agent pipeline
    Processing --> Translated: Translation complete
    Processing --> Failed: Agent error

    Translated --> Synthesizing: TTS generation
    Synthesizing --> Delivered: Audio URL stored &<br/>notification sent
    Synthesizing --> Failed: TTS error

    Delivered --> Acknowledged: Worker plays audio
    Acknowledged --> [*]

    Failed --> [*]
```

## State Reference Table

### Task States

| State | Description | Triggered By |
|-------|-------------|--------------|
| `pending` | Task created, awaiting assignment | Task Planner Agent |
| `assigned` | Worker matched, notification sent | Assignment Agent |
| `in_progress` | Worker accepted and is working | Worker action |
| `completed` | Task finished successfully | Worker voice/button update |
| `escalated` | Requires supervisor intervention | Monitoring Agent or timeout |
| `cancelled` | Task terminated by supervisor | Supervisor action |

### LangGraph Workflow States

| State | Agent | Input → Output |
|-------|-------|----------------|
| SpeechProcessing | Agent 1 | Audio → Structured text + language |
| IntentUnderstanding | Agent 2 | Text → {task, priority} |
| TaskPlanning | Agent 3 | Intent → Sub-task list |
| WorkerAssignment | Agent 4 | Tasks → Worker mapping |
| Translation | Agent 5 | Text → Per-worker translated text |
| Notification | System | Translated text → TTS + WebSocket |
| Monitoring | Agent 6 | Worker response → Status update |
