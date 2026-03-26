# Ollama Docker Server

A Docker Compose setup that runs local LLMs via [Ollama](https://ollama.ai/), exposes them through a [LiteLLM](https://docs.litellm.ai/) OpenAI-compatible proxy, and provides a [Open WebUI](https://github.com/open-webui/open-webui) chat interface — all locally, with no cloud API keys required.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Docker Network                     │
│                                                       │
│  ┌──────────────┐    ┌──────────────────────────┐    │
│  │    Ollama    │◄───│        LiteLLM           │    │
│  │  :11434      │    │  OpenAI + Anthropic API  │    │
│  │  (local LLMs)│    │        :4000             │    │
│  └──────────────┘    └──────────────────────────┘    │
│         ▲                                             │
│         │            ┌──────────────────────────┐    │
│         └────────────│       Open WebUI         │    │
│                      │    Chat interface :3000   │    │
│                      └──────────────────────────┘    │
└─────────────────────────────────────────────────────┘
        ▲                    ▲                  ▲
        │                    │                  │
  Direct API           Claude Code         Web Browser
  :11434               VS Code ext         :3000
                        :4000
```

### Services

| Service | Port | Description |
|---------|------|-------------|
| Ollama | 11434 | Local LLM server — direct REST API |
| LiteLLM | 4000 | OpenAI & Anthropic-compatible proxy — routes to Ollama |
| Open WebUI | 3000 | ChatGPT-style web interface |

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/)
- At least 8 GB RAM (16 GB recommended for larger models)

## Quick Start

### 1. Configure environment

The `.env` file controls which ports are exposed on the host. The defaults work out of the box:

```bash
OLLAMA_PORT=11434
LITELLM_PORT=4000
WEBUI_PORT=3000
```

Change any value if a port is already in use on your machine.

### 2. Start all services

```bash
docker compose up -d
```

### 3. Pull a model

```bash
docker exec ollama ollama pull mistral
```

### 4. Verify everything is running

```bash
docker compose ps
```

All three services should show `running`.

---

## Downloading Models

Pull models into the running Ollama container. They are stored in `./local_data/models` and persist across restarts.

```bash
# General-purpose
docker exec ollama ollama pull mistral
docker exec ollama ollama pull llama3.2

# Code-focused
docker exec ollama ollama pull codellama:13b
docker exec ollama ollama pull codeqwen
docker exec ollama ollama pull deepseek-coder

# List downloaded models
docker exec ollama ollama list
```

Browse all available models at [ollama.ai/library](https://ollama.ai/library).

---

## Using Open WebUI

Open `http://localhost:3000` in your browser.

- On first launch, create an admin account.
- Select any downloaded model from the dropdown.
- Chat, manage conversations, and switch models freely.

---

## API Usage

### Ollama API (port 11434)

Direct access to the Ollama REST API:

```bash
# Generate a completion
curl http://localhost:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mistral",
    "prompt": "Why is the sky blue?",
    "stream": false
  }'

# Chat
curl http://localhost:11434/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mistral",
    "messages": [{"role": "user", "content": "Hello!"}],
    "stream": false
  }'
```

### LiteLLM Proxy — OpenAI-compatible (port 4000)

Anything that speaks the OpenAI API can point to `http://localhost:4000`:

```bash
curl http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer dummy" \
  -d '{
    "model": "mistral",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

---

## VS Code — Claude Code Extension

The [Claude Code VS Code extension](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code) can be pointed at the local LiteLLM proxy instead of Anthropic's cloud API. LiteLLM maps Claude model names to your local Ollama models.

### How it works

`litellm_config.yaml` defines Claude-named aliases that route to local models:

| Claude model name | Local model |
|-------------------|-------------|
| `claude-3-5-sonnet-20241022` | `mistral` |
| `claude-3-5-haiku-20241022` | `mistral` |
| `claude-3-opus-20240229` | `codellama:13b` |

You can edit `litellm_config.yaml` to remap any alias to whichever local model you prefer, then restart LiteLLM:

```bash
docker compose restart litellm
```

### Configuration

#### Option A — Environment variables (recommended)

Export these in your shell profile (`~/.zshrc`, `~/.bashrc`, etc.) or in a `.env` file loaded before launching VS Code:

```bash
export ANTHROPIC_BASE_URL=http://localhost:4000
export ANTHROPIC_API_KEY=local          # any non-empty string
```

Then restart VS Code and open Claude Code. It will send requests to LiteLLM, which forwards them to Ollama.

#### Option B — Claude Code settings file

Add the following to `~/.claude/settings.json` (create the file if it does not exist):

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:4000",
    "ANTHROPIC_API_KEY": "local"
  }
}
```

#### Option C — Per-project override

Create `.claude/settings.json` in your project root (this overrides global settings for that project only):

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:4000",
    "ANTHROPIC_API_KEY": "local"
  }
}
```

### Verify the connection

With the extension configured, open the Claude Code panel in VS Code and send a test message. You can confirm the request reached Ollama by watching the logs:

```bash
docker compose logs -f ollama
```

### Switching back to Anthropic cloud

Remove or unset the environment variables and restart VS Code:

```bash
unset ANTHROPIC_BASE_URL
unset ANTHROPIC_API_KEY
```

---

## Managing the Stack

```bash
# Start
docker compose up -d

# Stop (containers removed, data preserved)
docker compose down

# Restart a single service after config change
docker compose restart litellm

# Follow logs
docker compose logs -f
docker compose logs -f ollama
docker compose logs -f litellm

# Interactive chat in the terminal
docker exec -it ollama ollama run mistral
```

---

## Data Persistence

All model weights and Open WebUI data are stored in `./local_data/`:

```
local_data/
├── models/        # Ollama model blobs and manifests
└── open-webui/    # Open WebUI database and cache
```

To wipe everything and start fresh:

```bash
docker compose down
rm -rf local_data
```

---

## Configuration Reference

### `.env`

| Variable | Default | Description |
|----------|---------|-------------|
| `OLLAMA_PORT` | `11434` | Host port for the Ollama API |
| `LITELLM_PORT` | `4000` | Host port for the LiteLLM proxy |
| `WEBUI_PORT` | `3000` | Host port for Open WebUI |

### `litellm_config.yaml`

Defines which models LiteLLM exposes and where each routes to. Edit this file to:
- Add new Ollama models as they are pulled
- Remap Claude aliases to different local models
- Add multiple aliases for the same underlying model

After any change, run `docker compose restart litellm`.

---

## Troubleshooting

### Port already in use

```bash
lsof -i :11434   # check Ollama port
lsof -i :4000    # check LiteLLM port
lsof -i :3000    # check WebUI port
```

Change the relevant variable in `.env` and restart with `docker compose up -d`.

### Ollama not reachable from host

The Ollama container is configured with `OLLAMA_HOST=0.0.0.0` so it listens on all interfaces inside the container. If it is still unreachable, confirm the port mapping is active:

```bash
docker compose ps
docker inspect ollama | grep -A5 Ports
```

### Model not listed in Open WebUI or LiteLLM

Pull the model first, then wait a few seconds for Open WebUI to refresh. For LiteLLM, add the model to `litellm_config.yaml` and restart the service.

### Out of memory

- Check Docker Desktop memory allocation (Settings → Resources)
- Switch to a smaller model variant, e.g. `mistral:7b-instruct-q4_K_M`
- Reduce the number of simultaneously running services

### LiteLLM returns errors for Claude model names

Ensure the model referenced in `litellm_config.yaml` is already pulled in Ollama:

```bash
docker exec ollama ollama list
```

---

## Resources

- [Ollama documentation](https://github.com/ollama/ollama)
- [Ollama model library](https://ollama.ai/library)
- [Ollama API reference](https://github.com/ollama/ollama/blob/main/docs/api.md)
- [LiteLLM documentation](https://docs.litellm.ai/)
- [Open WebUI documentation](https://docs.openwebui.com/)
- [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code)
