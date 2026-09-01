# DeepSeek Harness - Working Configuration

## Version
- DSH: v0.1.2-alpha.2 (latest as of Sept 2026)
- Environment: proot Ubuntu on ARM64 (Termux/Android)

## Working LLM Providers (pi-ai catalog)

### FREE Tier Providers

#### Groq (6 models in catalog)
- **Status:** WORKING - 4 keys tested
- **API Keys:** `GROQ_API_KEY`, `GROQ_API_KEY_1`, `GROQ_API_KEY_3`, `GROQ_API_KEY_6`
- **Default Model:** `llama-3.3-70b-versatile` (70B, best free)
- **Fast Model:** `llama-3.1-8b-instant` (8B, fastest)
- **Other models:** `qwen/qwen3.6-27b`, `openai/gpt-oss-120b`, `openai/gpt-oss-20b`

#### OpenRouter (346 models in catalog)
- **Status:** WORKING - all 8 keys tested
- **API Keys:** `OPENROUTER_API_KEY` + 7 more
- **Free models:** `google/gemini-2.0-flash`, `meta-llama/llama-3.3-70b-instruct`
- **Models:** 420 per key

### PAID Providers

#### Mistral (31 models in catalog)
- **Status:** WORKING
- **API Key:** `MISTRAL_API_KEY`
- **Verified working:** `codestral-latest`, `mistral-small-latest`, `magistral-medium-latest`
- **No credits issue** - confirmed working

#### NVIDIA NIM (30 catalog, 3 verified)
- **Status:** WORKING (limited)
- **API Keys:** `NVIDIA_API_KEY` + 2 more
- **Verified working:**
  - `openai/gpt-oss-120b`
  - `openai/gpt-oss-20b`
  - `meta/llama-3.2-11b-vision-instruct`
- **EOL models:** `meta/llama-3.1-8b-instruct`, `meta/llama-3.3-70b-instruct`, `google/gemma-3-12b-it`

## NOT Working Providers

| Provider | Reason | Keys |
|---------|--------|------|
| DeepSeek | No balance (0 credits) | Multiple keys |
| OpenAI | No balance | `OPENAI_API_KEY` |
| Anthropic | Invalid API key | `ANTHROPIC_API_KEY` |
| xAI | No balance | `XAI_API_KEY`, `XAI_MANAGEMENT_KEY` |
| Google/Gemini | Key leaked/compromised | `GOOGLE_API_KEY` |
| Cohere | Works via baseURL | `COHERE_API_KEY` |
| HuggingFace | No chat endpoint | `HF_API_KEY`, `HF_API_KEY_3` |
| Venice | No balance | `VENICE_API_KEY` |
| FreeModel | No balance | `FREEMODEL_API_KEY` |
| BytePlus | Endpoint access denied | `BYTEPLUS_API_KEY` |
| Mimo | No balance | `MIMO_API_KEY` |
| Sakana | No balance | `SAKANA_API_KEY` |
| StepFun | Invalid key | `STEPFUN_API_KEY` |
| Alibaba/DashScope | Invalid key | `ALIBABA_CLAUDE_KEY` |
| Cerebras/Together | No TOGETHER_API_KEY | N/A |
| Moonshot/Kimi | Invalid authentication | Multiple keys |
| Exa Search API | Invalid key | `CSE6BF92-...` (uppercase) |

## pi-ai Catalog Summary

Total: **1267 catalog entries across 39 providers**

Key providers NOT in our .env: `amazon-bedrock`, `baseten`, `cloudflare-ai-gateway`, `fireworks`, `github-copilot`, `google-vertex`, `openai-codex`, `opencode`, `vercel-ai-gateway`

## Current cordis.patch.yml (Working)

```yaml
# DeepSeek Harness - COMPLETE Configuration
# - All working LLM providers with ACTUAL pi-ai catalog model IDs
# - Exa web search provider configured
# Updated: Sept 2026
# DSH version: 0.1.2-alpha.2 (latest)

- id: llm-pi-ai
  config:
    providers:
      groq:
        apiKeyEnv: GROQ_API_KEY
        models:
          - id: llama-3.1-8b-instant
            name: Llama 3.1 8B (Fastest FREE)
          - id: llama-3.3-70b-versatile
            name: Llama 3.3 70B (Best FREE)
          - id: qwen/qwen3.6-27b
            name: Qwen 3.6 27B
          - id: openai/gpt-oss-120b
            name: GPT-OSS 120B
          - id: openai/gpt-oss-20b
            name: GPT-OSS 20B
        retryPolicy:
          mode: normal
          maxRetries: 5

      openrouter:
        apiKeyEnv: OPENROUTER_API_KEY
        models:
          - id: deepseek/deepseek-chat-v3
            name: DeepSeek V3
          - id: google/gemini-2.0-flash
            name: Gemini 2.0 Flash
          - id: meta-llama/llama-3.3-70b-instruct
            name: Llama 3.3 70B
          - id: mistralai/mistral-7b-instruct
            name: Mistral 7B
          - id: anthropic/claude-3-haiku
            name: Claude 3 Haiku
        retryPolicy:
          mode: normal
          maxRetries: 5

      mistral:
        apiKeyEnv: MISTRAL_API_KEY
        models:
          - id: codestral-latest
            name: Codestral (Coding)
          - id: mistral-small-latest
            name: Mistral Small
          - id: magistral-medium-latest
            name: Magistral Medium
          - id: devstral-small-2507
            name: Devstral Small
        retryPolicy:
          mode: normal
          maxRetries: 3

      nvidia:
        apiKeyEnv: NVIDIA_API_KEY
        models:
          - id: openai/gpt-oss-120b
            name: GPT-OSS 120B
          - id: openai/gpt-oss-20b
            name: GPT-OSS 20B
          - id: meta/llama-3.2-11b-vision-instruct
            name: Llama 3.2 Vision 11B
        retryPolicy:
          mode: normal
          maxRetries: 3

- id: web
  config:
    searchProvider: exa
    fetchProvider: http

- id: web-search-exa
  name: '@deepseek-ai/dsh-web-search-exa'
  config:
    apiKeyEnv: EXA_API_KEY
    baseURL: https://api.exa.ai

- id: agent-default-model
  config:
    provider: groq
    model: llama-3.3-70b-versatile
```

## Setup Commands

```bash
# Install Exa search package
cd /root/deepseek-harness
pnpm dsh plugin --profile web add packages/web/web-search-exa

# Restart server
pm2 restart dsh-web

# Check status
pm2 list
pm2 logs dsh-web --lines 10
```

## Model Selection Priority

1. **Default:** `groq/llama-3.3-70b-versatile` (FREE, 70B)
2. **Fast:** `groq/llama-3.1-8b-instant` (FREE, 8B)
3. **Reasoning:** `groq/openai/gpt-oss-120b` (FREE, 120B)
4. **Coding:** `mistral/codestral-latest` (PAID)
5. **Vision:** `nvidia/meta/llama-3.2-11b-vision-instruct` (PAID)
