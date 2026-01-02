# Knowledge Item (KI) System

## Overview

Knowledge Items are persistent, curated knowledge units that store insights across conversations. They serve as the system's long-term memory, enabling continuity and preventing redundant work.

---

## KI Structure

```
~/.gemini/antigravity/
├── knowledge/
│   ├── {ki_id}/
│   │   ├── metadata.yaml
│   │   │   ├── title: "Human-readable title"
│   │   │   ├── summary: "One paragraph summary"
│   │   │   └── references: [...]
│   │   └── artifacts/
│   │       ├── file1.md
│   │       └── file2.txt
```

---

## KI Discovery Workflow

### Step 1: Search for Existing KIs

```
Search for existing related KIs before creating new ones.
```

### Step 2: Choose Action

Choose **ONE** of the following based on search results:

| Action | Condition | Description |
|--------|-----------|-------------|
| **A. CONSOLIDATE** | Multiple relevant KIs found | Merge related KIs into one |
| **B. UPDATE** | Exactly one relevant KI found | Add new information to existing KI |
| **C. CREATE** | No relevant KIs found | Create a new KI |

---

## Mandatory KI Checks

> **At the start of each conversation, you receive KI summaries with artifact paths.** These summaries exist precisely to help you avoid redundant work.

**BEFORE performing ANY research, analysis, or creating documentation, you MUST:**

1. **Review the KI summaries** already provided to you at conversation start
2. **Identify relevant KIs** by checking if any KI titles/summaries match your task
3. **Read relevant KI artifacts** using the artifact paths listed in the summaries BEFORE doing independent research
4. **Build upon KIs** by using the information from the KIs to inform your own research

---

## When to Check KIs

### Must Check Scenarios

- Before ANY research or analysis
- Before creating documentation
- When KI title matches request
- When encountering new concepts
- Before debugging unexpected behavior
- When experiencing resource issues
- Before designing "new" features
- When implementing common functionality

### Search Tool Guidance

| Tool | Best For |
|------|----------|
| `trajectory_search` | Discovery; uses LLM to evaluate relevance |
| `grep_search` | Exact text matching |
| `list_dir` | File/directory name patterns |
| **Combine tools** | Use multiple tools in parallel for comprehensive search |

---

## KI Trust Guidelines

> **CRITICAL:** KIs are snapshots from past work. They are valuable starting points, but **NOT** a substitute for independent research and verification.

- **Always verify:** Use the references in metadata to check original sources
- **Expect gaps:** KIs may not cover all aspects—supplement with your own investigation
- **Question everything:** Treat KIs as clues that must be verified and supplemented

---

## Consolidation

**What is consolidation?**
Merging content from multiple related KIs into a single target KI.

**When to consolidate:**
KIs cover the same component, feature, or concept.

**How to consolidate:**

1. Choose a target KI (usually the most comprehensive one)
2. Merge content from source KIs into target
3. Delete source KIs after successful merge

---

## Conversation Logs and Artifacts

You can access the original, raw information from past conversations through the corresponding conversation logs, as well as the ASSISTANT-generated artifacts within the conversation, through the filesystem.

### When to Use

Read the conversation logs when you need the details of the conversation, and there are a small number of relevant conversations to study.

### When NOT to Use

Do not read the conversation logs if:

- They are likely to be irrelevant to the current conversation
- They are likely to contain more information than necessary
