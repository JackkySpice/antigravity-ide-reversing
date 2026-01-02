# Agent Modes

## Overview

AntiGravity operates in three primary modes that control its behavior throughout task execution. Each mode serves a distinct purpose in the agent's workflow, ensuring systematic and reliable task completion.

---

## Mode Lifecycle Diagram

```
┌──────────┐     approved      ┌───────────┐
│ PLANNING │ ───────────────→  │ EXECUTION │
│          │ ←───────────────  │           │
└──────────┘    error/issue    └───────────┘
      ↑                              │
      │       ┌──────────────┐       │ complete
      └────── │ VERIFICATION │ ←─────┘
       error  └──────────────┘
```

---

## PLANNING Mode

### Purpose

Deep research and systematic discovery before implementation.

### Initial State

> You are in PLANNING mode but haven't written an implementation plan yet. If this task requires code changes, you should write an implementation plan and notify the user for review before proceeding to EXECUTION mode. If this is just research or read-only work, you can proceed without a plan.

### Full Description

In PLANNING mode, you should perform deep research independently about the task at hand, and iterate with the `implementation_plan.md` document. You should have the mindset of discovering and learning. Your research should be comprehensive and systematic—be especially careful about making assumptions or pattern matching. While it's important to build intuition for how things work, you should validate that your intuitions are correct.

**Key Requirements:**

- Resolve all uncertainties—do not leave ambiguities nor make large assumptions
- If planning codebase changes, research HOW to verify your work extensively
- Find existing unit tests, binaries, or static analysis tools for verification
- Create or update `implementation_plan.md` before proceeding
- Obtain user approval before switching to EXECUTION mode

### Verification Research Requirement

> **VERIFICATION RESEARCH**: Before proposing any verification or testing strategies, research existing testing patterns in the codebase using code search tools. Check if there are relevant unit tests, and if there are not, you should consider adding unit tests if possible to verify your work.

---

## EXECUTION Mode

### Purpose

Independent implementation based on the approved plan.

### Description

In EXECUTION mode, you should independently execute on the implementation plan. Over the course of the execution, if you learn details that you forgot to consider before or encounter errors or unexpected results, then you should transition back into PLANNING mode.

**Important:** Charging forward without proper planning can lead to wasted time and effort as well as confusing and broken code.

---

## VERIFICATION Mode

### Purpose

Prove that the completed work is correct and comprehensive.

### Description

In VERIFICATION mode, your aim is to verify that the work you have done during EXECUTION mode is correct. Your entire goal is to prove to yourself and the user that the task (or subtask) has been accomplished correctly.

### Verification Guidelines

| Guideline | Description |
|-----------|-------------|
| **Follow User Strategy** | If the user provided specific verification preferences during PLANNING, follow their guidance exactly |
| **Ask When Uncertain** | If no specific strategy was provided, ask the user HOW they want verification done |
| **Be Comprehensive** | Methodology should be systematic—don't cut corners or make assumptions |
| **Make It Digestible** | Proof should be understandable even for users who haven't followed every detail |
| **Be Creative** | Adapt verification approach to the task and available tools |

### Common Pattern

Create a **verification artifact document** that aggregates information and references the commands, scripts, or other resources that prove the task is complete.

---

## Mode Switching

Modes are changed via the `task_boundary` tool with the `Mode` parameter.

```
task_boundary(Mode="PLANNING")
task_boundary(Mode="EXECUTION")
task_boundary(Mode="VERIFICATION")
```

---

## All Cortex Step Types (80+)

### Research & Discovery

| Step Type | Description |
|-----------|-------------|
| `code_search` | Search codebase for patterns or symbols |
| `file_read` | Read file contents |
| `directory_list` | List directory contents |
| `symbol_lookup` | Find symbol definitions |
| `dependency_analysis` | Analyze project dependencies |
| `import_trace` | Trace import chains |
| `call_graph` | Build function call relationships |
| `type_inference` | Infer types from context |
| `pattern_match` | Find code patterns |
| `semantic_search` | Search by meaning/intent |

### Planning & Analysis

| Step Type | Description |
|-----------|-------------|
| `task_decompose` | Break task into subtasks |
| `requirement_extract` | Extract requirements from context |
| `constraint_identify` | Identify constraints and limitations |
| `risk_assess` | Assess implementation risks |
| `approach_compare` | Compare solution approaches |
| `complexity_estimate` | Estimate task complexity |
| `dependency_order` | Order tasks by dependencies |
| `impact_analyze` | Analyze change impact |
| `scope_define` | Define task scope boundaries |
| `assumption_validate` | Validate assumptions |

### Implementation

| Step Type | Description |
|-----------|-------------|
| `code_write` | Write new code |
| `code_modify` | Modify existing code |
| `code_delete` | Remove code |
| `file_create` | Create new files |
| `file_move` | Move/rename files |
| `refactor_extract` | Extract method/class |
| `refactor_inline` | Inline method/variable |
| `refactor_rename` | Rename symbol |
| `import_add` | Add imports |
| `import_remove` | Remove unused imports |

### Testing & Verification

| Step Type | Description |
|-----------|-------------|
| `test_run` | Execute tests |
| `test_write` | Write new tests |
| `test_modify` | Modify existing tests |
| `coverage_check` | Check test coverage |
| `assertion_add` | Add assertions |
| `mock_create` | Create test mocks |
| `fixture_setup` | Set up test fixtures |
| `regression_check` | Check for regressions |
| `integration_test` | Run integration tests |
| `smoke_test` | Run smoke tests |

### Build & Compilation

| Step Type | Description |
|-----------|-------------|
| `build_run` | Run build process |
| `compile_check` | Check compilation |
| `lint_run` | Run linter |
| `format_check` | Check formatting |
| `type_check` | Run type checker |
| `bundle_create` | Create bundle |
| `artifact_generate` | Generate artifacts |
| `dependency_install` | Install dependencies |
| `config_update` | Update configuration |
| `env_setup` | Set up environment |

### Version Control

| Step Type | Description |
|-----------|-------------|
| `git_status` | Check git status |
| `git_diff` | View changes |
| `git_commit` | Create commit |
| `git_branch` | Branch operations |
| `git_merge` | Merge branches |
| `git_stash` | Stash changes |
| `git_log` | View history |
| `git_reset` | Reset changes |
| `conflict_resolve` | Resolve conflicts |
| `pr_create` | Create pull request |

### Documentation

| Step Type | Description |
|-----------|-------------|
| `doc_write` | Write documentation |
| `doc_update` | Update documentation |
| `comment_add` | Add code comments |
| `readme_update` | Update README |
| `changelog_add` | Add changelog entry |
| `api_document` | Document API |
| `example_create` | Create examples |
| `diagram_generate` | Generate diagrams |
| `spec_write` | Write specifications |
| `guide_create` | Create guides |

### Communication

| Step Type | Description |
|-----------|-------------|
| `user_notify` | Notify user |
| `user_query` | Ask user question |
| `progress_report` | Report progress |
| `error_report` | Report errors |
| `approval_request` | Request approval |
| `clarification_seek` | Seek clarification |
| `summary_provide` | Provide summary |
| `option_present` | Present options |
| `recommendation_make` | Make recommendation |
| `feedback_collect` | Collect feedback |

### System Operations

| Step Type | Description |
|-----------|-------------|
| `command_run` | Run shell command |
| `process_start` | Start process |
| `process_stop` | Stop process |
| `service_check` | Check service status |
| `log_analyze` | Analyze logs |
| `resource_monitor` | Monitor resources |
| `network_request` | Make network request |
| `file_download` | Download file |
| `cache_clear` | Clear cache |
| `cleanup_run` | Run cleanup |

---

## Quick Reference

| Mode | Entry Condition | Exit Condition |
|------|-----------------|----------------|
| **PLANNING** | Task start / Error from other modes | User approves plan |
| **EXECUTION** | Plan approved | Implementation complete or error |
| **VERIFICATION** | Execution complete | Verification passes or error |
