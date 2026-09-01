# Web Search Configuration

## Free Web Search Options Ranked

| Provider | Free Tier | Per Search | Notes |
|----------|-----------|-----------|-------|
| **Exa** | 1000/mo | free | Best quality, real API key from https://dashboard.exa.ai/api-keys |
| **Tavily** | 1000/mo | free | Easy, https://tavily.com |
| **Firecrawl** | 1000 credits/mo | 2/10 results | 500 searches/mo free |
| **SerpAPI** | 100/mo | free | Limited |
| **Brave Search** | 2000/mo | 1 q/s | |
| **Google CSE** | 100/day | free | |

## Built-in DSH Search Providers

DSH has 3 built-in web search providers:

1. **`deepseek-official`** (default) - uses `DEEPSEEK_API_KEY`
2. **`exa`** - uses `EXA_API_KEY`, exa.ai
3. **`perplexity`** - uses `PERPLEXITY_API_KEY`, perplexity.ai

## How to Configure

Edit `~/.dsh/profiles/web/cordis.patch.yml`:

```yaml
# Change search provider
- id: web
  config:
    searchProvider: exa  # or 'deepseek-official' or 'perplexity'
    fetchProvider: http

# Register the provider plugin
- id: web-search-exa
  name: '@deepseek-ai/dsh-web-search-exa'
  config:
    apiKeyEnv: EXA_API_KEY
    baseURL: https://api.exa.ai
```

## Exa Setup

1. Get API key from https://dashboard.exa.ai/api-keys
2. **IMPORTANT:** The key is case-sensitive! Copy exactly (lowercase + dashes)
3. Add to `~/.dsh/.env`:
   ```bash
   export EXA_API_KEY="your-key-here"
   ```
4. Install package into profile:
   ```bash
   cd /root/deepseek-harness
   pnpm dsh plugin --profile web add packages/web/web-search-exa
   ```
5. Restart PM2:
   ```bash
   pm2 restart dsh-web
   ```

## Free Search Without API Key

**FreeSerp** (https://freeserp.ai) - 100% free, no API key:
- REST: `curl 'https://freeserp.ai/api.php?q=ai+chatbot&size=3'`
- MCP: `https://freeserp.ai/mcp`
- Tools: `freeserp_search`, `freeserp_list_endpoints`, `freeserp_get`
- Not currently registered in DSH (would need MCP client setup)

## Tavily as Free Alternative

Tavily (https://tavily.com) - 1000 free searches/month:
- Endpoint: `https://api.tavily.com/search` (POST)
- Auth: `{"api_key":"..."}` in body
- Not natively supported by DSH (would need custom provider)

## Tested Exa Key (Working)

```bash
# This format works (lowercase + dashes):
export EXA_API_KEY="c5e6bf92-d458-4f72-bc61-c59f2b83529c"

# These don't work (uppercase fails):
# "CSE6BF92-D458-4F72-BC61-C59F2B83529C" → INVALID_API_KEY
```
