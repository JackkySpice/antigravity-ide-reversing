# AntiGravity IDE - Additional Systems Documentation

## 1. Artifact System

### Artifact Types
| Type | Purpose |
|------|---------|
| `ARTIFACT_TYPE_IMPLEMENTATION_PLAN` | Development plans with proposed changes |
| `ARTIFACT_TYPE_TASK` | Task tracking (task.md) |
| `ARTIFACT_TYPE_WALKTHROUGH` | User walkthroughs |
| `GEN_AI_ARTIFACT_TYPE_MODEL_NAME` | Model name artifacts |
| `other` | General code files |

### Storage Paths
```
~/.gemini/antigravity/brain/<UUID>/           # Planning artifacts
    ├── task.md                               # Task tracking
    ├── implementation_plan.md                # Development plan
    └── walkthrough.md                        # User walkthrough

~/.gemini/antigravity/scratch/<project>/      # Code artifacts
    └── src/, style.css, index.html, etc.
```

### Artifact Metadata Structure
```json
{
  "ArtifactMetadata": {
    "ArtifactType": "implementation_plan|task|walkthrough|other",
    "Summary": "Brief description of artifact content"
  },
  "CodeContent": "...",
  "Complexity": 1-10,
  "Description": "What this artifact does",
  "EmptyFile": false,
  "IsArtifact": true,
  "Overwrite": true,
  "TargetFile": "/absolute/path/to/file"
}
```

### Approval Status Constants
| Status | Meaning |
|--------|---------|
| `ARTIFACT_APPROVAL_STATUS_UNSPECIFIED` | No status set |
| `ARTIFACT_APPROVAL_STATUS_APPROVED` | User approved |
| `ARTIFACT_APPROVAL_STATUS_REQUESTED_CHANGES` | User requested revisions |
| `ARTIFACT_REVIEW_STATUS_PENDING_REVIEW` | Awaiting user review |
| `ARTIFACT_REVIEW_STATUS_REVIEWED` | Review complete |
| `ARTIFACT_REVIEW_STATUS_MODIFIED` | Modified after review |

### Artifact Lifecycle
```
PLANNING → Create implementation_plan.md
         → Set IsArtifact: true
         → Request review via notify_user with PathsToReview
              ↓
USER REVIEW → Status: PENDING_REVIEW
            → ShouldAutoProceed: true → continue
            → BlockedOnUser: true → wait for approval
              ↓
EXECUTION → Write code to scratch/<project>/
          → IsArtifact: false for code files
          → Update task.md with checkboxes
              ↓
VERIFICATION → Update walkthrough.md
             → Final notify_user
             → Status: APPROVED on success
```

---

## 2. MCP (Model Context Protocol) Integration

### Server Configuration
```
mcp_config.json                              # Primary config file
~/.gemini/antigravity/mcp/                   # MCP server configs
```

### Proto Structures
- `McpServerConfig` - Server configuration
- `McpServerSpec` - Server specification
- `McpServerState` - Runtime state
- `McpServerStatus` - Status enum

### API Endpoints
```
/LanguageServerService/RefreshMcpServers
/LanguageServerService/GetMcpServerStates
/LanguageServerService/ListMcpResources
/ApiServerService/GetMcpServerTemplates
```

### MCP Operations
| Category | Operations |
|----------|------------|
| Tools | `tools/call`, `tools/list` |
| Resources | `resources/list`, `resources/read` |
| Prompts | `prompts/list`, `prompts/get` |
| Session | `initialize`, `ping`, `wait` |
| Logging | `log` |
| Progress | `notifications/progress` |

### Feature Flag
```
CASCADE_ENABLE_MCP_TOOLS                     # Enable/disable MCP tools
```

---

## 3. Git Integration

### Commit Workflow
```
CORTEX_STEP_TYPE_GIT_COMMIT
  ↓
Generate commit message (if ENABLE_COMMIT_MESSAGE_GENERATION)
  ↓
Get parent commits
  ↓
Create commit with message
```

### PR Operations
| Component | Purpose |
|-----------|---------|
| `POST_PR_REVIEW` | Post review comments |
| `PullRequestDescriptionGuidelines` | PR description rules |
| `ReviewJudgeConfig` | Review judgment config |

### GitHub Integration
```
ConnectGithubAccountRequest/Response         # Account linking
DisconnectGithubAccountRequest               # Account unlinking
GetGitHubAccountStatusRequest/Response       # Connection status
GetGitHubAccessTokenRequest/Response         # Token management
```

### Diff Types
| Type | Purpose |
|------|---------|
| `FileDiff_Add` | Added file |
| `FileDiff_Modify` | Modified file |
| `FileDiff_Remove` | Removed file |
| `FileDiffComment` | Comments on diffs |

---

## 4. Workspace Management

### Workspace Tracking
```go
map[fs.Uri]*file_watcher.Workspace           // Main workspace map
GetWorkspacePaths()                          // Get workspace paths
AddWorkspaceEvent                            // Add workspace event
RemoveWorkspaceFolder                        // Remove workspace
```

### File Watching
```
*file_watcher.FileWatcherDefault             // Default watcher
DidChangeWatchedFiles                        // LSP file change notifications
FILE_CHANGE_TYPE_UNSPECIFIED                 // Change type enum
```

### Gitignore Handling
```
allowTabAccessGitignoreFiles                 // Control gitignore access
"Search uses smart case and will ignore gitignored files by default"
```

### Access Restrictions
```
"You may not edit file extensions: [%s]"     // Extension restrictions
"path %s is not within a Knowledge Item"     // Path containment
"Cannot delete files that do not exist"      // Existence validation
"You are not allowed to access files not in active workspaces"
```

---

## 5. Error Handling & Recovery

### Error Types
| Category | Examples |
|----------|----------|
| Proto errors | `EncodedErrorLeaf`, `EncodedErrorDetails` |
| gRPC/HTTP | `unexpected HTTP status code: %d` |
| Auth | `authorization failed`, `expired certificate` |
| Edit failures | `Could not successfully apply any edits` |

### Retry Configurations
| Config | Purpose |
|--------|---------|
| `RetryConfig` | General retry with count/delay |
| `PlannerRetryConfig` | Planner-level retries |
| `ModelAPIRetryConfig` | API call retries |
| `ModelOutputRetryConfig` | Model output retries |
| `EmbeddingRetryPolicy` | Embedding retries |

### Retry Reasons
```
RETRY_REASON_EDIT                            # Edit failed
RETRY_REASON_REGENERATE                      # Regeneration needed
RETRY_REQUEST_LATER                          # Defer retry
```

### Fallback Behaviors
| Fallback | Purpose |
|----------|---------|
| `FallbackDialer` | Network fallback |
| `HandleReplaceFileContentWithFallback` | Edit fallback chain |
| Browser fallbacks | `checkVisibility` → `getBoundingClientRect` |
| Input fallbacks | keyboard → fill method |

### User-Facing Error Messages
```
"Could not successfully apply any edits. Please review the file and try 
smaller edits that you are more confident about."

"Page navigated during text input. Please refresh the page state and try again."

"The agent encountered an error while saving unsaved changes in a file it 
was trying to %s. Please save the file and try again."

"No results found in this codebase search. Either the target search paths 
do not exist, or the search index is not ready."

"Model produced a malformed edit that the agent was unable to apply."
```

---

## 6. Image Generation

### Configuration
```json
{
  "imageGenerationModelIds": ["gemini-3-pro-image"]
}
```

### Step Type
```
CORTEX_STEP_TYPE_GENERATE_IMAGE
```

### Usage
```
"Generating image using prompt: %s and images at paths: %s"

"Do not output the path of this image to show to the user since the user 
can already see it. However, you can embed this image in artifacts for 
the USER's review."
```

---

## 7. Deployment (Firebase/Cloudflare)

### Step Types
```
CORTEX_STEP_TYPE_DEPLOY_FIREBASE
CORTEX_STEP_TYPE_SET_UP_FIREBASE
```

### Deployment Providers
```
DEPLOYMENT_PROVIDER_CLOUDFLARE
DEPLOYMENT_PROVIDER_UNSPECIFIED
```

### Build Status
```
DEPLOYMENT_BUILD_STATUS_UNSPECIFIED
DEPLOYMENT_BUILD_STATUS_INITIALIZING
DEPLOYMENT_BUILD_STATUS_BUILDING
DEPLOYMENT_BUILD_STATUS_CANCELED
```

### Hybrid Deployment
```
RegisterHybridDeployment
CheckHybridDeploymentStatus
GetHybridDeploymentsInternal
RemoveHybridDeploymentInternal
HYBRID_DEPLOYMENT_STATUS_HEALTHY
```

### Cloudflare Tunnel
```
"cloudflared: command not found"             # Error if not installed
"Cloudflare Tunnel for public hosting"       # Feature description
"Hosting in Tunnel" task                     # Task name
```

---

## 8. Rate Limiting & Quotas

### Quota Structure
```json
{
  "quotaInfo": {
    "remainingFraction": 1,
    "resetTime": "2026-01-02T09:48:31Z"
  }
}
```

### Proto Types
```
*codeium_common_go_proto.QuotaInfo
*errdetails.QuotaFailure_Violation
*v1_storage_go_proto.ThrottleState
```

### Tier-Based Limits
```
TEAMS_TIER_PRO
TEAMS_TIER_TEAMS
TEAMS_TIER_ENTERPRISE_SELF_HOSTED
TEAMS_TIER_ENTERPRISE_SAAS
TEAMS_TIER_HYBRID
TEAMS_TIER_PRO_ULTIMATE
```

### Upgrade Message
```
"You can upgrade to the Google AI Ultra plan to receive the highest rate limits."
```

### Throttling
```javascript
throttleFunction(updatePositions, 16);       // ~60fps throttle
RetryThrottling                              // gRPC retry throttle
```

---

## 9. Web Search Integration

### Step Type
```
CORTEX_STEP_TYPE_SEARCH_WEB
SEARCH_WEB_TYPE_IMAGE                        # Image search
```

### Search Providers
```
GROUNDING_WITH_GOOGLE_SEARCH
WEB_GROUNDING_FOR_ENTERPRISE
THIRD_PARTY_WEB_SEARCH_MODEL_O3
```

### Feature Flags
```
CASCADE_WEB_SEARCH_TOOL_ENABLED
CASCADE_WEB_SEARCH_TOOL_DISABLED
```

---

## 10. Memory System

### Step Types
```
CORTEX_STEP_TYPE_MEMORY
CORTEX_STEP_TYPE_BRAIN_UPDATE
CORTEX_STEP_TYPE_RETRIEVE_MEMORY
```

### Memory Triggers
```
CORTEX_MEMORY_TRIGGER_UNSPECIFIED
CORTEX_MEMORY_TRIGGER_MANUAL
CORTEX_MEMORY_TRIGGER_GLOB
CORTEX_MEMORY_TRIGGER_ALWAYS_ON
CORTEX_MEMORY_TRIGGER_MODEL_DECISION
```

### Memory Sources
```
CORTEX_MEMORY_SOURCE_CASCADE
```

### Brain Entry Types
```
BRAIN_ENTRY_TYPE_UNSPECIFIED
BRAIN_ENTRY_TYPE_PLAN
BRAIN_ENTRY_TYPE_TASK
```

### Update Triggers
```
BRAIN_UPDATE_TRIGGER_SYSTEM_FORCED
BRAIN_UPDATE_TRIGGER_USER_NEW_INFO
```

### Feature Flags
```
CASCADE_ENABLE_AUTOMATED_MEMORIES
CASCADE_MEMORY_CONFIG_OVERRIDE
```

---

## 11. Conversation Sharing

### Status
```
TRAJECTORY_SHARE_STATUS_TEAM
```

### API Endpoints
```
/ApiServerService/DeleteTrajectoryShare
IsConversationSharingBlocked
```

---

## 12. Code Acknowledgement

### Scope Types
```
CODE_ACKNOWLEDGEMENT_SCOPE_FILE
CODE_ACKNOWLEDGEMENT_SCOPE_HUNK
```

### Step Type
```
CORTEX_STEP_TYPE_CODE_ACKNOWLEDGEMENT
```

---

## 13. Linting & Compilation

### Lint Step Types
```
CORTEX_STEP_TYPE_LINT_DIFF
CORTEX_STEP_TYPE_LINT_APPLET
```

### Lint Diff Types
```
LINT_DIFF_TYPE_UNSPECIFIED
LINT_DIFF_TYPE_INSERT
LINT_DIFF_TYPE_DELETE
LINT_DIFF_TYPE_UNCHANGED
```

### Compile Step Types
```
CORTEX_STEP_TYPE_COMPILE
CORTEX_STEP_TYPE_COMPILE_APPLET
```

### Compile Tools
```
CORTEX_STEP_COMPILE_TOOL_UNSPECIFIED
CORTEX_STEP_COMPILE_TOOL_PYLINT
```

---

## 14. Supercomplete (Tab Jump)

### Step Types
```
CORTEX_STEP_TYPE_SUPERCOMPLETE (trajectory type)
```

### Feature Flags
```
SUPERCOMPLETE_USE_CURRENT_LINE
SUPERCOMPLETE_REGULAR_DEBOUNCE
SUPERCOMPLETE_INLINE_PURE_DELETE
SUPERCOMPLETE_FILTER_PREFIX_MATCH
SUPERCOMPLETE_FILTER_DELETION_CAP
SUPERCOMPLETE_FILTER_SUFFIX_MATCH
```

### API Methods
```
Tab.CreateSupercompleteTrajectory
Tab.UpdateSupercompleteTrajectory
Tab.SupercompleteTrajectoryPromptParams
```

---

## 15. Feature Flags (Complete List)

### Cascade Flags
```
CASCADE_GLOBAL_CONFIG_OVERRIDE
CASCADE_MEMORY_CONFIG_OVERRIDE
CASCADE_DEFAULT_MODEL_OVERRIDE
CASCADE_PREMIUM_CONFIG_OVERRIDE
CASCADE_ENABLE_MCP_TOOLS
CASCADE_ENABLE_AUTOMATED_MEMORIES
CASCADE_WEB_SEARCH_TOOL_ENABLED
CASCADE_WEB_SEARCH_TOOL_DISABLED
CASCADE_RUN_EXTENSION_CODE_ONLY
```

### Model Flags
```
ENABLE_COMMIT_MESSAGE_GENERATION
ENABLE_AUTOCOMPLETE_DURING_INTELLISENSE
```

### UI Flags
```
BLOCK_TAB_ON_SHOWN_AUTOCOMPLETE
TAB_JUMP_FILTER_SCORE_THRESHOLD
TAB_JUMP_FILTER_WHITESPACE_ONLY
```

### Debug Flags
```
API_SERVER_ENABLE_MORE_LOGGING
PROFILING_TELEMETRY_SAMPLE_RATE
ANTIGRAVITY_SENTRY_SAMPLE_RATE
```

---

## 16. Supported MIME Types

```json
{
  "application/json": true,
  "application/pdf": true,
  "application/rtf": true,
  "application/x-ipynb+json": true,
  "application/x-javascript": true,
  "application/x-python-code": true,
  "application/x-typescript": true,
  "audio/webm;codecs=opus": true,
  "image/heic": true,
  "image/heif": true,
  "image/jpeg": true,
  "image/png": true,
  "image/webp": true,
  "text/css": true,
  "text/csv": true,
  "text/html": true,
  "text/javascript": true,
  "text/markdown": true,
  "text/plain": true,
  "text/rtf": true,
  "text/x-python": true,
  "text/x-python-script": true,
  "text/x-typescript": true,
  "text/xml": true,
  "video/audio/s16le": true,
  "video/audio/wav": true,
  "video/jpeg2000": true,
  "video/text/timestamp": true,
  "video/videoframe/jpeg2000": true
}
```

---

*Generated: January 2, 2026*
