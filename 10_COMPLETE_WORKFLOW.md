# AntiGravity IDE - Complete Workflow Documentation

## Executive Summary

AntiGravity is an agentic AI coding assistant. The core workflow follows a **3-mode state machine** (PLANNING → EXECUTION → VERIFICATION) with a **tool-based execution engine** called Cortex.

---

## System Prompt Assembly Order

The system prompt is constructed in this **exact order**:

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        SYSTEM PROMPT ASSEMBLY ORDER                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  1. IDENTITY (STATIC)                                                           │
│     └─ "You are Antigravity, a powerful agentic AI coding assistant designed    │
│         by the Google Deepmind team working on Advanced Agentic Coding."        │
│                                                                                  │
│  2. COMMUNICATION STYLE (STATIC)                                                │
│     └─ Formatting, expressiveness, helpfulness, proactiveness guidelines        │
│                                                                                  │
│  3. TOOL CALLING FORMAT (STATIC)                                                │
│     └─ XML format: <tools></tools>, <tool_response></tool_response>             │
│     └─ "DO NOT WRITE MORE TEXT AFTER THE TOOL CALLS IN A RESPONSE"              │
│                                                                                  │
│  4. AGENTIC MODE OVERVIEW (STATIC)                                              │
│     └─ "You are in AGENTIC mode. You should take more time to research..."      │
│                                                                                  │
│  5. AGENTIC MODE OVERVIEW DYNAMIC (DYNAMIC - runtime injected)                  │
│     └─ Artifact directory paths, conversation-specific settings                 │
│                                                                                  │
│  6. MODE DESCRIPTIONS (STATIC)                                                  │
│     └─ PLANNING, EXECUTION, VERIFICATION mode definitions                       │
│                                                                                  │
│  7. PERSISTENT CONTEXT (DYNAMIC - user memories/rules)                          │
│     └─ "User-defined rules that you MUST ALWAYS FOLLOW WITHOUT EXCEPTION"       │
│                                                                                  │
│  8. TOOL-SPECIFIC SECTIONS (CONDITIONAL)                                        │
│     ├─ notify_user_tool.tmpl                                                    │
│     ├─ task_boundary_tool.tmpl                                                  │
│     ├─ task_artifact.tmpl                                                       │
│     └─ implementation_plan_artifact.tmpl                                        │
│                                                                                  │
│  9. FILE DIFFS ARTIFACT (STATIC + DYNAMIC)                                      │
│     └─ How to format and display file changes                                   │
│                                                                                  │
│ 10. ARTIFACT FORMATTING GUIDELINES (STATIC)                                     │
│     └─ Markdown format, embedding rules, file links                             │
│                                                                                  │
│ 11. KNOWLEDGE DISCOVERY (DYNAMIC - KI injection)                                │
│     └─ Available Knowledge Items for this context                               │
│                                                                                  │
│ 12. EPHEMERAL MESSAGE (DYNAMIC - per-request)                                   │
│     └─ CRITICAL REMINDERS, real-time context                                    │
│                                                                                  │
│ 13. CONVERSATION LOGS (DYNAMIC - chat history)                                  │
│     └─ Previous conversation context                                            │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Core Agent Workflow

### High-Level State Machine

```
                    ┌─────────────────────────────────────────┐
                    │              USER REQUEST               │
                    └───────────────────┬─────────────────────┘
                                        │
                                        ▼
                    ┌─────────────────────────────────────────┐
                    │         IS THIS SIMPLE WORK?            │
                    │  (questions, quick refactors, <3 files) │
                    └───────────────────┬─────────────────────┘
                                        │
                         YES ───────────┴─────────── NO
                          │                          │
                          ▼                          ▼
              ┌─────────────────────┐   ┌─────────────────────────┐
              │   DIRECT RESPONSE   │   │   CALL task_boundary    │
              │  (No task boundary) │   │   Enter PLANNING mode   │
              └─────────────────────┘   └───────────┬─────────────┘
                                                    │
                                                    ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              PLANNING MODE                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ALLOWED ACTIONS:                                                               │
│  ✓ Create/update task.md                                                        │
│  ✓ Create implementation_plan.md                                                │
│  ✓ Research codebase (view_file, grep_search, list_directory)                   │
│  ✓ Read documentation                                                           │
│  ✓ Define features and scope                                                    │
│                                                                                  │
│  EXIT CONDITIONS:                                                               │
│  • Implementation plan approved by user                                         │
│  • Research complete, ready to implement                                        │
│  • Call task_boundary with Mode="EXECUTION"                                     │
│                                                                                  │
│  REQUIRED BEFORE EXIT:                                                          │
│  → If plan created: notify_user for review, wait for approval                   │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        │ User approves / Ready
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                             EXECUTION MODE                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ALLOWED ACTIONS:                                                               │
│  ✓ code_action (edit files)                                                     │
│  ✓ write_to_file (create files)                                                 │
│  ✓ run_command (terminal commands)                                              │
│  ✓ delete_directory                                                             │
│  ✓ git_commit                                                                   │
│  ✓ browser_subagent (for testing)                                               │
│                                                                                  │
│  EXIT CONDITIONS:                                                               │
│  • All implementation complete                                                  │
│  • Ready for verification                                                       │
│  • Call task_boundary with Mode="VERIFICATION"                                  │
│                                                                                  │
│  BEHAVIOR:                                                                      │
│  → Proactively execute without asking permission                                │
│  → "The following changes were made by %v. If relevant, proactively run         │
│     terminal commands to execute this code for the USER. Don't ask permission." │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        │ Implementation complete
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           VERIFICATION MODE                                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ALLOWED ACTIONS:                                                               │
│  ✓ Run linters (lint_diff)                                                      │
│  ✓ Run tests (run_command)                                                      │
│  ✓ Static analysis                                                              │
│  ✓ Browser verification (capture_screenshot)                                    │
│  ✓ Build verification                                                           │
│                                                                                  │
│  IF ERRORS FOUND:                                                               │
│  → Return to EXECUTION mode to fix                                              │
│  → "Fix lint errors", "Fix test failures"                                       │
│                                                                                  │
│  EXIT CONDITIONS:                                                               │
│  • All checks pass                                                              │
│  • Call notify_user to inform completion                                        │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        │ All checks pass
                                        ▼
                    ┌─────────────────────────────────────────┐
                    │             TASK COMPLETE               │
                    │         (Return to IDLE state)          │
                    └─────────────────────────────────────────┘
```

---

## Tool Execution Workflow

### Tool Call Lifecycle

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          TOOL CALL LIFECYCLE                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│   MODEL RESPONSE                                                                │
│   ├─ Contains: [Tool Call A, Tool Call B, Tool Call C]                          │
│   │                                                                              │
│   ▼                                                                              │
│   FOR EACH TOOL CALL:                                                           │
│   │                                                                              │
│   ├─→ CHECK: waitForPreviousTools == true?                                      │
│   │      │                                                                       │
│   │      ├─ NO  → Execute IMMEDIATELY (parallel with others)                    │
│   │      │                                                                       │
│   │      └─ YES → Wait for ALL previous tools to complete first                 │
│   │                                                                              │
│   ▼                                                                              │
│   STEP STATUS PROGRESSION:                                                      │
│   │                                                                              │
│   │   GENERATING → QUEUED → PENDING → RUNNING → WAITING → DONE                  │
│   │                    │         │        │         │        │                   │
│   │                    └─────────┴────────┴─────────┘        │                   │
│   │                              │                           │                   │
│   │                              └─────────────────────────→ ├─→ ERROR           │
│   │                                                          ├─→ CANCELED        │
│   │                                                          ├─→ INVALID         │
│   │                                                          └─→ INTERRUPTED     │
│   │                                                                              │
│   ▼                                                                              │
│   RESULT RETURNED TO MODEL                                                      │
│   └─ Wrapped in <tool_response></tool_response> XML tags                        │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Cortex Step Types (Complete List)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        ALL CORTEX STEP TYPES (85+)                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  FILE OPERATIONS:                                                               │
│  ├─ VIEW_FILE              - Read file contents                                 │
│  ├─ VIEW_FILE_OUTLINE      - Get file structure/symbols                         │
│  ├─ VIEW_CODE_ITEM         - View specific code element                         │
│  ├─ VIEW_CONTENT_CHUNK     - Paginated content viewing                          │
│  ├─ CODE_ACTION            - Edit file (search/replace)                         │
│  ├─ WRITE_TO_FILE          - Create new file (via write_to_file tool)          │
│  ├─ DELETE_DIRECTORY       - Remove files/directories                          │
│  ├─ LIST_DIRECTORY         - List directory contents                           │
│  ├─ FIND                   - Find files by pattern                             │
│  └─ FILE_CHANGE            - Track file modifications                          │
│                                                                                  │
│  SEARCH OPERATIONS:                                                             │
│  ├─ GREP_SEARCH            - Search file contents                               │
│  ├─ CODE_SEARCH            - Semantic code search                               │
│  ├─ SEARCH_WEB             - Web search                                         │
│  ├─ INTERNAL_SEARCH        - Internal knowledge search                          │
│  └─ MQUERY                 - Multi-query search                                 │
│                                                                                  │
│  TERMINAL OPERATIONS:                                                           │
│  ├─ RUN_COMMAND            - Execute shell command                              │
│  ├─ COMMAND_STATUS         - Check command status                               │
│  ├─ READ_TERMINAL          - Read terminal output                               │
│  ├─ SEND_COMMAND_INPUT     - Send input to running command                      │
│  └─ SHELL_EXEC             - Direct shell execution                             │
│                                                                                  │
│  BROWSER OPERATIONS:                                                            │
│  ├─ BROWSER_SUBAGENT       - Delegate to browser subagent                       │
│  ├─ BROWSER_GET_DOM        - Get page DOM                                       │
│  ├─ BROWSER_CLICK_ELEMENT  - Click DOM element                                  │
│  ├─ CLICK_BROWSER_PIXEL    - Click at coordinates                               │
│  ├─ BROWSER_INPUT          - Type text                                          │
│  ├─ BROWSER_PRESS_KEY      - Press keyboard key                                 │
│  ├─ BROWSER_SCROLL_UP/DOWN - Scroll page                                        │
│  ├─ BROWSER_MOVE_MOUSE     - Move mouse cursor                                  │
│  ├─ BROWSER_MOUSE_WHEEL    - Mouse wheel scroll                                 │
│  ├─ BROWSER_SELECT_OPTION  - Select dropdown option                             │
│  ├─ BROWSER_RESIZE_WINDOW  - Resize browser window                              │
│  ├─ BROWSER_DRAG_PIXEL_TO_PIXEL - Drag operation                                │
│  ├─ CAPTURE_BROWSER_SCREENSHOT - Take screenshot                                │
│  ├─ CAPTURE_BROWSER_CONSOLE_LOGS - Get console logs                             │
│  ├─ EXECUTE_BROWSER_JAVASCRIPT - Run JS in browser                              │
│  ├─ READ_BROWSER_PAGE      - Read page content                                  │
│  ├─ LIST_BROWSER_PAGES     - List open pages                                    │
│  ├─ OPEN_BROWSER_URL       - Navigate to URL                                    │
│  └─ READ_URL_CONTENT       - Fetch URL content                                  │
│                                                                                  │
│  TASK MANAGEMENT:                                                               │
│  ├─ TASK_BOUNDARY          - Switch modes, update status                        │
│  ├─ NOTIFY_USER            - Request user attention                             │
│  ├─ CHECKPOINT             - Save conversation state                            │
│  └─ MEMORY                 - Store/retrieve memories                            │
│                                                                                  │
│  CODE QUALITY:                                                                  │
│  ├─ LINT_DIFF              - Lint changed files                                 │
│  ├─ LINT_APPLET            - Lint applet code                                   │
│  ├─ COMPILE                - Compile code                                       │
│  ├─ COMPILE_APPLET         - Compile applet                                     │
│  └─ CODE_ACKNOWLEDGEMENT   - Acknowledge code review                            │
│                                                                                  │
│  GIT OPERATIONS:                                                                │
│  ├─ GIT_COMMIT             - Create git commit                                  │
│  └─ POST_PR_REVIEW         - Post PR review comment                             │
│                                                                                  │
│  KNOWLEDGE SYSTEM:                                                              │
│  ├─ KNOWLEDGE_GENERATION   - Create Knowledge Items                             │
│  ├─ KNOWLEDGE_ARTIFACTS    - Manage KI artifacts                                │
│  ├─ BRAIN_UPDATE           - Update memory/knowledge                            │
│  └─ RETRIEVE_MEMORY        - Retrieve stored memories                           │
│                                                                                  │
│  MEDIA:                                                                         │
│  └─ GENERATE_IMAGE         - Generate images via Imagen                         │
│                                                                                  │
│  MCP (Model Context Protocol):                                                  │
│  ├─ MCP_TOOL               - Call MCP tool                                      │
│  ├─ LIST_RESOURCES         - List MCP resources                                 │
│  └─ READ_RESOURCE          - Read MCP resource                                  │
│                                                                                  │
│  SPECIAL:                                                                       │
│  ├─ USER_INPUT             - User message                                       │
│  ├─ SYSTEM_MESSAGE         - System injection                                   │
│  ├─ EPHEMERAL_MESSAGE      - Temporary context                                  │
│  ├─ ERROR_MESSAGE          - Error reporting                                    │
│  ├─ FINISH                 - End conversation                                   │
│  └─ DUMMY                  - No-op for testing                                  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Task Boundary Tool - The Mode Switch

The `task_boundary` tool is the PRIMARY mechanism for mode transitions:

```json
{
  "TaskName": "Planning Authentication",    // UI header
  "Mode": "PLANNING",                       // PLANNING | EXECUTION | VERIFICATION
  "TaskSummary": "Analyzed codebase...",    // Past tense - what's done
  "TaskStatus": "Researching auth libs",    // Future tense - what's next
  "PredictedTaskSize": 15                   // Estimated remaining tool calls
}
```

### Mode Transition Rules

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         MODE TRANSITION RULES                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  PLANNING → EXECUTION:                                                          │
│  ├─ Trigger: Plan approved OR research complete                                 │
│  ├─ Required: implementation_plan.md created (if complex task)                  │
│  └─ Action: task_boundary(Mode="EXECUTION", TaskSummary="Approved plan...")     │
│                                                                                  │
│  EXECUTION → VERIFICATION:                                                      │
│  ├─ Trigger: All code changes complete                                          │
│  ├─ Required: Implementation matches plan                                       │
│  └─ Action: task_boundary(Mode="VERIFICATION", TaskSummary="Implemented...")    │
│                                                                                  │
│  VERIFICATION → EXECUTION (backtrack):                                          │
│  ├─ Trigger: Lint errors, test failures, build errors                           │
│  ├─ Required: Identify what needs fixing                                        │
│  └─ Action: task_boundary(Mode="EXECUTION", TaskSummary="Found issues...")      │
│                                                                                  │
│  ANY → PLANNING (re-plan):                                                      │
│  ├─ Trigger: Major scope change, new requirements                               │
│  ├─ Required: User feedback requiring new plan                                  │
│  └─ Action: task_boundary(Mode="PLANNING", TaskSummary="Scope changed...")      │
│                                                                                  │
│  SPECIAL VALUE: "%SAME%"                                                        │
│  └─ Keeps current TaskName/Mode/Summary unchanged, only updates TaskStatus      │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Cascade Pipeline Status

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         CASCADE RUN STATUS                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│                     ┌────────┐                                                  │
│                     │  IDLE  │ ←─────────────────────────┐                      │
│                     └───┬────┘                           │                      │
│                         │                                │                      │
│                    User Request                          │                      │
│                         │                                │                      │
│                         ▼                                │                      │
│                   ┌──────────┐                           │                      │
│                   │ RUNNING  │ ──── User Cancel ────→ ┌──────────┐             │
│                   └────┬─────┘                        │CANCELING │             │
│                        │                              └────┬─────┘             │
│                   Processing                               │                    │
│                        │                                   │                    │
│                        ▼                                   │                    │
│                   ┌────────┐                               │                    │
│                   │  BUSY  │ ────────────────────────────→─┘                    │
│                   └────────┘                                                    │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Executor Termination Reasons

The agent stops when:

| Reason | Description |
|--------|-------------|
| `ERROR` | Tool execution failed |
| `USER_CANCELED` | User pressed cancel |
| `MAX_INVOCATIONS` | Hit tool call limit |
| `MAX_FORCED_INVOCATIONS` | Hit forced limit |
| `NO_TOOL_CALL` | Model didn't call any tools |
| `TERMINAL_STEP_TYPE` | Called notify_user or similar |
| `TERMINAL_CUSTOM_HOOK` | Custom hook triggered stop |
| `EARLY_CONTINUE` | Early continuation triggered |

---

## Checkpoint System (Context Overflow)

When context gets too long:

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         CHECKPOINT SYSTEM                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  TRIGGER: Token count > max_token_limit (45K-160K depending on model/tier)      │
│                                                                                  │
│  PROCESS:                                                                       │
│  1. Create CORTEX_STEP_TYPE_CHECKPOINT step                                     │
│  2. Summarize conversation with checkpoint_model (usually Gemini Flash)         │
│  3. Truncate older messages                                                     │
│  4. Inject summary as context                                                   │
│  5. Continue with reduced context                                               │
│                                                                                  │
│  CONFIG:                                                                        │
│  {                                                                              │
│    "max_token_limit": "128000",        // When to trigger                       │
│    "token_threshold": "50000",          // Target after truncation              │
│    "max_steps_per_checkpoint": 3,       // Steps between checkpoints            │
│    "checkpoint_model": "GEMINI_2_5_FLASH"  // Model for summarization           │
│  }                                                                              │
│                                                                                  │
│  SKIP DURING CHECKPOINT:                                                        │
│  • Previous checkpoint steps                                                    │
│  • Memory steps                                                                 │
│  • Ephemeral message steps                                                      │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Background Agent Mode

When user enables background execution:

```
"You are operating as a **background agent** that runs automatically after 
conversations to extract and preserve knowledge."

"This means that the USER has submitted their request and is not actively 
monitoring your work. They will only check back when you are completely done."

"You won't be able to ask clarifying questions, just write your thinking 
and use tools. You're on your own now."
```

---

## Critical Rules (Ephemeral Reminders)

Injected as `<EPHEMERAL_MESSAGE>` during execution:

1. **Aesthetics**: "If your web app looks simple and basic then you have FAILED!"
2. **Conciseness**: "Artifacts should be AS CONCISE AS POSSIBLE"
3. **Task Status**: "TaskStatus should describe NEXT STEPS, NOT previous steps"
4. **Browser Verification**: "NEVER trust the subagent's claims. IMMEDIATELY verify the screenshot"
5. **Tool Order**: "Call this tool ONLY when you have completed all other tool calls"

---

## Complete Request-Response Cycle

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      COMPLETE REQUEST-RESPONSE CYCLE                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  1. USER INPUT                                                                  │
│     └─→ Chat UI receives message                                                │
│                                                                                  │
│  2. EXTENSION PROCESSING                                                        │
│     ├─→ Extension.js processes input                                           │
│     └─→ Sends to Language Server via LSP                                        │
│                                                                                  │
│  3. LANGUAGE SERVER                                                             │
│     ├─→ Constructs cumulative prompt (13 sections in order)                     │
│     ├─→ Injects dynamic context (KIs, memories, workspace state)                │
│     ├─→ Checks token limits, creates checkpoint if needed                       │
│     └─→ Sends to Cloud API                                                      │
│                                                                                  │
│  4. CLOUD API (daily-cloudcode-pa.googleapis.com)                               │
│     ├─→ Routes to appropriate Gemini model                                      │
│     ├─→ Processes with Cortex engine                                            │
│     └─→ Returns CortexStep response(s)                                          │
│                                                                                  │
│  5. STEP EXECUTION                                                              │
│     ├─→ For each CortexStep:                                                    │
│     │   ├─→ Parse tool call                                                     │
│     │   ├─→ Check waitForPreviousTools                                          │
│     │   ├─→ Execute tool (parallel or sequential)                               │
│     │   └─→ Collect result                                                      │
│     └─→ Package results as tool_response                                        │
│                                                                                  │
│  6. CONTINUATION                                                                │
│     ├─→ If more tool calls expected: goto step 3                                │
│     ├─→ If TERMINAL_STEP_TYPE: stop and show result                             │
│     └─→ If error: show error and potentially retry                              │
│                                                                                  │
│  7. USER OUTPUT                                                                 │
│     └─→ Display response in Chat UI                                             │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Key Extracted Prompts

### Core Identity
```
You are Antigravity, a powerful agentic AI coding assistant designed by the 
Google Deepmind team working on Advanced Agentic Coding.
```

### Agentic Mode
```
You are in AGENTIC mode. You should take more time to research and think 
deeply about the given task, which will usually be more complex. Instead of 
bouncing ideas back and forth with the user very frequently, AGENTIC mode 
means you should interact less often and do as much work independently as 
you can in between interactions. As much as possible, you should also verify 
your work before presenting it to the user.
```

### Pair Programming
```
You are pair programming with a USER to solve their coding task. The task 
may require creating a new codebase, modifying or debugging an existing 
codebase, or simply answering a question.
```

### Tool Calling
```
You are a tool calling agent. You are provided with function signatures 
within <tools> </tools> XML tags. You may call one or more functions to 
assist with the user query. After calling & executing the functions, you 
will be provided with function results within <tool_response> </tool_response> 
XML tags.
```

### Knowledge Items Specialist
```
You are a Knowledge Items (KI) specialist responsible for managing the 
knowledge base. Your role encompasses three key responsibilities:
[discovery, consolidation, usage]
```

---

## Extraction Completeness: ~95%

| Component | Status | Confidence |
|-----------|--------|------------|
| Core Identity Prompt | ✅ Complete | 100% |
| Mode Definitions | ✅ Complete | 100% |
| All 85+ Step Types | ✅ Complete | 100% |
| Tool Schemas | ✅ Complete | 95% |
| Workflow State Machine | ✅ Complete | 100% |
| Prompt Assembly Order | ✅ Complete | 95% |
| Checkpoint System | ✅ Complete | 90% |
| Model Routing | ✅ Complete | 90% |
| Ephemeral Messages | ✅ Complete | 85% |
| Template Variables | ✅ Complete | 90% |

---

*Generated: January 2, 2026*
