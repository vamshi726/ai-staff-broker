# Use Case Diagram

Actors and use cases for the Multilingual Voice-Based Workforce Management Agent MVP.

## Primary Use Case Diagram

```mermaid
flowchart TB
    subgraph System["MVWMA System Boundary"]
        UC1["Create Organization"]
        UC2["Add Users & Assign Roles"]
        UC3["Configure Preferred Languages"]
        UC4["Login / Register"]
        UC5["Record Voice Instruction"]
        UC6["Upload Audio Instruction"]
        UC7["Monitor Task Progress"]
        UC8["View Reports"]
        UC9["Review Tasks"]
        UC10["Reassign Tasks"]
        UC11["Monitor Workers"]
        UC12["Handle Escalations"]
        UC13["Receive Translated Tasks"]
        UC14["Listen to Voice Instructions"]
        UC15["Accept Task"]
        UC16["Send Voice Update"]
        UC17["Mark Task Completed"]
        UC18["Process Speech to Text"]
        UC19["Understand Intent & Extract Tasks"]
        UC20["Plan & Prioritize Tasks"]
        UC21["Assign Workers by Skill & Language"]
        UC22["Translate Instructions"]
        UC23["Generate Voice Instructions"]
        UC24["Track Progress & Update Dashboard"]
    end

    Owner(("Owner"))
    Supervisor(("Supervisor"))
    Worker(("Worker"))
    AI(("AI Supervisor<br/>(System)"))

    Owner --> UC1
    Owner --> UC2
    Owner --> UC3
    Owner --> UC4
    Owner --> UC5
    Owner --> UC6
    Owner --> UC7
    Owner --> UC8

    Supervisor --> UC4
    Supervisor --> UC9
    Supervisor --> UC10
    Supervisor --> UC11
    Supervisor --> UC12
    Supervisor --> UC7

    Worker --> UC4
    Worker --> UC13
    Worker --> UC14
    Worker --> UC15
    Worker --> UC16
    Worker --> UC17

    AI --> UC18
    AI --> UC19
    AI --> UC20
    AI --> UC21
    AI --> UC22
    AI --> UC23
    AI --> UC24

    UC5 -.->|includes| UC18
    UC6 -.->|includes| UC18
    UC5 -.->|triggers| UC19
    UC6 -.->|triggers| UC19
    UC19 -.->|includes| UC20
    UC20 -.->|includes| UC21
    UC21 -.->|includes| UC22
    UC22 -.->|includes| UC23
    UC23 -.->|extends| UC13
    UC16 -.->|triggers| UC24
    UC17 -.->|triggers| UC24
```

## Use Cases by Actor

### Owner

| Use Case | Description |
|----------|-------------|
| Create Organization | Set up a new business organization in the system |
| Add Users & Assign Roles | Onboard supervisors and workers with role assignment |
| Configure Preferred Languages | Set language preferences for each user |
| Record / Upload Voice Instruction | Submit tasks via microphone or audio file |
| Monitor Task Progress | View pending, in-progress, and completed tasks |
| View Reports | Access workforce and task completion reports |

### Supervisor

| Use Case | Description |
|----------|-------------|
| Review Tasks | Inspect AI-generated and assigned tasks |
| Reassign Tasks | Manually reassign tasks to different workers |
| Monitor Workers | Track worker availability and workload |
| Handle Escalations | Resolve delayed or blocked tasks |

### Worker

| Use Case | Description |
|----------|-------------|
| Receive Translated Tasks | Get task assignments in preferred language |
| Listen to Voice Instructions | Play TTS-generated audio instructions |
| Accept Task | Acknowledge and start working on a task |
| Send Voice Update | Report progress via voice message |
| Mark Task Completed | Signal task completion |

### AI Supervisor (System)

| Use Case | Description |
|----------|-------------|
| Process Speech to Text | Convert owner/worker audio to structured text |
| Understand Intent | Extract actions and priorities from instructions |
| Plan & Prioritize Tasks | Break complex instructions into sub-tasks |
| Assign Workers | Match tasks to available workers by skill and language |
| Translate Instructions | Convert task text to each worker's language |
| Generate Voice Instructions | Produce TTS audio for workers |
| Track Progress | Monitor delays and push real-time dashboard updates |

## Portal-to-Use-Case Mapping

```mermaid
flowchart LR
    subgraph OwnerPortal["Owner Portal"]
        O1["Login"]
        O2["Dashboard"]
        O3["Voice Command Screen"]
        O4["User Management"]
        O5["Reports"]
    end

    subgraph SupervisorPortal["Supervisor Portal"]
        S1["Dashboard"]
        S2["Worker Monitoring"]
        S3["Task Assignment"]
        S4["Escalations"]
    end

    subgraph WorkerPortal["Worker Portal"]
        W1["My Tasks"]
        W2["Voice Inbox"]
        W3["Task Updates"]
        W4["Profile Settings"]
    end

    O3 --> UC5
    O3 --> UC6
    O2 --> UC7
    O5 --> UC8
    O4 --> UC2

    S3 --> UC10
    S2 --> UC11
    S4 --> UC12
    S1 --> UC9

    W1 --> UC15
    W2 --> UC14
    W3 --> UC16
    W3 --> UC17
    W4 --> UC3
```
