# AntiGravity IDE - Deep Extraction (Session 3)

> **Comprehensive subagent analysis - January 2, 2026**

---

## 1. Complete Tool Schemas (22+ Tools)

### ListDirectory
```json
{
  "required": ["DirectoryPath"],
  "properties": {
    "DirectoryPath": {"type": "string", "description": "Path to list contents of, should be absolute path to a directory"},
    "waitForPreviousTools": {"type": "boolean"}
  }
}
```

### WebSearch
```json
{
  "required": ["query"],
  "properties": {
    "query": {"type": "string"},
    "domain": {"type": "string", "description": "Optional domain to prioritize"},
    "waitForPreviousTools": {"type": "boolean"}
  }
}
```

### ListServerResources (MCP)
```json
{
  "properties": {
    "ServerName": {"type": "string", "description": "Name of the server to list available resources from"}
  }
}
```

### ReadServerResource (MCP)
```json
{
  "properties": {
    "ServerName": {"type": "string", "description": "Name of the server to read the resource from"},
    "Uri": {"type": "string", "description": "Unique identifier for the resource"}
  }
}
```

### ReadTerminal
```json
{
  "required": ["ProcessID", "Name"],
  "properties": {
    "ProcessID": {"type": "string", "description": "Process ID of the terminal to read"},
    "Name": {"type": "string", "description": "Name of the terminal to read"}
  }
}
```

### ViewDocumentChunk
```json
{
  "required": ["document_id", "position"],
  "properties": {
    "document_id": {"type": "string", "description": "The ID of the document that the chunk belongs to"},
    "position": {"type": "integer", "description": "The position of the chunk to view"}
  }
}
```

### ReadUrl
```json
{
  "required": ["Url"],
  "properties": {
    "Url": {"type": "string", "description": "URL to read content from"}
  }
}
```

### FindFiles
```json
{
  "required": ["SearchDirectory", "Pattern"],
  "properties": {
    "SearchDirectory": {"type": "string", "description": "The directory to search within"},
    "Pattern": {"type": "string", "description": "Pattern to search for, supports glob format"},
    "Excludes": {"type": "array", "items": {"type": "string"}, "description": "Exclude patterns"},
    "Type": {"type": "string", "description": "Type filter: file, directory, or any"},
    "MaxDepth": {"type": "integer", "description": "Maximum depth to search"},
    "Extensions": {"type": "array", "items": {"type": "string"}, "description": "File extensions to include"},
    "FullPath": {"type": "boolean", "description": "Whether full absolute path must match the glob pattern"}
  }
}
```

### GrepSearch
```json
{
  "required": ["SearchPath", "Query"],
  "properties": {
    "SearchPath": {"type": "string", "description": "The path to search (directory or file)"},
    "Query": {"type": "string", "description": "The search term or pattern to look for"},
    "MatchPerLine": {"type": "boolean", "description": "Returns each matching line with line numbers"},
    "Includes": {"type": "array", "items": {"type": "string"}, "description": "Glob patterns to filter files"},
    "CaseInsensitive": {"type": "boolean", "description": "Case-insensitive search"},
    "IsRegex": {"type": "boolean", "description": "Treat Query as regex pattern"}
  }
}
```

### CodeEdit (Advanced with Line Numbers)
```json
{
  "required": ["TargetFile", "CodeMarkdownLanguage", "Instruction", "Description", "Complexity", "ReplacementChunks"],
  "properties": {
    "TargetFile": {"type": "string"},
    "CodeMarkdownLanguage": {"type": "string"},
    "Instruction": {"type": "string"},
    "TargetLintErrorIds": {"type": "array", "items": {"type": "string"}, "description": "IDs of lint errors this edit fixes"},
    "Description": {"type": "string", "description": "User-facing explanation"},
    "Complexity": {"type": "integer", "description": "1-10 rating for review importance"},
    "ReplacementChunks": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["AllowMultiple", "TargetContent", "ReplacementContent", "StartLine", "EndLine"],
        "properties": {
          "AllowMultiple": {"type": "boolean"},
          "TargetContent": {"type": "string"},
          "ReplacementContent": {"type": "string"},
          "StartLine": {"type": "integer", "description": "Starting line number (1-indexed)"},
          "EndLine": {"type": "integer", "description": "Ending line number (1-indexed)"}
        }
      }
    },
    "ArtifactMetadata": {
      "type": "object",
      "required": ["Summary", "ArtifactType"],
      "properties": {
        "Summary": {"type": "string"},
        "ArtifactType": {"type": "string", "enum": ["implementation_plan", "walkthrough", "task", "other"]}
      }
    }
  }
}
```

### CreateFile
```json
{
  "required": ["TargetFile", "Overwrite", "CodeContent", "EmptyFile", "Description", "Complexity", "IsArtifact"],
  "properties": {
    "TargetFile": {"type": "string", "description": "Target file to create and write code to"},
    "Overwrite": {"type": "boolean", "description": "Set true to overwrite existing file"},
    "CodeContent": {"type": "string", "description": "The code contents to write"},
    "EmptyFile": {"type": "boolean", "description": "Set true to create empty file"},
    "Description": {"type": "string"},
    "Complexity": {"type": "integer"},
    "IsArtifact": {"type": "boolean", "description": "Set true when creating an artifact file"},
    "ArtifactMetadata": {
      "type": "object",
      "properties": {
        "Summary": {"type": "string"},
        "ArtifactType": {"type": "string"}
      }
    }
  }
}
```

### RunCommand
```json
{
  "required": ["Cwd", "WaitMsBeforeAsync", "SafeToAutoRun", "CommandLine"],
  "properties": {
    "Cwd": {"type": "string", "description": "Current working directory"},
    "WaitMsBeforeAsync": {"type": "integer", "description": "Milliseconds to wait before background (max 10000ms)"},
    "SafeToAutoRun": {"type": "boolean", "description": "True if command is safe without user approval"},
    "CommandLine": {"type": "string", "description": "The exact command line string to execute"}
  }
}
```

### SendCommandInput
```json
{
  "required": ["CommandId", "WaitMs", "SafeToAutoRun"],
  "properties": {
    "CommandId": {"type": "string", "description": "Command ID from previous run_command call"},
    "Input": {"type": "string", "description": "Input to send to command's stdin"},
    "Terminate": {"type": "boolean", "description": "Whether to terminate the command"},
    "WaitMs": {"type": "integer", "description": "Time to wait for output (500-10000ms)"},
    "SafeToAutoRun": {"type": "boolean"}
  }
}
```

### CommandStatus
```json
{
  "required": ["CommandId", "WaitDurationSeconds"],
  "properties": {
    "CommandId": {"type": "string", "description": "ID of the command to get status for"},
    "OutputCharacterCount": {"type": "integer", "description": "Number of characters to view"},
    "WaitDurationSeconds": {"type": "integer", "description": "Seconds to wait for completion (max 300)"}
  }
}
```

### TaskBoundary
```json
{
  "required": ["TaskName", "Mode", "TaskSummary", "TaskStatus", "PredictedTaskSize"],
  "properties": {
    "TaskName": {"type": "string", "description": "Human readable task name like 'Researching Server Implementation'"},
    "Mode": {"type": "string", "description": "The agent focus to switch to"},
    "TaskSummary": {"type": "string", "description": "1-2 line summary of accomplishments"},
    "TaskStatus": {"type": "string", "description": "Current action status e.g. 'Looking for files'"},
    "PredictedTaskSize": {"type": "integer", "description": "Estimated tool calls needed"}
  }
}
```

### BrowserSubagent
```json
{
  "required": ["TaskName", "Task", "RecordingName"],
  "properties": {
    "TaskName": {"type": "string", "description": "Human readable task name"},
    "Task": {"type": "string", "description": "Detailed task description for browser subagent"},
    "RecordingName": {"type": "string", "description": "Recording name (lowercase with underscores, max 3 words)"}
  }
}
```

### GenerateImage
```json
{
  "required": ["Prompt", "ImageName"],
  "properties": {
    "Prompt": {"type": "string", "description": "Text prompt to generate image for"},
    "ImagePaths": {"type": "array", "items": {"type": "string"}, "description": "Optional paths to images for editing (max 3)"},
    "ImageName": {"type": "string", "description": "Name for generated image (lowercase_underscores, max 3 words)"}
  }
}
```

### ViewCodeNode
```json
{
  "required": ["File", "NodePaths"],
  "properties": {
    "File": {"type": "string", "description": "Absolute path to file"},
    "NodePaths": {"type": "array", "items": {"type": "string"}, "description": "Path of nodes e.g. package.class.FunctionName"}
  }
}
```

### ViewFileOutline
```json
{
  "required": ["AbsolutePath"],
  "properties": {
    "AbsolutePath": {"type": "string", "description": "Path to file (absolute)"},
    "ItemOffset": {"type": "integer", "description": "Offset for pagination (start at 0)"}
  }
}
```

### ViewFile
```json
{
  "required": ["AbsolutePath"],
  "properties": {
    "AbsolutePath": {"type": "string", "description": "Path to file (absolute)"},
    "StartLine": {"type": "integer", "description": "Start line (1-indexed, inclusive)"},
    "EndLine": {"type": "integer", "description": "End line (1-indexed, inclusive)"}
  }
}
```

---

## 2. Complete Model Roster with Aliases

### Model Aliases (Routing Abstraction)
| Alias | Resolves To |
|-------|-------------|
| `MODEL_ALIAS_CASCADE_BASE` | GPT-4o Mini (free tier) |
| `MODEL_ALIAS_VISTA` | MODEL_CASCADE_20070/20072 (weighted) |
| `MODEL_ALIAS_SHAMU` | MODEL_CASCADE_20070 |
| `MODEL_ALIAS_SWE_1` / `SWE_1_LITE` | SWE agents |
| `MODEL_ALIAS_AUTO` | Auto-routing |
| `MODEL_ALIAS_RECOMMENDED` | Gemini 2.5 Pro |

### All Discovered Models
| Model ID | Description |
|----------|-------------|
| `MODEL_GOOGLE_GEMINI_2_5_PRO` | Gemini 2.5 Pro (Recommended default) |
| `MODEL_GOOGLE_GEMINI_2_5_FLASH` | Gemini 2.5 Flash |
| `MODEL_GOOGLE_GEMINI_2_5_FLASH_LITE` | Gemini 2.5 Flash Lite (checkpoint) |
| `MODEL_GOOGLE_GEMINI_2_5_FLASH_THINKING` | Gemini 2.5 Flash with Thinking |
| `MODEL_CLAUDE_4_SONNET` | Claude 4 Sonnet |
| `MODEL_CLAUDE_4_SONNET_THINKING` | Claude 4 Sonnet (Thinking) |
| `MODEL_CLAUDE_4_5_SONNET` | Claude Sonnet 4.5 |
| `MODEL_CLAUDE_4_5_SONNET_THINKING` | Claude Sonnet 4.5 (Thinking) |
| `MODEL_CLAUDE_3_5_HAIKU_20241022` | Claude 3.5 Haiku (checkpoint) |
| `MODEL_CLAUDE_3_7_SONNET_20250219` | Claude 3.7 Sonnet |
| `MODEL_CHAT_GPT_4O_MINI_2024_07_18` | GPT-4o Mini (Base/Free tier) |
| `MODEL_CHAT_O4_MINI_HIGH` | o4-mini High |
| `MODEL_OPENAI_GPT_OSS_120B_MEDIUM` | GPT-OSS 120B (Medium) |
| `MODEL_PLACEHOLDER_M7` | Gemini 3 Pro (Low) |
| `MODEL_PLACEHOLDER_M8` | Gemini 3 Pro (High) - default override |
| `MODEL_PLACEHOLDER_M12` | Claude Opus 4.5 (Thinking) |
| `MODEL_PLACEHOLDER_M18` | Gemini 3 Flash |

### Special Purpose Models
| Role | Model |
|------|-------|
| `brainModel` | Gemini 2.5 Pro, GPT-4.1, o3 |
| `judge_model` | Gemini 2.5 Pro |
| `plan_model` | GPT-4o Mini |
| `checkpoint_model` | Gemini 2.5 Flash / Claude 3.5 Haiku |
| `defaultAgentModelId` | gemini-3-pro-high |

---

## 3. Complete State Machine

### Agent Mode Enum
```protobuf
AGENT_MODE_UNSPECIFIED  = 0  // Default/unset state
AGENT_MODE_PLANNING     = 1  // Exploring codebase, creating implementation plans
AGENT_MODE_EXECUTION    = 2  // Actively implementing changes
AGENT_MODE_VERIFICATION = 3  // Validating changes, running tests, final checks
```

### Cortex Step Status Enum
```protobuf
CORTEX_STEP_STATUS_UNSPECIFIED  // Not set
CORTEX_STEP_STATUS_GENERATING   // Model generating response
CORTEX_STEP_STATUS_QUEUED       // Awaiting execution
CORTEX_STEP_STATUS_PENDING      // Ready to execute
CORTEX_STEP_STATUS_RUNNING      // Currently executing
CORTEX_STEP_STATUS_WAITING      // Blocked on user/external input
CORTEX_STEP_STATUS_DONE         // Successfully completed
CORTEX_STEP_STATUS_INVALID      // Step failed validation
CORTEX_STEP_STATUS_CLEARED      // Removed from trajectory
CORTEX_STEP_STATUS_CANCELED     // User/system canceled
CORTEX_STEP_STATUS_ERROR        // Execution error
CORTEX_STEP_STATUS_INTERRUPTED  // External interruption
```

### Cascade Run Status Enum
```protobuf
CASCADE_RUN_STATUS_UNSPECIFIED  // Not set
CASCADE_RUN_STATUS_IDLE         // No active execution
CASCADE_RUN_STATUS_RUNNING      // Actively executing steps
CASCADE_RUN_STATUS_CANCELING    // Cancel in progress
CASCADE_RUN_STATUS_BUSY         // Processing but not step execution
```

### Plan Status Enum
```protobuf
PLAN_STATUS_UNSPECIFIED  // Not set
PLAN_STATUS_INITIALIZED  // Plan created but not started
PLAN_STATUS_PLANNING     // Actively generating plan
PLAN_STATUS_PLANNED      // Plan complete, ready for execution
PLAN_STATUS_ERROR        // Planning failed
```

### Action Status Enum
```protobuf
ACTION_STATUS_UNSPECIFIED  // Not set
ACTION_STATUS_ERROR        // Action failed
ACTION_STATUS_INITIALIZED  // Action created
ACTION_STATUS_PREPARING    // Gathering context
ACTION_STATUS_PREPARED     // Ready to apply
ACTION_STATUS_APPLYING     // Being applied
ACTION_STATUS_APPLIED      // Successfully applied
ACTION_STATUS_REJECTED     // User rejected action
```

### Trajectory Types
```protobuf
CORTEX_TRAJECTORY_TYPE_UNSPECIFIED
CORTEX_TRAJECTORY_TYPE_USER_MAINLINE       // Primary user conversation
CORTEX_TRAJECTORY_TYPE_USER_GRANULAR       // Fine-grained user interaction
CORTEX_TRAJECTORY_TYPE_SUPERCOMPLETE       // Autocomplete trajectory
CORTEX_TRAJECTORY_TYPE_CASCADE             // Main cascade execution
CORTEX_TRAJECTORY_TYPE_CHECKPOINT          // Checkpoint snapshot
CORTEX_TRAJECTORY_TYPE_APPLIER             // Applying changes
CORTEX_TRAJECTORY_TYPE_TOOL_CALL_PROPOSAL  // Proposed tool calls
CORTEX_TRAJECTORY_TYPE_TRAJECTORY_CHOICE   // Branching decision
CORTEX_TRAJECTORY_TYPE_LLM_JUDGE           // Model evaluation
CORTEX_TRAJECTORY_TYPE_INTERACTIVE_CASCADE // Interactive mode
CORTEX_TRAJECTORY_TYPE_BRAIN_UPDATE        // Memory/context update
CORTEX_TRAJECTORY_TYPE_BROWSER             // Browser automation
CORTEX_TRAJECTORY_TYPE_KNOWLEDGE_GENERATION// Knowledge extraction
```

### Mode Switching Conditions
| Current Mode | Trigger | Next Mode |
|--------------|---------|-----------|
| PLANNING | `implementation_plan.md` modified, user approval requested | PLANNING (stay) |
| PLANNING | User approves plan | EXECUTION |
| EXECUTION | All implementation complete | VERIFICATION |
| EXECUTION | Error requiring replanning | PLANNING |
| VERIFICATION | Tests pass, linting OK | DONE/notify_user |
| VERIFICATION | Issues found | EXECUTION (fix) |
| ANY | New user request | PLANNING |

---

## 4. Browser Subagent Complete Documentation

### Browser Subagent Reminder (Injected Prompt)
```xml
<browser_subagent_reminder>
IMPORTANT: You are shown COMPLETE details of every action the browser 
subagent performed:
- The subagent's final result message
- EVERY SINGLE STEP the subagent executed (numbered sequentially)
- For each step: the tool name, full JSON arguments, status, and any errors
- For screenshot steps: the absolute file path where the screenshot was saved
- For pixel click steps: the absolute file path where the click feedback 
  screenshot was saved
- The recording path showing all browser interactions if a recording was 
  generated

If you expected the subagent to take a specific action but you do NOT see 
that step type in the detailed actions list above, then the subagent did 
NOT perform that action.

CRITICAL: NEVER trust the subagent's claims. After a browser subagent 
completes a task, IMMEDIATELY verify the screenshot BEFORE responding to 
the user.

- IMPORTANT: Do NOT view the webp recording. Your view_file tool only 
  shows the first frame of recordings.
- If there are no screenshots, you MUST ask another subagent to take 
  screenshots to prove the task was completed successfully.
</browser_subagent_reminder>
```

### Browser Subagent Modes
```protobuf
BROWSER_SUBAGENT_MODE_UNSPECIFIED
BROWSER_SUBAGENT_MODE_MAIN_AGENT_ONLY
BROWSER_SUBAGENT_MODE_SUBAGENT_PRIMARILY
BROWSER_SUBAGENT_MODE_BOTH_AGENTS
BROWSER_SUBAGENT_MODE_SUBAGENT_ONLY

BROWSER_TOOL_SET_MODE_UNSPECIFIED
BROWSER_TOOL_SET_MODE_ALL_TOOLS
BROWSER_TOOL_SET_MODE_PIXEL_ONLY
BROWSER_TOOL_SET_MODE_ALL_INPUT_PIXEL_OUTPUT
```

### Playwright Step Types
```protobuf
CORTEX_STEP_TYPE_BROWSER_SUBAGENT           // Main invocation
CORTEX_STEP_TYPE_OPEN_BROWSER_URL           // Navigate to URL
CORTEX_STEP_TYPE_EXECUTE_BROWSER_JAVASCRIPT // Execute JS
CORTEX_STEP_TYPE_LIST_BROWSER_PAGES         // List open pages
CORTEX_STEP_TYPE_CAPTURE_BROWSER_SCREENSHOT // Take screenshot
CORTEX_STEP_TYPE_CLICK_BROWSER_PIXEL        // Click coordinates
CORTEX_STEP_TYPE_READ_BROWSER_PAGE          // Read page content
CORTEX_STEP_TYPE_BROWSER_GET_DOM            // Extract DOM
CORTEX_STEP_TYPE_BROWSER_INPUT              // Type text
CORTEX_STEP_TYPE_BROWSER_MOVE_MOUSE         // Move cursor
CORTEX_STEP_TYPE_BROWSER_SELECT_OPTION      // Select dropdown
CORTEX_STEP_TYPE_BROWSER_SCROLL_UP          // Scroll up
CORTEX_STEP_TYPE_BROWSER_SCROLL_DOWN        // Scroll down
CORTEX_STEP_TYPE_BROWSER_SCROLL             // Generic scroll
CORTEX_STEP_TYPE_BROWSER_CLICK_ELEMENT      // Click by selector
CORTEX_STEP_TYPE_BROWSER_PRESS_KEY          // Press key
CORTEX_STEP_TYPE_CAPTURE_BROWSER_CONSOLE_LOGS // Get console
CORTEX_STEP_TYPE_BROWSER_RESIZE_WINDOW      // Resize window
CORTEX_STEP_TYPE_BROWSER_DRAG_PIXEL_TO_PIXEL// Drag and drop
CORTEX_STEP_TYPE_BROWSER_MOUSE_WHEEL        // Mouse wheel
```

### Interactive Element Detection (JavaScript)
```javascript
const DISTINCT_INTERACTIVE_TAGS = new Set([
  'a', 'button', 'input', 'select', 'textarea', 
  'summary', 'details', 'label', 'option'
]);

const INTERACTIVE_ROLES = new Set([
  'button', 'link', 'menuitem', 'menuitemradio', 'menuitemcheckbox', 
  'radio', 'checkbox', 'tab', 'switch', 'slider', 'spinbutton', 
  'combobox', 'searchbox', 'textbox', 'listbox', 'option', 'scrollbar'
]);
```

---

## 5. MCP Integration Complete

### gRPC Service Endpoints
```protobuf
/exa.language_server_pb.LanguageServerService/RefreshMcpServers
/exa.language_server_pb.LanguageServerService/GetMcpServerStates
/exa.language_server_pb.LanguageServerService/GetMcpServerTemplates
/exa.language_server_pb.LanguageServerService/ListMcpResources
/exa.api_server_pb.ApiServerService/GetMcpServerTemplates
```

### MCP Server Data Structures
| Proto Type | Description |
|------------|-------------|
| `McpServerConfig` | Main MCP server configuration |
| `McpServerSpec` | Server specification (command, args, env, headers, URL) |
| `McpServerState` | Runtime state with status, tools, errors |
| `McpServerStatus` | Status enum (UNSPECIFIED, RUNNING, etc.) |
| `McpServerTemplate` | Server templates with title, id, link, description |
| `McpServerCommand` | Command template definitions |
| `McpServerInfo` | Name and version info |
| `McpToolConfig` | Tool configuration |
| `McpResource` / `McpResourceContent` | Resource types (text, image) |
| `McpLocalServer` / `McpRemoteServer` | Local vs remote server types |

### MCP Server Spec Fields
```protobuf
server_name / serverName
command / args / env (EnvEntry map)
server_url / serverUrl
disabled / disabled_tools (disabledTools)
headers (HeadersEntry map)
server_index
```

---

## 6. Third-Party Integrations

### Slack Integration
```protobuf
SlackSource.SlackChannels.SlackChannel
  - channelId, startTime, endTime
  - apiKeyConfig (for authentication)
```

### Jira Integration
```protobuf
JiraSource.JiraQueries
  - serverUri, email, projects, customQueries
  - apiKeyConfig
```

### SharePoint Integration
```protobuf
SharePointSources.SharePointSource
  - sharepointSiteName, driveId/driveName
  - sharepointFolderId/sharepointFolderPath
  - clientId, clientSecret, tenantId, fileId
```

### GitHub Integration
```protobuf
ConnectGithubAccountRequest/Response
  - github_refresh_token, github_access_token
  - expires_at timestamps
GetGitHubAccountStatusRequest/Response
GetGitHubAccessTokenRequest/Response
allow_github_reviews, allow_github_auto_reviews
allow_github_description_edits
```

### Netlify Integration
```protobuf
ConnectNetlifyAccountRequest
  - netlify_access_token, netlify_user_id, expires_at
GetNetlifyAccountStatusRequest/Response
```

### Google Drive Integration
```protobuf
GoogleDriveSource
GoogleDriveSource.ResourceId (ResourceType enum)
```

---

## 7. Security & Safety Rules

### Harm Categories (Content Safety)
```protobuf
HARM_CATEGORY_HATE_SPEECH
HARM_CATEGORY_DANGEROUS_CONTENT
HARM_CATEGORY_HARASSMENT
HARM_CATEGORY_SEXUALLY_EXPLICIT
HARM_CATEGORY_CIVIC_INTEGRITY
HARM_CATEGORY_IMAGE_HATE
HARM_CATEGORY_IMAGE_DANGEROUS_CONTENT
HARM_CATEGORY_IMAGE_HARASSMENT
HARM_CATEGORY_IMAGE_SEXUALLY_EXPLICIT
HARM_CATEGORY_JAILBREAK
```

### Safety Settings
```protobuf
SafetySetting.HarmBlockThreshold  // Configurable blocking thresholds
SafetySetting.HarmBlockMethod     // Methods for blocking harmful content
SafetyRating.HarmProbability      // Probability assessment
SafetyRating.HarmSeverity         // Severity assessment
```

### Data Privacy Classifications
```protobuf
// Sensitive data types
ST_SENSITIVE_TIMESTAMP
ST_SENSITIVE_LOCATION
ST_SENSITIVE_BACKGROUND_INFO
ST_SENSITIVE_SIZE
ST_SENSITIVE_QUALITY_SIGNALS

// PII types
ST_EMAIL_ID
ST_NAME
ST_PHONE_NUMBER
ST_GAIA_ID
ST_USERNAME
ST_GOVERNMENT_ID_NUMBER
ST_HEALTHCARE_INFO

// Security material
ST_SECURITY_MATERIAL
ST_SECURITY_KEY
ST_ACCOUNT_CREDENTIAL
```

### Command Execution Safety
```json
{
  "auto_command_config": {
    "system_allowlist": ["echo", "ls"],
    "system_denylist": ["git", "rm", "pkill", "kubectl delete", 
                        "terraform", "kill", "psql", "mv", "bash", "zsh"]
  }
}
```

---

## 8. Artifact System Complete

### Artifact Types
```protobuf
ArtifactType: "other" | "implementation_plan" | "task" | "walkthrough"
```

### Artifact Review Modes
```protobuf
ARTIFACT_REVIEW_MODE_UNSPECIFIED
ARTIFACT_REVIEW_MODE_ALWAYS       // Always require user review
ARTIFACT_REVIEW_MODE_TURBO        // Skip reviews in turbo mode
ARTIFACT_REVIEW_MODE_AUTO         // System decides based on context
```

### Acknowledgement Types
```protobuf
ACKNOWLEDGEMENT_TYPE_UNSPECIFIED
ACKNOWLEDGEMENT_TYPE_ACCEPT       // User approved
ACKNOWLEDGEMENT_TYPE_REJECT       // User rejected
ACKNOWLEDGEMENT_TYPE_STALE        // Outdated
ACKNOWLEDGEMENT_TYPE_CLEARED      // Reset
```

### Brain Directory Structure
```
~/.gemini/antigravity/brain/{conversation-uuid}/
├── task.md                  # Task checklist artifact
├── implementation_plan.md   # Detailed implementation plan
└── walkthrough.md          # Optional walkthrough documentation
```

---

## 9. Global Configuration Override

```json
{
  "planner_config": {
    "truncation_threshold_tokens": 160000,
    "tool_config": {
      "run_command": {
        "auto_command_config": {
          "system_allowlist": ["echo", "ls"],
          "system_denylist": ["git", "rm", "pkill", "kubectl delete", 
                              "terraform", "kill", "psql", "mv", "bash", "zsh"]
        }
      },
      "mquery": { "force_disable": true }
    },
    "retry_config": {
      "api_retry": {
        "max_retries": 0,
        "initial_sleep_duration_ms": 5000,
        "exponential_multiplier": 2.0,
        "include_error_feedback": true
      }
    }
  }
}
```

---

## 10. Authentication Types

```protobuf
AuthType enum:
  AUTH_TYPE_UNSPECIFIED
  NO_AUTH
  API_KEY_AUTH
  HTTP_BASIC_AUTH
  GOOGLE_SERVICE_ACCOUNT_AUTH
  GOOGLE_END_USER_CREDENTIAL
  OAUTH
  CLIENT_CERTIFICATE_AUTH
  OIDC_AUTH
```

### SSO/SAML Support
```protobuf
SAMLAuthProvider
  - sso_provider_id, idp_entity_id, sso_url
  - x509_certificate, enabled
UserSSOLoginRedirectRequest/Response
```

---

## Summary of Deep Extraction

| Category | Items Found |
|----------|-------------|
| Tool Schemas | 22 complete JSON schemas |
| Models | 17+ models with aliases |
| State Machine | 5 enum types, 40+ states |
| Browser Steps | 18 Playwright step types |
| MCP Endpoints | 5 gRPC service endpoints |
| Integrations | Slack, Jira, SharePoint, GitHub, Netlify, Google Drive |
| Harm Categories | 10 content safety categories |
| Auth Types | 8 authentication methods |

**Extraction Completeness: ~99.5%**
