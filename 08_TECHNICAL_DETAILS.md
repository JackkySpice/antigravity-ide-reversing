# Technical Architecture

## System Components

| Component | Path | Purpose |
|-----------|------|---------|
| Language Server | `/usr/share/antigravity/resources/app/extensions/antigravity/bin/language_server_linux_arm` | Core agent logic (Go binary) |
| Jetski Agent | `/usr/share/antigravity/resources/app/out/jetskiAgent/main.js` | Agent orchestration (JS) |
| Extension | `/usr/share/antigravity/resources/app/extensions/antigravity/dist/extension.js` | VSCode integration |
| Chat UI | `/usr/share/antigravity/resources/app/extensions/antigravity/out/media/chat.js` | User interface |

---

## Internal Codenames

| Name | Purpose |
|------|---------|
| Jetski | Project codename |
| Cortex | Agent execution engine |
| Cascade | Planning/execution pipeline |

---

## Cloud API

| Property | Value |
|----------|-------|
| Endpoint | `https://daily-cloudcode-pa.googleapis.com` |
| Protocol | gRPC with Protocol Buffers |
| Auth | OAuth via Google Account |

---

## Local Storage

| Path | Content |
|------|---------|
| `~/.gemini/antigravity/conversations/*.pb` | Encrypted conversation data |
| `~/.gemini/antigravity/brain/` | Artifacts by conversation |
| `~/.gemini/antigravity/knowledge/` | Knowledge Items |
| `~/.config/Antigravity/User/globalStorage/state.vscdb` | SQLite state database |

---

## Protobuf Schemas

### PromptSection

```protobuf
message PromptSection {
  string title = 1;
  string content = 2;
  repeated Criteria criteria = 3;
  PromptSectionMetadata metadata = 4;
  string dynamic_content = 5;
}
```

### CustomAgentConfig

```protobuf
message CustomAgentConfig {
  repeated PromptSection system_prompt_sections = 1;
  repeated string tool_names = 2;
  repeated McpServerInfo mcp_servers = 3;
  string mcp_cwd_workspace_relative_path = 4;
  HooksConfig hooks_config = 5;
}
```

### CascadeConfig

```protobuf
message CascadeConfig {
  CascadePlannerConfig planner_config = 1;
  CheckpointConfig checkpoint_config = 2;
  CascadeExecutorConfig executor_config = 3;
  TrajectoryConversionConfig trajectory_conversion_config = 4;
  bool apply_model_default_override = 6;
  ConversationHistoryConfig conversation_history_config = 7;
  bool split_dynamic_prompt_sections = 8;
}
```

### CortexStep

```protobuf
message CortexStep {
  CortexStepType type = 1;
  // Payload fields vary by type
  CortexStepGeneratorMetadata generator_metadata = N;
}
```

### Artifact Metadata

```json
{
  "artifactType": "ARTIFACT_TYPE_IMPLEMENTATION_PLAN",
  "summary": "Implementation plan for...",
  "updatedAt": "2026-01-01T10:34:10.110987922Z"
}
```

**Artifact Types:**

- `ARTIFACT_TYPE_IMPLEMENTATION_PLAN`
- `ARTIFACT_TYPE_TASK`
- `ARTIFACT_TYPE_WALKTHROUGH`

---

## Template System

Templates use Go `text/template` syntax.

### Template Files

| Template Path | Purpose |
|---------------|---------|
| `system_prompts/identity.tmpl` | Core identity |
| `system_prompts/agentic_mode_overview.tmpl` | Agentic mode description |
| `system_prompts/tool_calling.tmpl` | Tool calling format |
| `system_prompts/communication_style.tmpl` | Communication rules |
| `system_prompts/artifact_formatting_guidelines.tmpl` | Artifact format |
| `system_prompts/task_artifact.tmpl` | Task management |
| `system_prompts/implementation_plan_artifact.tmpl` | Planning |
| `system_prompts/knowledge_discovery.tmpl` | KI system |
| `system_prompts/mode_descriptions.tmpl` | Mode details |
| `system_prompts/conversation_logs.tmpl` | Conversation history |
| `system_prompts/ephemeral_message.tmpl` | Ephemeral system messages |
| `system_prompts/persistent_context.tmpl` | Persistent context |
| `system_prompts/task_boundary_tool.tmpl` | Task boundaries |
| `system_prompts/notify_user_tool.tmpl` | User notifications |
| `system_prompts/file_diffs_artifact.tmpl` | File diff formatting |
| `system_prompts/file_diffs_artifact_dynamic.tmpl` | Dynamic diffs |
| `system_prompts/walkthrough_artifact.tmpl` | Walkthrough format |
| `system_prompts/agentic_mode_overview_dynamic.tmpl` | Dynamic agentic mode |
| `ephemeral_prompts/active_task_reminder.tmpl` | Task reminders |
| `step_strings/run_command.tmpl` | Command formatting |
| `step_strings/send_command_input.tmpl` | Command input |
| `helpers/errors.tmpl` | Error formatting |

### Template Variables

```
{{ArtifactDirectoryPath}}           - Path to artifacts
{{KnowledgeDirectoryPath}}          - Path to KIs
{{SystemGeneratedLogsPath "conv"}}  - Path to logs
{{FileDiffsPath}}                   - Path to diffs
{{MetadataFileName}}                - metadata.yaml
```

---

## Checkpoint System (Context Overflow Handling)

When context exceeds token limits, the system uses a checkpoint mechanism:

### Checkpoint Configuration

```json
{
  "checkpoint_config": {
    "max_token_limit": "128000",
    "token_threshold": "50000",
    "max_overhead_ratio": "0.15",
    "moving_window_size": "1",
    "enabled": true,
    "checkpoint_model": "MODEL_GOOGLE_GEMINI_2_5_FLASH"
  }
}
```

### Config Variations by Model/Tier

| Tier | Max Token Limit | Checkpoint Model |
|------|-----------------|------------------|
| Enterprise | 135,000 | MODEL_CLAUDE_3_5_HAIKU_20241022 |
| SaaS | 45,000 | (default) |
| Gemini 2.5 Pro | 160,000 | MODEL_GOOGLE_GEMINI_2_5_FLASH |
| Gemini Flash | 128,000 | MODEL_GOOGLE_GEMINI_2_5_FLASH_LITE |

### Checkpoint Behavior

- `max_steps_per_checkpoint`: 1-3 (varies by trajectory type)
- Checkpoints skip: "checkpoint, memory, or ephemeral message" steps
- Truncation: `no checkpoint found, truncating conversation to step index %d`

### Trajectory Types

```
CORTEX_TRAJECTORY_TYPE_CASCADE
CORTEX_TRAJECTORY_TYPE_CHECKPOINT
CORTEX_TRAJECTORY_TYPE_USER_MAINLINE
```

---

## Model Routing System

### Available Models

| Model ID | Display Name | Purpose |
|----------|--------------|---------|
| MODEL_GOOGLE_GEMINI_2_5_PRO | Gemini 2.5 Pro | Main agent model |
| MODEL_GOOGLE_GEMINI_2_5_FLASH | Gemini 2.5 Flash | Fast responses |
| MODEL_GOOGLE_GEMINI_2_5_FLASH_LITE | Gemini 2.5 Flash Lite | Lightweight |
| MODEL_GOOGLE_GEMINI_2_5_FLASH_THINKING | Gemini 2.5 Flash (Thinking) | Extended reasoning |
| MODEL_GOOGLE_GEMINI_3_PRO_HIGH | Gemini 3 Pro (High) | High capability |
| MODEL_GOOGLE_GEMINI_3_PRO_LOW | Gemini 3 Pro (Low) | Lower latency |
| MODEL_GOOGLE_GEMINI_3_FLASH | Gemini 3 Flash | Fast G3 |
| MODEL_GOOGLE_GEMINI_3_PRO_IMAGE | Gemini 3 Pro Image | Image generation |
| MODEL_GOOGLE_GEMINI_COMPUTER_USE_EXPERIMENTAL | Computer Use | Browser automation |
| MODEL_GOOGLE_GEMINI_INTERNAL_BYOM | Internal BYOM | Bring your own model |

### Internal/Experimental Models

```
MODEL_GOOGLE_GEMINI_2_5_PRO_EVAL
MODEL_GOOGLE_GEMINI_GENTLEISLAND
MODEL_GOOGLE_GEMINI_INFINITYBLOOM
MODEL_GOOGLE_GEMINI_HORIZONDAWN
MODEL_GOOGLE_GEMINI_RIFTRUNNER
MODEL_GOOGLE_GEMINI_RIFTRUNNER_THINKING_LOW
MODEL_GOOGLE_GEMINI_RIFTRUNNER_THINKING_HIGH
MODEL_GOOGLE_GEMINI_TRAINING_POLICY
MODEL_GOOGLE_GEMINI_INTERNAL_TAB_FLASH_LITE
MODEL_GOOGLE_GEMINI_INTERNAL_TAB_JUMP_FLASH_LITE
```

### Model Selection Hierarchy

```json
{
  "brainModel": "MODEL_GOOGLE_GEMINI_2_5_PRO",
  "judge_model": "MODEL_GOOGLE_GEMINI_2_5_PRO",
  "checkpoint_model": "MODEL_GOOGLE_GEMINI_2_5_FLASH",
  "defaultAgentModelId": "gemini-3-pro-high"
}
```

### Planner Configuration

```json
{
  "planner_config": {
    "max_output_tokens": "32768",
    "truncation_threshold_tokens": "100000"
  }
}
```

---

## Internal Package Structure

```
google3/third_party/jetski/
├── cortex/
│   ├── artifacts/
│   ├── tools/
│   ├── utils/
│   ├── mixins/
│   └── shared/
├── cortex_pb/cortex.proto
├── prompt/template_provider/templates/
└── telemetry/extensions/
```

---

## Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│                        User Input                           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     Chat UI (chat.js)                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  Extension (extension.js)                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                Language Server (Go binary)                  │
└─────────────────────────────────────────────────────────────┘
                              │ gRPC
                              ▼
┌─────────────────────────────────────────────────────────────┐
│          Cloud API (daily-cloudcode-pa.googleapis.com)      │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       Gemini Model                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    CortexStep Response                      │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Step Execution                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       User Output                           │
└─────────────────────────────────────────────────────────────┘
```
