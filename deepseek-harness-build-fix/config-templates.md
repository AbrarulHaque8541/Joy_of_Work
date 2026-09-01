# DeepSeek Harness - Configuration Templates

## cordis.patch.yml Template

Copy this to `~/.dsh/profiles/web/cordis.patch.yml`:

```yaml
# DeepSeek Harness - Multi-Provider Configuration
# Free models prioritized, automatic failover enabled

- id: llm-pi-ai
  config:
    providers:
      # ===== Free Tier (Priority 1) =====
      groq:
        apiKeyEnv: GROQ_API_KEY
        models:
          - id: llama-3.3-70b-versatile
          - id: mixtral-8x7b-32768
          - id: llama-3.1-8b-instant
        retryPolicy:
          mode: normal
          maxRetries: 5

      deepseek:
        apiKeyEnv: DEEPSEEK_API_KEY
        models:
          - id: deepseek-chat
          - id: deepseek-coder
        retryPolicy:
          mode: normal
          maxRetries: 3

      openrouter:
        apiKeyEnv: OPENROUTER_API_KEY
        models:
          - id: deepseek/deepseek-chat-v3-0324:free
          - id: anthropic/claude-3-haiku:free
        retryPolicy:
          mode: normal
          maxRetries: 3

      google:
        apiKeyEnv: GEMINI_API_KEY
        models:
          - id: gemini-2.0-flash-exp
          - id: gemini-1.5-flash
        retryPolicy:
          mode: normal
          maxRetries: 3

      # ===== Paid Tier (Priority 2) =====
      openai:
        apiKeyEnv: OPENAI_API_KEY
        models:
          - id: gpt-4o-mini
        retryPolicy:
          mode: normal
          maxRetries: 3

      anthropic:
        apiKeyEnv: ANTHROPIC_API_KEY
        models:
          - id: claude-sonnet-4-6
          - id: claude-3-5-haiku-latest
        retryPolicy:
          mode: normal
          maxRetries: 3

      mistral:
        apiKeyEnv: MISTRAL_API_KEY
        models:
          - id: mistral-small-latest
          - id: codestral-latest
        retryPolicy:
          mode: normal
          maxRetries: 3

      huggingface:
        apiKeyEnv: HF_API_KEY
        models:
          - id: meta-llama/Llama-3.3-70B-Instruct
        retryPolicy:
          mode: normal
          maxRetries: 3

- id: agent-default-model
  config:
    provider: groq
    model: llama-3.3-70b-versatile
```

## settings.yaml Template

Copy this to `~/.dsh/settings.yaml`:

```yaml
ui-onboarding:
  welcomeNoticeVersion: 2026-08-13.1
ui-theme:
  preference: dark
permission:
  defaultPreset: danger-full-access

subagent-model-selection:
  enabled: true
  preferredProvider: groq
  preferredModel: llama-3.3-70b-versatile
  allowedModels:
    # Groq (Free, Fast)
    - provider: groq
      model: llama-3.3-70b-versatile
    - provider: groq
      model: mixtral-8x7b-32768
    - provider: groq
      model: llama-3.1-8b-instant
    # DeepSeek
    - provider: deepseek
      model: deepseek-chat
    - provider: deepseek
      model: deepseek-coder
    # OpenRouter (Free)
    - provider: openrouter
      model: deepseek/deepseek-chat-v3-0324:free
    - provider: openrouter
      model: anthropic/claude-3-haiku:free
    # Google
    - provider: google
      model: gemini-2.0-flash-exp
    # OpenAI
    - provider: openai
      model: gpt-4o-mini
    # Anthropic
    - provider: anthropic
      model: claude-3-5-haiku-latest

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
    reasoning:
      model:
        provider: deepseek
        model: deepseek-reasoner
    fast:
      model:
        provider: groq
        model: llama-3.1-8b-instant
    pro:
      model:
        provider: openai
        model: gpt-4o-mini
```

## .env Template

Copy this to `~/.dsh/.env` and add your keys:

```bash
# DeepSeek
export DEEPSEEK_API_KEY="your-key-here"

# Groq (Best Free Tier)
export GROQ_API_KEY="your-key-here"

# OpenRouter
export OPENROUTER_API_KEY="your-key-here"

# Google Gemini
export GEMINI_API_KEY="your-key-here"

# OpenAI
export OPENAI_API_KEY="your-key-here"

# Anthropic
export ANTHROPIC_API_KEY="your-key-here"

# Mistral
export MISTRAL_API_KEY="your-key-here"

# HuggingFace
export HF_API_KEY="your-key-here"

# xAI Grok (Optional)
export XAI_API_KEY="your-key-here"

# NVIDIA (Optional)
export NVIDIA_API_KEY="your-key-here"
```

## Startup Script Template

Create `/usr/local/bin/dsh-web.sh`:

```bash
#!/bin/bash
cd /root/deepseek-harness

if [ -f /root/.dsh/.env ]; then
    set -a
    source /root/.dsh/.env
    set +a
fi

exec pnpm dsh web --no-open
```

Make executable:
```bash
chmod +x /usr/local/bin/dsh-web.sh
```

## PM2 Setup

```bash
# Install PM2
npm install -g pm2

# Start server
pm2 start /usr/local/bin/dsh-web.sh --name "dsh-web"

# Save configuration
pm2 save

# Auto-start on boot
pm2 startup
```

## Verification

Test provider connectivity:

```bash
source /root/.dsh/.env

# DeepSeek
curl https://api.deepseek.com/v1/models \
  -H "Authorization: Bearer $DEEPSEEK_API_KEY"

# Groq
curl https://api.groq.com/openai/v1/models \
  -H "Authorization: Bearer $GROQ_API_KEY"

# Gemini
curl "https://generativelanguage.googleapis.com/v1beta/models?key=$GEMINI_API_KEY"
```

## Important Notes

- **NEVER** commit `.env` file with real keys
- API keys should be set via environment variables
- Free tier models are prioritized over paid
- All providers have automatic retry/failover
- `cordis.patch.yml` is hot-reloaded (no restart needed for most changes)
