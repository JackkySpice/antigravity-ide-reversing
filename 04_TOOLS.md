# Tools Reference

AntiGravity provides **43+ tools** organized into 10 functional categories for comprehensive development automation.

---

## Tool Categories at a Glance

| Category | Tools | Primary Use |
|----------|-------|-------------|
| [File Operations](#1-file-operations) | 7 | Read, write, edit files |
| [Search Operations](#2-search-operations) | 4 | Find code and text |
| [Browser Automation](#3-browser-automation) | 14 | Web interaction and testing |
| [Terminal Operations](#4-terminal-operations) | 3 | Command execution |
| [Web/Network](#5-webnetwork) | 3 | Fetch external content |
| [MCP](#6-mcp-model-context-protocol) | 3 | External tool integration |
| [Communication](#7-communication) | 2 | User interaction |
| [Code Operations](#8-code-operations) | 4 | Code changes and git |
| [Task Management](#9-task-management) | 2 | Workflow control |
| [Knowledge Management](#10-knowledge-management) | 3 | Learning and memory |

---

## 1. File Operations

Tools for reading, writing, and manipulating files in the workspace.

| Tool | Description |
|------|-------------|
| `view_file` | View file contents with optional line range |
| `view_file_outline` | View file structure/outline (classes, functions, etc.) |
| `list_directory` | List directory contents |
| `replace_file_content` | Single contiguous edit to a file |
| `multi_replace_file_content` | Multiple non-contiguous edits in one operation |
| `write_to_file` | Create new file or artifact |
| `delete_knowledge` | Delete Knowledge Item files/directories |

---

## 2. Search Operations

Tools for finding code, text, and references across the codebase.

| Tool | Description |
|------|-------------|
| `grep_search` | Text/regex search in files |
| `code_search` | Semantic code search with filters |
| `codebase_search` / `find` | Search across directories |
| `find_all_references` | LSP-based reference finding |

---

## 3. Browser Automation

Comprehensive tools for browser interaction, testing, and web automation.

| Tool | Description |
|------|-------------|
| `open_browser_url` | Open URL in browser |
| `browser_get_dom` | Get page DOM with element indices |
| `capture_browser_screenshot` | Screenshot viewport or specific element |
| `click_browser_pixel` | Click at pixel coordinates |
| `browser_click_element` | Click DOM element by index |
| `browser_input` | Keyboard input/typing |
| `browser_scroll_up` | Scroll page up |
| `browser_scroll_down` | Scroll page down |
| `browser_select_option` | Select dropdown option |
| `browser_move_mouse` | Move mouse cursor |
| `browser_drag_pixel_to_pixel` | Drag operation between coordinates |
| `capture_browser_console_logs` | Get browser console output |
| `read_browser_page` | Read page content as text |
| `browser_subagent` | Launch browser automation subagent |

---

## 4. Terminal Operations

Tools for executing commands and interacting with terminal sessions.

| Tool | Description |
|------|-------------|
| `run_command` | Execute shell command |
| `read_terminal` | Read terminal output |
| `send_command_input` | Send input to running command |

---

## 5. Web/Network

Tools for fetching and processing external web content.

| Tool | Description |
|------|-------------|
| `search_web` | Web search with citations |
| `read_url_content` | Fetch URL content |
| `view_content_chunk` | View chunk of large document |

---

## 6. MCP (Model Context Protocol)

Tools for integrating with external MCP servers and resources.

| Tool | Description |
|------|-------------|
| `mcp_tool` | Call MCP server tool |
| `list_resources` | List available MCP resources |
| `read_resource` | Read MCP resource content |

---

## 7. Communication

Tools for user interaction and response handling.

| Tool | Description |
|------|-------------|
| `notify_user` | Send message to user |
| `suggested_responses` | Provide response suggestions |

---

## 8. Code Operations

Tools for code modifications and version control.

| Tool | Description |
|------|-------------|
| `code_action` | Propose code changes |
| `git_commit` | Create git commit |
| `add_annotation` | Add code annotation |
| `run_extension_code` | Run TypeScript in VSCode context |

---

## 9. Task Management

Tools for controlling agent workflow and timing.

| Tool | Description |
|------|-------------|
| `task_boundary` | Control agent mode and progress |
| `wait` | Wait specified duration (max 30s) |

---

## 10. Knowledge Management

Tools for learning, memory, and external integrations.

| Tool | Description |
|------|-------------|
| `trajectory_search` | Search past conversations |
| `knowledge_generation` | Create Knowledge Items |
| `post_pr_review` | Post GitHub PR review |

---

## Key Tool Details

### browser_subagent

Launches a dedicated subagent for complex browser automation tasks.

**Parameters:**

```json
{
  "TaskName": "Name of the task (human readable title)",
  "Task": "Detailed task description for the subagent",
  "RecordingName": "lowercase_with_underscores (max 3 words)",
  "waitForPreviousTools": true
}
```

**Behavior:**
- Subagent has access to all browser interaction tools
- Can control both page content and browser window
- Define a clear return condition in the task description
- After return, read DOM or capture screenshot to verify results
- All interactions are automatically recorded as WebP videos to artifacts directory

---

### task_boundary

Controls agent mode transitions and task progress tracking.

> **IMPORTANT:** You must generate the following arguments first, before any others:
> - `TaskName`
> - `Mode`
> - `PredictedTaskSize`

---

### notify_user

Sends messages and notifications to the user.

> **IMPORTANT:** You must generate the following arguments first, before any others:
> - `PathsToReview`
> - `BlockedOnUser`

---

### replace_file_content

Performs a single contiguous edit to a file.

> **IMPORTANT:** The `TargetContent` of each `ReplacementChunk` MUST be a unique substring within the file. Otherwise, the tool call will error out.

**Best Practices:**
- Include enough context in `TargetContent` to ensure uniqueness
- For multiple edits, use `multi_replace_file_content` instead
- Verify the target string exists before attempting replacement

---

## Tool Selection Guide

| Task | Recommended Tool |
|------|------------------|
| Read a file | `view_file` |
| Find text in codebase | `grep_search` |
| Find code semantically | `code_search` |
| Make single edit | `replace_file_content` |
| Make multiple edits | `multi_replace_file_content` |
| Create new file | `write_to_file` |
| Run shell command | `run_command` |
| Automate browser | `browser_subagent` |
| Click specific element | `browser_click_element` |
| Search the web | `search_web` |
| Commit changes | `git_commit` |
| Store learned info | `knowledge_generation` |
