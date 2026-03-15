# Swiss AI Platform Setup

ZeroClaw supports Swiss AI Platform through the built-in `swiss-ai-platform`
provider, which wraps the existing OpenAI-compatible provider path.

This guide shows the recommended setup for tenant-specific Swiss AI Platform
endpoints, including validation and common troubleshooting steps.

## Overview

Use this provider when:

- your Swiss AI Platform endpoint is OpenAI-compatible
- you want a stable provider ID in `config.toml`
- you prefer `api_url` over embedding the full endpoint in `default_provider`

Current provider IDs:

| ID | Aliases | Notes |
|---|---|---|
| `swiss-ai-platform` | `swiss_ai_platform` | Requires `api_url` or `SWISS_AI_PLATFORM_API_URL` |

Credential resolution order for this provider:

1. Explicit `api_key` from config or CLI
2. `SWISS_AI_PLATFORM_API_KEY`
3. Generic fallback: `ZEROCLAW_API_KEY`
4. Generic fallback: `API_KEY`

## Required Values

You need:

- a Swiss AI Platform API key
- the correct tenant/model base URL for your deployment
- the exact model ID exposed by that endpoint

Typical configuration shape:

```toml
default_provider = "swiss-ai-platform"
api_url = "https://api.swisscom.com/layer/your-scope/your-model/v1"
api_key = "your-swiss-ai-platform-key"
default_model = "your-model-name"
default_temperature = 0.7
```

## Quick Start

If you already know the endpoint and key:

```bash
zeroclaw onboard \
  --provider "swiss-ai-platform" \
  --api-key "YOUR_SWISS_AI_PLATFORM_API_KEY"
```

Then set `api_url` in `~/.zeroclaw/config.toml` if onboarding did not already
write it for your environment.

## Manual Configuration

Edit `~/.zeroclaw/config.toml`:

```toml
default_provider = "swiss-ai-platform"
api_url = "https://api.swisscom.com/layer/your-scope/your-model/v1"
api_key = "your-swiss-ai-platform-key"
default_model = "your-model-name"
default_temperature = 0.7
```

Environment variable alternative:

```bash
export SWISS_AI_PLATFORM_API_URL="https://api.swisscom.com/layer/your-scope/your-model/v1"
export SWISS_AI_PLATFORM_API_KEY="your-swiss-ai-platform-key"
zeroclaw agent -m "hello"
```

## Optional Compatibility Settings

If your Swiss AI Platform gateway requires custom headers:

```toml
[extra_headers]
X-Project = "my-project"
X-Workspace = "prod"
```

If the gateway exposes a non-standard chat path:

```toml
api_path = "/v2/chat/completions"
```

If the backend is slower and needs more time:

```toml
provider_timeout_secs = 300
```

If the platform is plain OpenAI-compatible and you do not need the dedicated
provider ID, `custom:https://...` remains a valid alternative. See
[`../contributing/custom-providers.md`](../contributing/custom-providers.md).

## Verify Setup

### Validate with ZeroClaw

```bash
zeroclaw providers
zeroclaw status
zeroclaw agent -m "Reply with: setup ok"
```

If the endpoint implements model listing:

```bash
zeroclaw models refresh --provider swiss-ai-platform
zeroclaw models list --provider swiss-ai-platform
```

### Validate with curl

```bash
curl -X POST "https://api.swisscom.com/layer/your-scope/your-model/v1/chat/completions" \
  -H "Authorization: Bearer YOUR_SWISS_AI_PLATFORM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "your-model-name",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

## Troubleshooting

### Missing `api_url`

Symptom:

- ZeroClaw reports that Swiss AI Platform requires `api_url` or `SWISS_AI_PLATFORM_API_URL`

Fix:

- set `api_url` in `config.toml`
- or export `SWISS_AI_PLATFORM_API_URL`

### Authentication Fails

Symptom:

- `401`, `403`, or provider auth error

Fix:

- verify the API key value and surrounding whitespace
- confirm the endpoint expects `Authorization: Bearer ...`
- if your gateway expects additional headers, add them via `[extra_headers]`

### Model Not Found

Symptom:

- provider returns a model-not-found error

Fix:

- verify the exact model ID exposed by your tenant endpoint
- run `zeroclaw models refresh --provider swiss-ai-platform` if `/models` is implemented
- if `/models` is not available, test a minimal `curl` request and inspect the error text

### Non-Standard Endpoint Shape

Symptom:

- `/chat/completions` returns `404`

Fix:

- set `api_path` to the provider-specific suffix
- verify whether the base URL should end at `/v1` or one level above

## Related Documentation

- [Custom Provider Endpoints](../contributing/custom-providers.md)
- [Providers Reference](../reference/api/providers-reference.md)
- [Config Reference](../reference/api/config-reference.md)
- [ZeroClaw Docs Hub](../README.md)
