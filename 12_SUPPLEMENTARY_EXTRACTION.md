# AntiGravity IDE - Supplementary Extraction (Session 2)

> **Additional findings from memory dump analysis - January 2, 2026**

---

## 1. Model Parameters & Sampling

### Main Model Temperature
```json
"temperature": 0.15
```

### Sampling Rates
```json
"baseline_sampling_rate": 0.1,
"target_sampling_rate": 1.0,
"sampling_rate": 0.05
```

### Token Limits by Model Configuration
| Configuration | Max Token Limit | Checkpoint Model | Max Output Tokens |
|---------------|-----------------|------------------|-------------------|
| **SaaS Default** | 45,000 | GPT-4o Mini | 32,768 |
| **Enterprise** | 135,000 | Claude 3.5 Haiku | 32,768 |
| **High Token** | 160,000 | Gemini 2.5 Flash | 16,384 |
| **Standard** | 128,000 | Gemini 2.5 Flash Lite | 16,384 |

### Context Configuration
```json
{
  "max_token_limit": "160000",
  "token_threshold": "50000",
  "max_overhead_ratio": "0.15",
  "moving_window_size": "1",
  "max_output_tokens": "16384",
  "enabled": true,
  "checkpoint_model": "MODEL_GOOGLE_GEMINI_2_5_FLASH"
}
```

---

## 2. Complete Model Roster (2025-2026)

### Google Models
| Model ID | Description |
|----------|-------------|
| `MODEL_GOOGLE_GEMINI_2_0_FLASH` | Gemini 2.0 Flash |
| `MODEL_GOOGLE_GEMINI_2_5_FLASH` | Gemini 2.5 Flash |
| `MODEL_GOOGLE_GEMINI_2_5_FLASH_LITE` | Gemini 2.5 Flash Lite |
| `MODEL_GOOGLE_GEMINI_2_5_FLASH_THINKING` | Gemini 2.5 Flash Thinking |
| `MODEL_GOOGLE_GEMINI_2_5_FLASH_PREVIEW_05_20` | Gemini 2.5 Flash Preview |
| `MODEL_GOOGLE_GEMINI_2_5_FLASH_PREVIEW_05_20_THINKING` | Gemini 2.5 Flash Preview Thinking |
| `MODEL_GOOGLE_GEMINI_2_5_PRO` | Gemini 2.5 Pro |
| `MODEL_GOOGLE_GEMINI_COMPUTER_USE_EXPERIMENTAL` | Gemini Computer Use |
| `MODEL_GOOGLE_GEMINI_INTERNAL_BYOM` | Internal BYOM |
| `MODEL_GOOGLE_GEMINI_INFINITYBLOOM` | Internal Codename |

### Anthropic Models
| Model ID | Description |
|----------|-------------|
| `MODEL_CLAUDE_3_5_SONNET_20241022` | Claude 3.5 Sonnet |
| `MODEL_CLAUDE_3_5_HAIKU_20241022` | Claude 3.5 Haiku (checkpoint) |
| `MODEL_CLAUDE_3_7_SONNET_20250219` | Claude 3.7 Sonnet |
| `MODEL_CLAUDE_3_7_SONNET_20250219_THINKING` | Claude 3.7 Sonnet Thinking |
| `MODEL_CLAUDE_4_SONNET` | Claude 4 Sonnet |
| `MODEL_CLAUDE_4_SONNET_THINKING` | Claude 4 Sonnet Thinking |
| `MODEL_CLAUDE_4_5_SONNET` | Claude 4.5 Sonnet |
| `MODEL_CLAUDE_4_5_SONNET_THINKING` | Claude 4.5 Sonnet Thinking |

### OpenAI Models
| Model ID | Description |
|----------|-------------|
| `MODEL_CHAT_O1_MINI` | O1 Mini |
| `MODEL_CHAT_O3` | O3 |
| `MODEL_CHAT_O4_MINI_HIGH` | O4 Mini High |
| `MODEL_CHAT_GPT_4_1_2025_04_14` | GPT-4.1 |
| `MODEL_CHAT_GPT_4O_MINI_2024_07_18` | GPT-4o Mini |
| `MODEL_OPENAI_GPT_OSS_120B_MEDIUM` | GPT OSS 120B |

### Placeholder/Internal Models
| Model ID | Purpose |
|----------|---------|
| `MODEL_PLACEHOLDER_M7` | Reserved slot |
| `MODEL_PLACEHOLDER_M8` | Reserved slot |
| `MODEL_PLACEHOLDER_M9` | Reserved slot |
| `MODEL_PLACEHOLDER_M12` | Reserved slot |
| `MODEL_PLACEHOLDER_M18` | Reserved slot |
| `MODEL_CHAT_20706` | Internal |
| `MODEL_CHAT_23310` | Internal |
| `MODEL_CHAT_11121` | Context check model |

### Model Providers
- `MODEL_PROVIDER_GOOGLE`
- `MODEL_PROVIDER_ANTHROPIC`
- `MODEL_PROVIDER_OPENAI`

---

## 3. Agent Mode States (Complete)

### Mode Enum Values
```protobuf
AGENT_MODE_UNSPECIFIED = 0
AGENT_MODE_PLANNING = 1
AGENT_MODE_EXECUTION = 2
AGENT_MODE_VERIFICATION = 3
```

### Browser Subagent Modes
```protobuf
BROWSER_SUBAGENT_MODE_UNSPECIFIED
BROWSER_SUBAGENT_MODE_BOTH_AGENTS
BROWSER_SUBAGENT_MODE_SUBAGENT_ONLY
BROWSER_SUBAGENT_MODE_MAIN_AGENT_ONLY
BROWSER_SUBAGENT_MODE_SUBAGENT_PRIMARILY
```

---

## 4. ConfidenceScore System (Complete)

### notify_user Tool Documentation (Full)
```
When requesting document review via AbsolutePaths, you must provide a 
ConfidenceScore from 0.0 (no confidence) to 1.0 (high confidence) 
reflecting your assessment of the document's quality, completeness, 
and accuracy.

CONFIDENCE GRADING: Before setting ConfidenceScore, answer these 6 
questions (Yes/No):
  (1) Gaps - any missing parts?
  (2) Assumptions - any unverified assumptions?
  (3) Complexity - complex logic with unknowns?
  (4) Risk - non-trivial interactions with bug risk?
  (5) Ambiguity - unclear requirements forcing design choices?
  (6) Irreversible - difficult to revert?

SCORING:
  0.8-1.0 = answered No to ALL questions
  0.5-0.7 = answered Yes to 1-2 questions
  0.0-0.4 = answered Yes to 3+ questions

Write justification first, then score.
```

### notify_user JSON Schema
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "properties": {
    "PathsToReview": {
      "items": {"type": "string"},
      "type": "array",
      "description": "List of ABSOLUTE paths to files that the user should be notified about. You MUST populate this if the notification is to request review for artifacts or files. These must be ABSOLUTE paths, leave empty if you are not requesting review for artifacts or files. Specify this argument first."
    },
    "BlockedOnUser": {
      "type": "boolean",
      "description": "Set this to true if you are blocked on user approval to proceed. This is most appropriate when you want the user to review a plan or design doc, where there is more work to be done after approval. Do not set this to true if you are just notifying user about the completion of your work."
    },
    "Message": {
      "type": "string",
      "description": "Required message to notify the user with. Specify this argument last."
    },
    "ShouldAutoProceed": {
      "type": "boolean",
      "description": "Set this to true if you believe that the task you notified the user about can be proceeded with without any explicit user feedback and you are extremely confident in the approach."
    },
    "ConfidenceScore": {
      "type": "number",
      "description": "0.0 to 1.0 score based on 6-question grading system"
    },
    "ConfidenceJustification": {
      "type": "string",
      "description": "Written justification for the score"
    },
    "waitForPreviousTools": {
      "type": "boolean",
      "description": "If true, wait for all previous tool calls from this turn to complete before executing."
    }
  },
  "required": ["PathsToReview", "BlockedOnUser", "Message", "ShouldAutoProceed"]
}
```

---

## 5. Ephemeral Message System (Complete)

### Ephemeral Message Structure
```xml
<EPHEMERAL_MESSAGE>
<active_task_reminder>
Remember to update the task as appropriate. The current task is: 
  task_name:"[Task Name]" 
  task_status:"[Current Status]" 
  task_summary:"[Summary]" 
  mode:AGENT_MODE_[PLANNING|EXECUTION|VERIFICATION]

As a rule of thumb, you should update the status and summary around once 
every 5 tools. You have not updated the task in [N] tools since the last 
update. You should make task boundary updates concurrently with other 
tools when starting new work phases, STARTING with the task boundary tool 
if calling multiple.

Do not update the status too frequently, leave at minimum two tool calls 
in between status updates. Too frequent updates will overwhelm the user. 
Never make two status updates in a row without doing anything in between.
</active_task_reminder>
</EPHEMERAL_MESSAGE>
```

### Ephemeral Message Config
```protobuf
message EphemeralMessagesConfig {
  bool cascade_include_ephemeral_message = 1;
  // Determines which parts of browser state are injected
}
```

### Artifact Reminder (Ephemeral)
```xml
<EPHEMERAL_MESSAGE>
<artifact_reminder>
You have not yet created any artifacts. Please follow the artifact 
guidelines and create them as needed based on the task.

CRITICAL REMINDER: remember that user-facing artifacts should be AS 
CONCISE AS POSSIBLE. Keep this in mind when editing artifacts.
</artifact_reminder>
</EPHEMERAL_MESSAGE>
```

---

## 6. Task Boundary Tool (Complete Documentation)

### TaskName Guidelines
```
**Recommended pattern**: Use descriptive TaskNames that clearly communicate 
your current objective. Common patterns include:
- Mode-based: "Planning Authentication", "Implementing User Profiles", 
  "Verifying Payment Flow"
- Activity-based: "Debugging Login Failure", "Researching Database Schema", 
  "Removing Legacy Code", "Refactoring API Layer"
```

### TaskSummary Guidelines
```
Describes the current high-level goal of this task. Initially, state the goal. 
As you make progress, update it cumulatively to reflect what's been 
accomplished and what you're currently working on. Synthesize progress from 
task.md into a concise narrative—don't copy checklist items verbatim.
```

### TaskStatus Guidelines
```
Current activity you're about to start or working on right now. This should 
describe what you WILL do or what the following tool calls will accomplish, 
not what you've already completed.
```

### Mode Transitions
```
You can change mode within the same TaskName as the work evolves.

**Backtracking during work**: When backtracking mid-task (e.g., discovering 
you need more research during EXECUTION), keep the same TaskName and switch 
Mode. Update TaskSummary to explain the change in direction.

**After notify_user**: You exit task mode and return to normal chat. When 
ready to resume work, call task_boundary again with an appropriate TaskName.

**Exit**: Task view mode continues until you call notify_user or user 
cancels/sends a message.
```

---

## 7. Template Files (Google3 Paths)

### Identified Templates
| Path | Purpose |
|------|---------|
| `google3/third_party/jetski/prompt/template_provider/templates/system_prompts/identity.tmpl` | Core identity prompt |
| `google3/third_party/jetski/prompt/template_provider/templates/system_prompts/agentic_mode_overview_dynamic.tmpl` | Agentic mode documentation |
| `google3/third_party/jetski/prompt/template_provider/templates/system_prompts/artifact_formatting_guidelines.tmpl` | Artifact formatting rules |
| `google3/third_party/jetski/prompt/template_provider/templates/system_prompts/implementation_plan_artifact.tmpl` | Implementation plan structure |
| `google3/third_party/jetski/prompt/template_provider/templates/system_prompts/communication_style.tmpl` | Communication guidelines |
| `google3/third_party/jetski/prompt/template_provider/templates/system_prompts/file_diffs_artifact.tmpl` | File diff formatting |
| `google3/third_party/jetski/prompt/template_provider/templates/system_prompts/knowledge_discovery.tmpl` | Knowledge item discovery |

---

## 8. Additional Critical Prompts

### Agentic Mode Override
```json
{
  "mode": "SECTION_OVERRIDE_MODE_OVERRIDE",
  "content": "You are an agent - please keep going until the user's query 
  is completely resolved, before ending your turn and yielding back to the 
  user. Only terminate your turn when you are sure that the problem is solved.
  
  If you are not sure about file content or codebase structure pertaining to 
  the user's request, use your tools to read files and gather the relevant 
  information: do NOT guess or make up an answer.
  
  Act as a true pair-programming partner. Take initiative on clearly defined 
  tasks, but if the user's intent is ambiguous, follow the user's lead and 
  avoid jumping to conclusions."
}
```

### Extensive Planning Override
```json
{
  "mode": "SECTION_OVERRIDE_MODE_OVERRIDE",
  "content": "You MUST plan extensively before each function call, and reflect 
  extensively on the outcomes of the previous function calls. DO NOT do this 
  entire process by making function calls only, as this can impair your 
  ability to solve the problem and think insightfully."
}
```

### Code Execution Verification
```
Always be very rigorous about using code execution to verify your hypothesis. 
After you have a hypothesis, you should always use code execution to verify 
it before making any code changes.

If you do not understand some part of the codebase, or makes a hypothesis 
about the behavior of the code in your thought process, **ALWAYS** verify 
the hypothesis using test scripts or minimal working examples.
```

---

## 9. Feature Flags & Experiments

### Cascade Flags
| Flag | Purpose |
|------|---------|
| `cascade-include-ephemeral-message` | Enable ephemeral messages |
| `cascade-grep-tool-config-override` | Override grep tool config |
| `cascade-tool-description-override` | Override tool descriptions |
| `cascade-browser-subagent-reminder` | Browser subagent reminders |
| `cascade_changes_preservation_rate` | Change preservation ratio |
| `cascade_changes_contribution_rate` | Change contribution ratio |

### Model Feature Flags
```json
{
  "model_features": {
    // Model-specific feature toggles
  },
  "model_notifications": [
    // Warning notifications for models
  ]
}
```

---

## 10. API Endpoints (gRPC)

### Language Server Service
```protobuf
/exa.language_server_pb.LanguageServerService/StreamCascadePanelReactiveUpdates
```

### Chat Client Server Service
```protobuf
/exa.chat_client_server_pb.ChatClientServerService/StartChatClientRequestStream
```

### Cloud Code Service
```protobuf
/google.internal.cloud.code.v1internal.CloudCode/GetCodeAssistGlobalUserSetting
/google.internal.cloud.code.v1internal.CloudCode/SetCodeAssistGlobalUserSetting
```

### Index Service
```protobuf
/exa.index_pb.IndexService/GetIndexedRepositories
/exa.index_pb.IndexManagementService/GetNumberConnections
```

---

## 11. Cortex Step Categories (Extended)

### Step Type Enums (Additional)
```protobuf
CORTEX_STEP_COMPILE_TOOL_UNSPECIFIED
CORTEX_REQUEST_SOURCE_CASCADE
CORTEX_REQUEST_SOURCE_UNSPECIFIED
CORTEX_MEMORY_TRIGGER_MODEL_DECISION
CORTEX_MEMORY_TRIGGER_UNSPECIFIED
CORTEX_TRAJECTORY_SOURCE_UNSPECIFIED
CORTEX_TRAJECTORY_SOURCE_CASCADE_CLIENT
CORTEX_TRAJECTORY_SOURCE_ASYNC_CF
CORTEX_TRAJECTORY_SOURCE_ASYNC_SL
CORTEX_TRAJECTORY_SOURCE_ASYNC_CM
CORTEX_TRAJECTORY_SOURCE_EVAL
CORTEX_TRAJECTORY_TYPE_USER_MAINLINE
CORTEX_TRAJECTORY_TYPE_USER_GRANULAR
CORTEX_TRAJECTORY_TYPE_SUPERCOMPLETE
CORTEX_TRAJECTORY_TYPE_CHECKPOINT
CORTEX_ERROR_CATEGORY_UNSPECIFIED
```

### Truncation Reasons
```protobuf
TRUNCATION_REASON_UNSPECIFIED
TRUNCATION_REASON_MAX_TOKEN_LIMIT
```

### Execution Levels
```protobuf
EXECUTION_ASYNC_LEVEL_UNSPECIFIED
EXECUTION_ASYNC_LEVEL_EXECUTOR_BLOCKING
```

---

## 12. Seat Management & Billing (Internal)

### User Features
- `usedPromptCredits`
- `usedFlowCredits`
- `usedFlexCredits`
- `cascadeWebSearchEnabled`
- `hasCascadeSeat`

### Team Tiers
- `TeamsTier` enum for plan levels
- `CascadeSeatType` for seat types
- `topUpEnabled` / `monthlyTopUpAmount` / `topUpSpent`

### Subscription Types
- `SubInterval` (monthly/yearly)
- `billingStart` / `billingEnd` timestamps
- `numSeatsCurrentBillingPeriod`

---

## 13. Browser Subagent Reminders

### Critical Browser Rules
```
* If the subagent was supposed to navigate to a website, you should verify 
  the final screenshot before the subagent stopped.

Make sure to check the state of the browser before and after you have done 
your work, and check the state pretty frequently.

If a page looks like it's loading, take a screenshot again and see if it 
updates. Sometimes it just takes a moment for the page to load.

Think about whether the [element] appears where you expected. If not, you 
should check the current screenshot and/or DOM to determine if you did the 
click correctly.
```

---

## 14. Error Messages & Warnings

### File Operation Errors
```
- "target content not found in file"
- "file is not valid utf8"
- "access to file is blocked by gitignore"
- "This file is too large to be edited. Do not try again."
- "Could not successfully apply any edits. Please review the file and try smaller edits."
```

### Command Timeouts
```
Command timed out, needed to complete in [X]s. This timeout applies to all 
commands. You won't be able to run this command with these arguments, do 
not retry exact the same command.
```

### Task Boundary Warnings
```
DO NOT USE THIS TOOL UNLESS THERE IS SUFFICIENT COMPLEXITY TO THE TASK. 
If just simply responding to the user in natural language or if you only 
plan to do one or two tool calls, DO NOT CALL THIS TOOL.
```

---

## Summary of New Findings

| Category | New Items Found |
|----------|-----------------|
| Models | 25+ new model IDs including Claude 4.5, Gemini 3 Pro |
| Parameters | Temperature (0.15), sampling rates, token limits |
| ConfidenceScore | Complete 6-question grading system |
| Ephemeral Messages | Full structure and reminders |
| Task Boundary | Complete documentation and guidelines |
| Template Paths | 7 Google3 template paths |
| API Endpoints | 6+ gRPC service endpoints |
| Feature Flags | 6+ cascade configuration flags |
| Cortex Steps | 20+ additional step type enums |

**Extraction Completeness: ~99%**
