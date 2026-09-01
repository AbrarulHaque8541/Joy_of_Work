# DeepSeek Harness - Working Providers & Models

## Provider Test Results (Sept 2026)

### ✅ WORKING PROVIDERS

#### 1. **Groq** (FREE - 4 Keys Working)
- **Models:**
  - `qwen/qwen3.8-27b` ⭐ (Recommended)
  - `qwen/qwen3.6-27b`
  - `openai/gpt-oss-120b`
  - `openai/gpt-oss-20b`
  - `groq/compound`
  - `groq/compound-mini`
  - `allam-2-7b`
- **Best for:** Fastest inference, free tier
- **API Endpoint:** `https://api.groq.com/openai/v1`

#### 2. **OpenRouter** (FREE - 8 Keys Working)
- **Models:**
  - `minimax/minimax-m3:free` ⭐ (Best free model)
  - `inclusionai/ling-3.0-flash-fin:free`
  - `nvidia/nemotron-3.5-lightning:free`
  - `cohere/north-mini-code:free`
  - `dots-studio/dots-3-note-preview:free`
  - `liquid/lfm-2.5-2.6b:free`
  - `minimax/minimax-m2.7:free`
- **Best for:** Multiple free models, auto-failover
- **API Endpoint:** `https://openrouter.ai/api/v1`

#### 3. **Mistral** (Paid - 1 Key Working)
- **Models:**
  - `mistral-small-latest` ⭐
  - `mistral-small-2603`
  - `codestral-latest` ⭐ (Best for coding)
  - `mistral-code-latest`
  - `mistral-vibe-cli-fast`
  - `magistral-small-latest`
- **Best for:** Coding tasks, fast small models
- **API Endpoint:** `https://api.mistral.ai/v1`

### ❌ PROVIDERS WITH ISSUES

#### DeepSeek (Keys Valid but NO Balance)
- `deepseek-v4-flash`
- `deepseek-v4-pro`
- `deepseek-v4-flash-vision-exp`
- **Issue:** Account has no credits remaining

#### OpenAI (NO Balance)
- **Issue:** No credits remaining

#### Anthropic (Invalid Key)
- **Issue:** API key is invalid

#### NVIDIA (Partial Working)
- Only `meta/llama-3.2-11b-vision-instruct` works
- Needs `baseURL: https://integrate.api.nvidia.com/v1`

#### xAI, Mimo, Sakana, FreeModel, StepFun, Z.ai
- **Issue:** No credits or token expired/invalid

#### Cohere, HuggingFace, Upstage
- Keys work but need custom `api` and `baseURL` config
- Not yet configured in pi-ai plugin

---

## Configuration Files

### `~/.dsh/profiles/web/cordis.patch.yml`
Contains the active provider configuration.

### `~/.dsh/settings.yaml`
Contains model preferences and agent presets.

---

## Quick Reference

**Default Model:** `groq` / `qwen/qwen3.8-27b`

**Free Models Available:**
1. Groq: 7 models (qwen, gpt-oss, compound, allam)
2. OpenRouter: 7 models (minimax, ling, nemotron, etc.)

**Paid Models Available:**
1. Mistral: 6 models (small, codestral, etc.)
