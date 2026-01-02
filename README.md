# Google AntiGravity IDE - Reverse Engineering Documentation

> **Comprehensive analysis of Google DeepMind's AI-powered agentic coding IDE**  
> Analysis performed: January 2026

---

## Overview

**AntiGravity** is an AI-powered agentic coding IDE developed by Google DeepMind, publicly released in November 2025. It represents a significant advancement in AI-assisted software development, featuring autonomous task execution, multi-modal understanding, and deep IDE integration.

### Internal Codenames

| Codename    | Component                                      |
|-------------|------------------------------------------------|
| **Jetski**  | Overall project codename                       |
| **Cortex**  | Core agent engine / reasoning system           |
| **Cascade** | Planning and task decomposition pipeline       |

---

## Quick Start

Navigate directly to specific documentation:

1. [Core Identity & Role](01_IDENTITY_AND_ROLE.md) — Who is the agent?
2. [Agent Modes](02_AGENT_MODES.md) — PLANNING / EXECUTION / VERIFICATION
3. [Communication Guidelines](03_COMMUNICATION.md) — User interaction style
4. [Tools Reference](04_TOOLS.md) — Complete tool catalog
5. [Knowledge System](05_KNOWLEDGE_SYSTEM.md) — Knowledge Items (KI)
6. [Browser Automation](06_BROWSER_AUTOMATION.md) — Web interaction subagent
7. [Task Management](07_TASK_MANAGEMENT.md) — Task lifecycle & artifacts
8. [Technical Details](08_TECHNICAL_DETAILS.md) — Protobufs, APIs, architecture
9. [Behavioral Rules](09_BEHAVIORAL_RULES.md) — Critical reminders & edge cases
10. [**Complete Workflow**](10_COMPLETE_WORKFLOW.md) — **Full workflow diagram & state machine**
11. [Additional Systems](11_ADDITIONAL_SYSTEMS.md) — Artifacts, MCP, Git, Deployments, etc.
12. [**Supplementary Extraction**](12_SUPPLEMENTARY_EXTRACTION.md) — **Models, parameters, ConfidenceScore, ephemeral messages**

---

## Key Discoveries Summary

| Component                      | Status | Details                              |
|--------------------------------|--------|--------------------------------------|
| Core Identity Prompt           | ✅     | Full system prompt extracted         |
| Agent Modes                    | ✅     | PLANNING, EXECUTION, VERIFICATION    |
| Cortex Step Types              | ✅     | 100+ distinct step types identified      |
| Tools with JSON Schemas        | ✅     | 43+ tools fully documented           |
| Communication Style Guidelines | ✅     | Complete style guide recovered       |
| Knowledge Item System          | ✅     | KI creation, retrieval, lifecycle    |
| Browser Subagent System        | ✅     | Playwright-based automation layer    |
| Checkpoint/Context System      | ✅     | Token limit handling, model routing  |
| Behavioral Rules               | ✅     | Critical reminders, edge cases       |
| Protobuf Schemas               | ✅     | Core message types extracted         |
| Model Parameters               | ✅     | Temperature, sampling, token limits  |
| ConfidenceScore System         | ✅     | Complete 6-question grading system   |
| Ephemeral Messages             | ✅     | Full structure and task reminders    |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           AntiGravity IDE                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌──────────┐    ┌───────────────┐    ┌─────────────────┐             │
│   │          │    │               │    │                 │             │
│   │   User   │───▶│   Chat UI     │───▶│   Extension     │             │
│   │          │    │   (Webview)   │    │   (VSCode)      │             │
│   └──────────┘    └───────────────┘    └────────┬────────┘             │
│                                                  │                      │
│                                                  ▼                      │
│                                        ┌─────────────────┐             │
│                                        │                 │             │
│                                        │ Language Server │             │
│                                        │     (LSP)       │             │
│                                        │                 │             │
│                                        └────────┬────────┘             │
│                                                  │                      │
└──────────────────────────────────────────────────┼──────────────────────┘
                                                   │
                                                   ▼
                                          ┌───────────────┐
                                          │               │
                                          │   Cloud API   │
                                          │   (Vertex)    │
                                          │               │
                                          └───────┬───────┘
                                                  │
                                                  ▼
                                          ┌───────────────┐
                                          │               │
                                          │ Gemini Model  │
                                          │   + Cortex    │
                                          │               │
                                          └───────────────┘
```

---

## Documentation Files

| File                                                  | Description                                        |
|-------------------------------------------------------|----------------------------------------------------|
| [`01_IDENTITY_AND_ROLE.md`](01_IDENTITY_AND_ROLE.md)  | Core identity prompt, agent persona, capabilities  |
| [`02_AGENT_MODES.md`](02_AGENT_MODES.md)              | PLANNING, EXECUTION, VERIFICATION mode behaviors   |
| [`03_COMMUNICATION.md`](03_COMMUNICATION.md)          | User interaction style, tone, formatting rules     |
| [`04_TOOLS.md`](04_TOOLS.md)                          | All 43+ tools with JSON schemas and examples       |
| [`05_KNOWLEDGE_SYSTEM.md`](05_KNOWLEDGE_SYSTEM.md)    | Knowledge Items (KI) creation, storage, retrieval  |
| [`06_BROWSER_AUTOMATION.md`](06_BROWSER_AUTOMATION.md)| Browser subagent, Playwright integration, commands |
| [`07_TASK_MANAGEMENT.md`](07_TASK_MANAGEMENT.md)      | Task boundaries, artifacts, checkpoints, rollback  |
| [`08_TECHNICAL_DETAILS.md`](08_TECHNICAL_DETAILS.md)  | Protobuf schemas, API endpoints, architecture      |
| [`09_BEHAVIORAL_RULES.md`](09_BEHAVIORAL_RULES.md)    | Critical reminders, edge cases, behavioral rules   |
| [`10_COMPLETE_WORKFLOW.md`](10_COMPLETE_WORKFLOW.md)  | **Complete workflow, state machine, step types**   |
| [`11_ADDITIONAL_SYSTEMS.md`](11_ADDITIONAL_SYSTEMS.md)| Artifacts, MCP, Git, Deployments, Rate Limits      |
| [`12_SUPPLEMENTARY_EXTRACTION.md`](12_SUPPLEMENTARY_EXTRACTION.md)| **Models, sampling params, ConfidenceScore, ephemeral** |

---

## Extraction Methodology

The following techniques were used to extract system prompts and internal documentation:

| Method                        | Target                          | Yield                          |
|-------------------------------|---------------------------------|--------------------------------|
| **String extraction**         | Extension binary (VSIX)         | Tool schemas, static prompts   |
| **Memory dump analysis**      | Running extension process       | Runtime prompts, mode configs  |
| **Artifact file inspection**  | `.antigravity/` workspace dir   | Task state, KI serialization   |
| **Network traffic capture**   | Extension ↔ Cloud API           | Request/response structures    |
| **Protobuf reconstruction**   | Serialized message payloads     | Schema definitions             |

---

## Completeness Assessment

```
Overall Extraction:  ████████████████████░  ~99%
```

| Category               | Coverage | Notes                                    |
|------------------------|----------|------------------------------------------|
| System Prompts         | ~98%     | Core prompts fully recovered             |
| Tool Definitions       | ~98%     | All known tools documented               |
| Mode Configurations    | ~95%     | Dynamic switching logic documented       |
| Protobuf Schemas       | ~85%     | Good reconstruction from payloads        |
| Model Parameters       | ~95%     | Temperature, sampling, token limits      |
| Runtime Behavior       | ~90%     | Observed patterns, ephemeral messages    |

### Known Gaps

- Dynamic prompt injection based on workspace context
- Exact model fine-tuning parameters
- Server-side Cascade pipeline implementation details
- Rate limiting and quota management internals

---

## Legal Notice

This documentation is produced for educational and research purposes only. All trademarks, codenames, and product references belong to their respective owners. This analysis was conducted on publicly available software through standard reverse engineering techniques.

---

## Contributing

Found additional details or corrections? Documentation improvements are welcome.

---

*Last updated: January 2, 2026*
