# Project Diagrams

Documentation diagrams for the **Multilingual Voice-Based Workforce Management Agent** (MVWMA), derived from [plan.md](../plan.md).

## Diagram Index

| Diagram | File | Description |
|---------|------|-------------|
| Architecture | [architecture-diagram.md](./architecture-diagram.md) | System layers, tech stack, and component interactions |
| Use Case | [use-case-diagram.md](./use-case-diagram.md) | Actors, responsibilities, and system use cases |
| Sequence | [sequence-diagram.md](./sequence-diagram.md) | End-to-end voice instruction and supporting flows |
| Activity | [activity-diagram.md](./activity-diagram.md) | Business process workflows |
| State Machine | [state-machine-diagram.md](./state-machine-diagram.md) | Task lifecycle and LangGraph agent states |

## How to View

These diagrams use [Mermaid](https://mermaid.js.org/) syntax. You can render them in:

- **GitHub / GitLab** — Mermaid renders automatically in Markdown preview
- **VS Code / Cursor** — Install the "Markdown Preview Mermaid Support" extension
- **Mermaid Live Editor** — Paste diagram blocks at [mermaid.live](https://mermaid.live)

## Project Summary

MVWMA is an Agentic AI system that acts as a multilingual supervisor between business owners and workers. Owners give voice instructions in their native language; the AI breaks instructions into tasks, assigns workers by skill and language, translates instructions, and tracks completion in real time.
