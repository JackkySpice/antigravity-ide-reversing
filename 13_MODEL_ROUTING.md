# Model Routing and Configuration

This document details the model selection, routing logic, fallback chains, and configuration extracted from AntiGravity IDE's language server.

## 1. Available Models

### Primary Chat/Agent Models

| Model ID | Display Label | Provider |
|----------|---------------|----------|
| `MODEL_GOOGLE_GEMINI_2_5_PRO` | Gemini 2.5 Pro | Google |
| `MODEL_GOOGLE_GEMINI_2_5_FLASH` | Gemini 2.5 Flash | Google |
| `MODEL_GOOGLE_GEMINI_2_5_FLASH_LITE` | Gemini 2.5 Flash Lite | Google |
| `MODEL_GOOGLE_GEMINI_2_5_FLASH_THINKING` | Gemini 2.5 Flash (Thinking) | Google |
| `MODEL_CLAUDE_4_SONNET` | Claude 4 Sonnet | Anthropic |
| `MODEL_CLAUDE_4_SONNET_THINKING` | Claude 4 Sonnet (Thinking) | Anthropic |
| `MODEL_CLAUDE_4_5_SONNET` | Claude Sonnet 4.5 | Anthropic |
| `MODEL_CLAUDE_4_5_SONNET_THINKING` | Claude Sonnet 4.5 (Thinking) | Anthropic |
| `MODEL_CLAUDE_3_5_HAIKU_20241022` | Claude 3.5 Haiku | Anthropic |
| `MODEL_CLAUDE_3_7_SONNET_20250219` | Claude 3.7 Sonnet | Anthropic |
| `MODEL_CHAT_GPT_4O_MINI_2024_07_18` | GPT-4o Mini | OpenAI |
| `MODEL_CHAT_O4_MINI_HIGH` | o4-mini High | OpenAI |
| `MODEL_OPENAI_GPT_OSS_120B_MEDIUM` | GPT-OSS 120B (Medium) | OpenAI |

### Placeholder Models (Future/Internal)

| Model ID | Display Label |
|----------|---------------|
| `MODEL_PLACEHOLDER_M7` | Gemini 3 Pro (Low) |
| `MODEL_PLACEHOLDER_M8` | Gemini 3 Pro (High) |
| `MODEL_PLACEHOLDER_M12` | Claude Opus 4.5 (Thinking) |
| `MODEL_PLACEHOLDER_M18` | Gemini 3 Flash |
| `MODEL_PLACEHOLDER_M19` | Supercomplete Gemini |

### Internal/Experimental Models

| Model ID | Purpose |
|----------|---------|
| `MODEL_GOOGLE_GEMINI_INTERNAL_BYOM` | Bring Your Own Model |
| `MODEL_GOOGLE_GEMINI_INTERNAL_TAB_FLASH_LITE` | Tab completion |
| `MODEL_GOOGLE_GEMINI_COMPUTER_USE_EXPERIMENTAL` | Computer use |
| `MODEL_CASCADE_20068` | Cascade base model |
| `MODEL_CASCADE_20070` | Vista/Shamu alias target |
| `MODEL_CASCADE_20072` | Vista variant |

## 2. Model Aliases

The system uses aliases to abstract model selection:

```
MODEL_ALIAS_CASCADE_BASE   → MODEL_CHAT_GPT_4O_MINI_2024_07_18 (free tier)
MODEL_ALIAS_VISTA          → MODEL_CASCADE_20070/20072 (weighted distribution)
MODEL_ALIAS_SHAMU          → MODEL_CASCADE_20070
MODEL_ALIAS_SWE_1          → SWE agent model
MODEL_ALIAS_SWE_1_LITE     → Lightweight SWE agent
MODEL_ALIAS_AUTO           → Auto-routing based on context
MODEL_ALIAS_RECOMMENDED    → MODEL_GOOGLE_GEMINI_2_5_PRO
```

## 3. Fallback and Checkpoint Configuration

### Checkpoint Model Strategy

When context exceeds `token_threshold`, the system switches to a lighter "checkpoint" model for summarization:

| Primary Model Category | Checkpoint Model |
|------------------------|------------------|
| Most SaaS models | `MODEL_GOOGLE_GEMINI_2_5_FLASH` or `MODEL_GOOGLE_GEMINI_2_5_FLASH_LITE` |
| Enterprise Claude models | `MODEL_CLAUDE_3_5_HAIKU_20241022` |
| Background Research | `MODEL_CHAT_GPT_4O_MINI_2024_07_18` |

### Fallback Flow

```
User Request
    ↓
Primary Model (e.g., Gemini 2.5 Pro)
    ↓
[Context > token_threshold?]
    ↓ YES
Checkpoint Model summarizes context
    ↓
Continue with Primary Model
```

## 4. Token Budget Allocation

### SaaS Tier Defaults

| Configuration | Value |
|---------------|-------|
| `max_token_limit` | 45,000 |
| `token_threshold` | 30,000 - 50,000 |
| `max_output_tokens` | 8,192 - 32,768 (model-dependent) |
| `truncation_threshold_tokens` | 100,000 - 160,000 |
| `max_overhead_ratio` | 0.15 |
| `moving_window_size` | 1 |

### Enterprise Tier

| Configuration | Value |
|---------------|-------|
| `max_token_limit` | 128,000 - 160,000 |
| `token_threshold` | 50,000 |
| `max_output_tokens` | 16,384 - 32,768 |
| `checkpoint_model` | `MODEL_CLAUDE_3_5_HAIKU_20241022` |

### Per-Model Output Token Limits

| Model | max_output_tokens |
|-------|-------------------|
| `MODEL_GOOGLE_GEMINI_2_5_PRO` | 16,384 |
| `MODEL_CLAUDE_4_SONNET` variants | 8,192 |
| `MODEL_CHAT_O4_MINI_HIGH` | 32,768 |
| Default | 8,192 |

## 5. Model Selection Routing

### Routing Context Variables

The system routes based on these context variables:

| Variable | Description |
|----------|-------------|
| `requestedModelId` | User-selected model ID |
| `cascadeEnterpriseMode` | Enterprise vs SaaS tier |
| `ide` | IDE variant (windsurf, windsurf-insiders, windsurf-next, jetski, antigravity) |
| `ideVersion` | Semantic version for feature gating |
| `devMode` | Developer mode enabled |
| `userTierId` | User's subscription tier |
| `hasAnthropicModelAccess` | Anthropic model access flag |
| `os` | Operating system |
| `extensionVersion` | Extension version |

### Feature Flag Routing (Unleash)

```json
{
  "name": "flexibleRollout",
  "constraints": [
    {
      "contextName": "requestedModelId",
      "operator": "IN",
      "values": ["MODEL_GOOGLE_GEMINI_2_5_PRO"]
    }
  ],
  "variants": [
    {
      "name": "default",
      "weight": 1000,
      "payload": { "type": "json", "value": "{...config...}" }
    }
  ],
  "stickiness": "userId" // or sessionId, installationId, random
}
```

## 6. Model-Specific Configurations

### CASCADE_PLAN_BASED_CONFIG_OVERRIDE

Per-model configuration overrides:

#### Gemini 2.5 Pro
```json
{
  "checkpoint_config": {
    "max_token_limit": "45000"
  },
  "planner_config": {
    "max_output_tokens": "16384",
    "truncation_threshold_tokens": "100000"
  }
}
```

#### o4-mini High
```json
{
  "checkpoint_config": {
    "max_token_limit": "45000"
  },
  "planner_config": {
    "max_output_tokens": "32768",
    "truncation_threshold_tokens": "100000"
  }
}
```

#### Claude Models (3.7, 4 Sonnet)
```json
{
  "checkpoint_config": {
    "max_token_limit": "45000"
  },
  "planner_config": {
    "max_output_tokens": "8192",
    "truncation_threshold_tokens": "100000"
  }
}
```

#### Enterprise Mode (Any Model)
```json
{
  "checkpoint_config": {
    "max_token_limit": "135000",
    "checkpoint_model": "MODEL_CLAUDE_3_5_HAIKU_20241022"
  },
  "planner_config": {
    "max_output_tokens": "8192"
  }
}
```

## 7. Feature Flags for Model Configuration

### Model Selection Flags

| Flag | Purpose |
|------|---------|
| `CASCADE_BASE_MODEL_ID` | Free tier model (GPT-4o Mini or Haiku) |
| `CASCADE_DEFAULT_MODEL_OVERRIDE` | Default model per IDE version |
| `recommended-model` | Recommended model (Gemini 2.5 Pro) |
| `vista-model-id` | Vista alias → CASCADE_20070/20072 |
| `shamu-model-id` | Shamu alias → CASCADE_20070 |

### Feature-Specific Model Flags

| Flag | Model Used |
|------|------------|
| `SUPERCOMPLETE_MODEL_CONFIG` | `MODEL_CHAT_23310` or `MODEL_PLACEHOLDER_M19` |
| `TAB_JUMP_MODEL_CONFIG` | `MODEL_CHAT_20706`, `MODEL_CHAT_15305`, etc. |
| `COMMAND_MODEL_CONFIG` | `MODEL_GOOGLE_GEMINI_2_5_FLASH` |
| `cascade-input-model-config` | `MODEL_CHAT_23310` |

### Access Control Flags

| Flag | Purpose |
|------|---------|
| `CASCADE_DEEPSEEK_R1_ACCESS` | DeepSeek R1 model access |
| `CASCADE_DEEPSEEK_V3_ACCESS` | DeepSeek V3 model access |
| `ANTHROPIC_ACCESS_ACL_GROUPS` | Anthropic access (gdm-ftes, labs-fte, jetski-has-anthropic-access) |

## 8. Model Capabilities

### Image Support

Models with image processing capabilities:

```
Supported MIME types:
- image/heic, image/heif, image/jpeg, image/png, image/webp
- video/jpeg2000, video/videoframe/jpeg2000
- application/pdf
```

**Image-capable models:**
- All Gemini models
- Claude 4/4.5 Sonnet variants
- Claude Opus 4.5

### Model Features Schema

```json
{
  "zero_shot_capable": true,
  "supports_images": true,
  "supports_tool_calls": true,
  "supports_thinking": true
}
```

### Tier Restrictions

Models are restricted by subscription tier:

```json
"allowedTiers": [
  "TEAMS_TIER_PRO",
  "TEAMS_TIER_TEAMS",
  "TEAMS_TIER_ENTERPRISE_SELF_HOSTED",
  "TEAMS_TIER_ENTERPRISE_SAAS",
  "TEAMS_TIER_HYBRID",
  "TEAMS_TIER_PRO_ULTIMATE"
]
```

## 9. Special Purpose Models

| Role | Model(s) |
|------|----------|
| `brainModel` | Gemini 2.5 Pro, GPT-4.1, o3 |
| `judge_model` | Gemini 2.5 Pro |
| `plan_model` | GPT-4o Mini |
| `checkpoint_model` | Gemini 2.5 Flash / Claude 3.5 Haiku |
| `defaultAgentModelId` | gemini-3-pro-high |

## 10. Global Configuration

### CASCADE_GLOBAL_CONFIG_OVERRIDE

```json
{
  "planner_config": {
    "truncation_threshold_tokens": 160000,
    "tool_config": {
      "run_command": {
        "auto_command_config": {
          "system_allowlist": ["echo", "ls"],
          "system_denylist": [
            "git", "rm", "pkill", "kubectl delete",
            "kubectl apply", "terraform", "kill", "del",
            "rmdir", "psql", "mv", "bash", "zsh"
          ]
        }
      },
      "mquery": {
        "force_disable": true
      }
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

## 11. Background Research Configuration

For background/research tasks:

```json
{
  "planner_config": {
    "plan_model": "MODEL_CHAT_GPT_4O_MINI_2024_07_18"
  },
  "checkpoint_config": {
    "token_threshold": "30000",
    "max_overhead_ratio": "0.05",
    "moving_window_size": "1",
    "max_token_limit": "40000",
    "enabled": true
  }
}
```

## 12. Client Model Configuration Response

Example of model configuration sent to clients:

```json
{
  "clientModelConfigs": [
    {
      "label": "Claude Sonnet 4.5",
      "modelOrAlias": {"model": "MODEL_CLAUDE_4_5_SONNET"},
      "supportsImages": true,
      "isRecommended": true,
      "allowedTiers": ["TEAMS_TIER_PRO", "TEAMS_TIER_TEAMS", ...],
      "quotaInfo": {
        "remainingFraction": 0.6666667,
        "resetTime": "2026-01-02T08:33:01Z"
      },
      "supportedMimeTypes": {
        "image/jpeg": true,
        "image/png": true,
        "image/webp": true
      }
    }
  ],
  "clientModelSorts": [
    {
      "name": "Recommended",
      "groups": [{
        "modelLabels": [
          "Gemini 3 Pro (High)",
          "Gemini 3 Pro (Low)",
          "Claude Sonnet 4.5",
          "Claude Sonnet 4.5 (Thinking)"
        ]
      }]
    }
  ],
  "defaultOverrideModelConfig": {
    "modelOrAlias": {"model": "MODEL_PLACEHOLDER_M8"}
  }
}
```

## 13. Model Provider Enum

```
MODEL_PROVIDER_UNSPECIFIED
MODEL_PROVIDER_ANTIGRAVITY
MODEL_PROVIDER_OPENAI
MODEL_PROVIDER_ANTHROPIC
MODEL_PROVIDER_GOOGLE
MODEL_PROVIDER_XAI
MODEL_PROVIDER_DEEPSEEK
```

## 14. Pricing Types

```
MODEL_PRICING_TYPE_UNSPECIFIED
MODEL_PRICING_TYPE_STATIC_CREDIT
MODEL_PRICING_TYPE_API
MODEL_PRICING_TYPE_BYOK (Bring Your Own Key)
```
