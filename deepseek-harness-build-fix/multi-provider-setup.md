# DeepSeek Harness - Multi-Provider Setup Guide

## Overview

This guide explains how to configure DeepSeek Harness with multiple LLM providers, prioritizing free models for maximum value.

## Quick Start

### 1. API Keys Configuration

Create `/root/.dsh/.env` with your API keys:

```bash
# DeepSeek
export DEEPSEEK_API_KEY="your-deepseek-key"

# Groq (Best Free Tier)
export GROQ_API_KEY="your-groq-key"

# OpenRouter
export OPENROUTER_API_KEY="your-openrouter-key"

# Google Gemini
export GEMINI_API_KEY="your-gemini-key"

# And more providers...
```

### 2. Provider Configuration

Edit `/root/.dsh/profiles/web/cordis.patch.yml`:

```yaml
- id: llm-pi-ai
  config:
    providers:
      # Groq - Fastest Free Tier
      groq:
        apiKeyEnv: GROQ_API_KEY
        models:
          - id: llama-3.3-70b-versatile
          - id: mixtral-8x7b-32768
          - id: llama-3.1-8b-instant
        retryPolicy:
          mode: normal
          maxRetries: 5

      # DeepSeek
      deepseek:
        apiKeyEnv: DEEPSEEK_API_KEY
        models:
          - id: deepseek-chat
          - id: deepseek-coder
        retryPolicy:
          mode: normal
          maxRetries: 3

      # OpenRouter (Free Tier)
      openrouter:
        apiKeyEnv: OPENROUTER_API_KEY
        models:
          - id: deepseek/deepseek-chat-v3-0324:free
          - id: anthropic/claude-3-haiku:free
        retryPolicy:
          mode: normal
          maxRetries: 3

      # Google Gemini
      google:
        apiKeyEnv: GEMINI_API_KEY
        models:
          - id: gemini-2.0-flash-exp
          - id: gemini-1.5-flash
        retryPolicy:
          mode: normal
          maxRetries: 3

- id: agent-default-model
  config:
    provider: groq
    model: llama-3.3-70b-versatile
```

### 3. Model Preferences

Edit `/root/.dsh/settings.yaml`:

```yaml
subagent-model-selection:
  enabled: true
  preferredProvider: groq
  preferredModel: llama-3.3-70b-versatile
  allowedModels:
    - provider: groq
      model: llama-3.3-70b-versatile
    - provider: groq
      model: mixtral-8x7b-32768
    - provider: deepseek
      model: deepseek-chat
    - provider: openrouter
      model: deepseek/deepseek-chat-v3-0324:free
    - provider: google
      model: gemini-2.0-flash-exp

agent-presets:
  default: standard
  presets:
    minimal:
      model:
        provider: groq
        model: llama-3.1-8b-instant
    standard:
      model:
        provider: groq
        model: llama-3.3-70b-versatile
    coding:
      model:
        provider: deepseek
        model: deepseek-coder
```

## Available Providers

### Free Tier Providers

| Provider | Best Models | Context Window | Speed |
|---------|------------|---------------|-------|
| **Groq** | Llama 3.3 70B, Mixtral 8x7B | 128K | Fastest |
| **OpenRouter** | DeepSeek V3:free, Claude 3 Haiku:free | 64K-200K | Fast |
| **DeepSeek** | V3, Coder | 64K | Fast |

### Paid Providers (Higher Quality)

| Provider | Best Models | Notes |
|---------|------------|-------|
| **OpenAI** | GPT-4o-mini, GPT-4o | Industry standard |
| **Anthropic** | Claude Sonnet 4, Claude 3.5 | Best reasoning |
| **Google** | Gemini 2.0 Flash, Gemini 1.5 Pro | Large context |
| **Mistral** | Mistral Large, Codestral | Great for coding |

## Supported Models

### Groq (Recommended - Free & Fast)
- `llama-3.3-70b-versatile` - Best overall
- `mixtral-8x7b-32768` - Great alternative
- `llama-3.1-8b-instant` - Fastest
- `gemma2-9b-it` - Small but capable

### DeepSeek
- `deepseek-chat` - General purpose
- `deepseek-coder` - Code-focused
- `deepseek-reasoner` - Deep reasoning (R1)

### OpenRouter (Free Tier)
- `deepseek/deepseek-chat-v3-0324:free` - DeepSeek V3
- `anthropic/claude-3-haiku:free` - Claude Haiku
- `google/gemini-2.0-flash-thinking-exp:free` - Gemini thinking

### Google Gemini
- `gemini-2.0-flash-exp` - Fast & capable
- `gemini-1.5-flash` - Balanced
- `gemini-1.5-pro` - High quality

## Auto-Failover

Each provider has `retryPolicy` configured:
- **mode**: `normal` (automatic retry)
- **maxRetries**: 3-5 (per provider)
- If one provider fails, the system retries automatically

## Starting the Server

### Using PM2
```bash
pm2 start /usr/local/bin/dsh-web.sh --name "dsh-web"
pm2 save  # Save for auto-restart
```

### Manual Start
```bash
cd /root/deepseek-harness
source /root/.dsh/.env
pnpm dsh web
```

## Environment Variables

All API keys should be stored in `/root/.dsh/.env` and NOT committed to git.

### Required Variables
```bash
# LLM Providers
DEEPSEEK_API_KEY=
GROQ_API_KEY=
OPENROUTER_API_KEY=
GEMINI_API_KEY=
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
HF_API_KEY=  # HuggingFace
```

### Optional Variables
```bash
# Additional Providers
XAI_API_KEY=          # xAI Grok
MISTRAL_API_KEY=      # Mistral AI
NVIDIA_API_KEY=       # NVIDIA NIM
COHERE_API_KEY=       # Cohere
MOONSHOT_API_KEY=     # Kimi/Moonshot
ZAI_API_KEY=          # Z.ai
STEPFUN_API_KEY=      # StepFun
OPENCODE_ZEN_KEY=     # OpenCode Zen

# Web & Search
FIRECRAWL_API_KEY=    # Firecrawl
SERPAPI_KEY=          # SerpAPI

# Communication
TELEGRAM_BOT_TOKEN_1= # Telegram Bot
DISCORD_BOT_TOKEN=    # Discord Bot
```

## Troubleshooting

### Provider Connection Issues

1. Check API key is valid
2. Verify network connectivity
3. Check provider status pages
4. Review logs: `pm2 logs dsh-web`

### Model Not Found

Some models require specific provider setup. Check:
- Provider has the model in their catalog
- API key has access to that model
- Model ID is correct (case-sensitive)

### Config Not Loading

1. Verify YAML syntax
2. Check file location: `/root/.dsh/profiles/web/cordis.patch.yml`
3. Restart server: `pm2 restart dsh-web`

## Security Notes

- **NEVER commit API keys to git**
- Use environment variables, not hardcoded keys
- Rotate keys periodically
- Use separate keys for dev/prod

## File Structure

```
~/.dsh/
├── .env                    # API keys (private)
├── settings.yaml           # User preferences
└── profiles/
    └── web/
        └── cordis.patch.yml  # Provider config
```

## Support

For issues with:
- **DSH**: Check `/root/deepseek-harness` docs
- **Providers**: Check respective provider documentation
- **pi-ai**: https://github.com/earendil-works/pi-ai
