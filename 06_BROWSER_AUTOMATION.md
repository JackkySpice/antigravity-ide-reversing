# Browser Automation

## Overview

AntiGravity can control a browser for web testing, verification, and automation tasks. All browser interactions are automatically recorded and saved as WebP videos to the artifacts directory.

---

## Browser Tools

| Tool | Purpose |
|------|---------|
| `open_browser_url` | Open/navigate to URL |
| `browser_get_dom` | Get annotated DOM with element indices |
| `capture_browser_screenshot` | Capture viewport or element screenshot |
| `click_browser_pixel` | Click at exact pixel coordinates |
| `browser_click_element` | Click element by DOM index |
| `browser_input` | Type text or press keys |
| `browser_scroll_up` | Scroll the page up |
| `browser_scroll_down` | Scroll the page down |
| `browser_select_option` | Select dropdown option |
| `browser_move_mouse` | Move mouse cursor |
| `browser_drag_pixel_to_pixel` | Drag between coordinates |
| `capture_browser_console_logs` | Get console output |
| `read_browser_page` | Read page content |

---

## Browser Subagent

### Description

Start a browser subagent to perform actions in the browser with the given task description. The subagent has access to tools for both interacting with web page content (clicking, typing, navigating, etc) and controlling the browser window itself (resizing, etc).

Please make sure to define a clear condition to return on. After the subagent returns, you should read the DOM or capture a screenshot to see what it did.

> **Note:** All browser interactions are automatically recorded and saved as WebP videos to the artifacts directory. This is the ONLY way you can record a browser session video/animation.

> **Important:** If the subagent returns that the `open_browser_url` tool failed, there is a browser issue that is out of your control. You MUST ask the user how to proceed and use the `suggested_responses` tool.

### Subagent Schema

```json
{
  "TaskName": {
    "type": "string",
    "description": "Name of the task that the browser subagent is performing. Should be properly capitalized and human readable.",
    "example": "Navigating to Example Page"
  },
  "Task": {
    "type": "string",
    "description": "A clear, actionable task description for the browser subagent. Since each agent invocation is a one-shot, autonomous execution, the prompt must be highly detailed."
  },
  "RecordingName": {
    "type": "string",
    "description": "Name of the browser recording. Should be all lowercase with underscores, max 3 words.",
    "example": "login_flow_demo"
  },
  "waitForPreviousTools": {
    "type": "boolean",
    "description": "If true, wait for all previous tool calls to complete before executing."
  }
}
```

### Subagent Verification

> **CRITICAL:** NEVER trust the subagent's claims. After a browser subagent completes a task, IMMEDIATELY verify the screenshot BEFORE responding to the user. Look at the actual screenshot content and describe what you see. If the screenshot doesn't show the expected result, acknowledge that the task may not have completed successfully and investigate further.

### Subagent Output

You are shown COMPLETE details of every action the browser subagent performed. The only exception is if the output of any JavaScript executed by the subagent shows that the browser subagent successfully performed the action.

---

## Page Loading Guidance

If a page looks like it's loading:

1. Call the DOM tool again and see if it updates
2. Take a screenshot again and see if it updates
3. Sometimes it just takes a moment for the page to load

---

## Recording Capabilities

### Automatic Recording Features

All browser interactions are automatically recorded and saved as WebP videos to the artifacts directory.

### What Gets Recorded

- All navigation actions
- Click interactions
- Text input
- Scrolling behavior
- Mouse movements
- Drag operations

### Recording Behavior

- Recordings are saved automatically
- Output format is WebP video
- Files are stored in the artifacts directory
- This is the ONLY way to record a browser session video/animation

---

## Safety Rules

| Rule | Description |
|------|-------------|
| **No Captcha Solving** | Do not try to solve captchas or get around bot detection mechanisms |
| **No Unauthorized Access** | NEVER attempt to access restricted areas, private accounts, or bypass authentication systems without explicit user authorization |
| **Privacy Protection** | Do not perform actions that could compromise user privacy or security |
| **No Suspicious Downloads** | Do not attempt to download suspicious files or execute potentially harmful content |
| **Credential Safety** | If a website requests personal information, credentials, or payment details, do not proceed and inform the user of the security implications |
