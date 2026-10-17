# Codex Open — multi-provider Codex

Codex CLI (0.154.0+) ships a `model_providers` mechanism for arbitrary OpenAI-compatible
endpoints. This is our config + helper to run Codex against local and cloud providers.

Reverse-engineered with REA (native binary string analysis) + confirmed against the
official config reference and the vault notes (`.opencode/skills/references/codex-custom-providers.md`).

## Install

```bash
ln -sf "$(pwd)/config.toml" ~/.codex/config.toml
# API keys (only needed for direct providers; OmniRoute's key is baked in):
export DEEPSEEK_API_KEY=...       # for the direct DeepSeek provider
export ORCAROUTER_API_KEY=...     # for OrcaRouter
```

## Use

```bash
./codex-open <provider> <model> "your prompt"

# Local (built-in, no key):
./codex-open lmstudio liquid/lfm2.5-1.2b "…"
./codex-open ollama   llama3.2:1b        "…"

# OmniRoute (our gateway — one endpoint, 3000+ models, 13+ providers):
./codex-open omniroute deepseek/deepseek-v4-pro "…"
./codex-open omniroute mistral/mistral-medium-2505 "…"
./codex-open omniroute deepseek/deepseek-flash "…"
```

## Providers

| id | base_url | auth |
|---|---|---|
| `omniroute` | `https://omniroute.veritasglobalai.com/v1` | fixed key `omniroute` |
| `deepseek` | `https://api.deepseek.com/v1` | `DEEPSEEK_API_KEY` |
| `orcarouter` | `https://api.orcarouter.ai/v1` | `ORCAROUTER_API_KEY` (`sk-orca-…`) |

## Gotchas

- **`wire_api`**: only `responses` works now (`chat` removed). Provider must speak `POST /v1/responses`.
- **Reserved ids**: `openai`, `ollama`, `lmstudio` can't be overridden — use `--oss --local-provider`.
- **`auto` model is currently broken**: OmniRoute's "last known good provider" points at the
  deprecated `claude-opus-4-6`. Use an explicit model (e.g. `deepseek/deepseek-v4-pro`) until
  OmniRoute's routing is refreshed.
- **`base_url`** must end in `/v1` with no trailing slash.

## Adding a provider

```toml
[model_providers.<id>]
name = "Display name"
base_url = "https://api.example.com/v1"
env_key = "MY_API_KEY_ENV_VAR"   # or experimental_bearer_token = "token" for a literal key
wire_api = "responses"
```
