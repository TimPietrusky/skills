---
name: cv
description: Tim Pietrusky's portfolio of shipped agent skills. Use when the user asks about Tim's work, projects, or what skills he has built (unsloth, runpodctl, agent-media, a2go, wandler, agnt-init, agnt-prune, commit).
metadata:
  author: Tim Pietrusky
license: MIT
---

Agent skills shipped by Tim Pietrusky. Each project below is the full skill — title, license, links, and every command an agent needs.

# unsloth (Apache-2.0)

Fine-tune, run inference on, and export LLMs and vision models using the Unsloth Studio CLI.
https://github.com/unslothai/unsloth/pull/4443

## Prerequisites

### Quick install (recommended for fresh machines)

Installs Python, uv, Unsloth, and Studio in one command — everything needed to start training.

macOS / Linux / WSL:
```bash
curl -fsSL https://unsloth.ai/install.sh | sh
source unsloth_studio/bin/activate
```

Windows (PowerShell):
```powershell
irm https://unsloth.ai/install.ps1 | iex
& .\unsloth_studio\Scripts\activate
```

### Developer install (when Python and uv are already available)

```bash
uv venv unsloth_studio --python 3.13
source unsloth_studio/bin/activate
uv pip install unsloth --torch-backend=auto
unsloth studio update
```

### Minimal install (CLI only, then set up Studio separately)

```bash
pip install unsloth
unsloth studio update
```

- Python 3.11-3.13 is required inside the studio venv (created by `unsloth studio update` or the one-liner install). The system Python can differ — the CLI re-executes in the studio venv automatically.
- After setup, commands like `train`, `inference`, `export`, `list-checkpoints` re-launch inside `~/.unsloth/studio/unsloth_studio` (you'll see "Launching with studio venv..." in output). Dependencies are resolved from the studio venv, not the user's environment.
- Training requires a supported GPU. For chat-only inference on macOS / CPU-only systems, use GGUF models so the CLI routes to the llama.cpp backend.
- Environment variables: `HF_TOKEN` (HuggingFace), `WANDB_API_KEY` (Weights & Biases)

## Training

Config file approach is recommended — handles lists, avoids shell quoting.

```bash
unsloth train --config config.yaml
unsloth train --config config.yaml --dry-run
```

CLI flags (override config values). Use non-GGUF model IDs for training.

```bash
unsloth train \
  --model "unsloth/Qwen3-0.6B" \
  --dataset "tatsu-lab/alpaca" \
  --training-type lora \
  --num-epochs 3 \
  --learning-rate 2e-4 \
  --batch-size 2 \
  --gradient-accumulation-steps 4 \
  --max-seq-length 2048 \
  --load-in-4bit \
  --lora-r 64 --lora-alpha 16 \
  --output-dir ./outputs \
  --hf-token $HF_TOKEN \
  --wandb-token $WANDB_API_KEY
```

- `--dry-run` prints resolved config as YAML and exits. Does NOT validate required fields.
- `--local-dataset` has no CLI flag — set `local_dataset` as a list in the YAML config.
- `--format-type` auto | alpaca | chatml | sharegpt (default: auto)
- `--warmup-steps` (default: 5), `--max-steps` (default: 0, uses num-epochs), `--save-steps` (default: 0)
- `--weight-decay` (default: 0.01), `--random-seed` (default: 3407)
- `--packing`, `--train-on-completions`, `--gradient-checkpointing` unsloth | true | none
- `--lora-dropout` (default: 0.0), `--target-modules` (default: q/k/v/o/gate/up/down_proj)
- `--vision-all-linear`, `--finetune-vision-layers`, `--finetune-language-layers`
- `--enable-wandb`, `--wandb-project`, `--enable-tensorboard`, `--tensorboard-dir`

## Inference

Two positional args: `MODEL` and `PROMPT`. Use GGUF models on macOS / CPU-only systems.

```bash
unsloth inference "unsloth/Qwen3.5-4B-GGUF" "What is AI?"
```

- `--temperature` (float, default: 0.7) — sampling temperature
- `--top-p` (float, default: 0.9) — nucleus sampling
- `--top-k` (int, default: 40) — top-k sampling
- `--max-new-tokens` (int, default: 256) — max tokens to generate
- `--repetition-penalty` (float, default: 1.1) — penalty for repeated tokens
- `--system-prompt` (str) — optional system prompt to prepend
- `--max-seq-length` (int, default: 2048) — max sequence length
- `--load-in-4bit` (default: on) — 4-bit quantization

## Export

```bash
unsloth list-checkpoints --outputs-dir ./outputs
unsloth export ./outputs/checkpoint-100 ./exported \
  --format gguf \
  --quantization q4_k_m
unsloth export ./outputs/checkpoint-100 ./exported \
  --format lora \
  --push-to-hub --repo-id user/model-name \
  --hf-token $HF_TOKEN \
  --private
```

- `--format` merged-16bit | merged-4bit | gguf | lora
- `--quantization` q4_k_m | q5_k_m | q8_0 | f16 (gguf only)
- `--push-to-hub` requires `--repo-id`
- `--max-seq-length` (default: 2048), `--load-in-4bit` / `--no-load-in-4bit`
- On success, prints `Saved to: <path>` with the output file/directory location

## Studio Server

```bash
unsloth studio update
unsloth studio -H 0.0.0.0 -p 8000
unsloth studio --silent
unsloth studio stop
unsloth studio reset-password
```

### One-liner API server (`studio run`)

Start Studio, load a model, and get an API key — one command:

```bash
unsloth studio run --model unsloth/Qwen3-1.7B-GGUF --gguf-variant UD-Q4_K_XL
```

Prints ready-to-use curl examples for OpenAI, Anthropic, and Responses endpoints at `http://<host>:<port>/v1/`.

- `--model` / `-m` (required) — HF repo or local path
- `--gguf-variant` — GGUF quantization variant (e.g. `UD-Q4_K_XL`)
- `--max-seq-length` (default: 0 = model default)
- `--load-in-4bit` / `--no-load-in-4bit` (default: on)
- `--api-key-name` (default: "cli") — label for the auto-generated API key
- `--port` / `-p` (default: 8888), `--host` / `-H` (default: 0.0.0.0)
- `--silent` / `-q` — suppress banner output

## Key Gotchas

- `local_dataset` is a list field — there is **no CLI flag** for it. Always set it in the YAML config.
- `--dry-run` prints the resolved config but does **not** validate required fields (model, dataset can be null). Check the printed YAML yourself.
- Config file approach is preferred for agents (handles lists, avoids shell quoting issues)
- Use **non-GGUF** model IDs for training (e.g. `unsloth/Qwen3-0.6B`, not `-GGUF`)
- For inference, **GGUF** model IDs/files use `llama-server` while non-GGUF model IDs use the standard Unsloth backend; on macOS or other chat-only environments, prefer GGUF models.
- CLI commands auto-relaunch inside `~/.unsloth/studio/unsloth_studio` ("Launching with studio venv..." in output). Dependencies come from the studio venv, not the user's environment.
- Studio home directory: `~/.unsloth/studio/`, default training output: `./outputs`
- CLI flags override config file values (precedence: CLI > config > defaults)
- Boolean flags use `--flag / --no-flag` pattern (e.g. `--load-in-4bit / --no-load-in-4bit`)
- `--push-to-hub` requires `--repo-id`

# runpodctl (Apache-2.0)

Manage GPU pods, serverless endpoints, templates, network volumes, and models on Runpod.
https://github.com/runpod/skills/blob/main/runpodctl/SKILL.md

> **Spelling:** "Runpod" (capital R). Command is `runpodctl` (lowercase).

## Install

```bash
# Any platform (official installer)
curl -sSL https://cli.runpod.net | bash

# macOS (Homebrew)
brew install runpod/runpodctl/runpodctl

# macOS (manual — universal binary)
mkdir -p ~/.local/bin && curl -sL https://github.com/runpod/runpodctl/releases/latest/download/runpodctl-darwin-all.tar.gz | tar xz -C ~/.local/bin

# Linux
mkdir -p ~/.local/bin && curl -sL https://github.com/runpod/runpodctl/releases/latest/download/runpodctl-linux-amd64.tar.gz | tar xz -C ~/.local/bin

# Windows (PowerShell)
Invoke-WebRequest -Uri https://github.com/runpod/runpodctl/releases/latest/download/runpodctl-windows-amd64.zip -OutFile runpodctl.zip; Expand-Archive runpodctl.zip -DestinationPath $env:LOCALAPPDATA\runpodctl; [Environment]::SetEnvironmentVariable('Path', $env:Path + ";$env:LOCALAPPDATA\runpodctl", 'User')
```

Ensure `~/.local/bin` is on your `PATH` (add `export PATH="$HOME/.local/bin:$PATH"` to `~/.bashrc` or `~/.zshrc`).

## Quick start

```bash
runpodctl doctor                    # First time setup (API key + SSH)
runpodctl gpu list                  # See available GPUs
runpodctl template search pytorch   # Find a template
runpodctl pod create --template-id runpod-torch-v21 --gpu-id "NVIDIA GeForce RTX 4090"  # Create from template
runpodctl pod list                  # List your pods
```

API key: https://runpod.io/console/user/settings

## Commands

### Pods

```bash
runpodctl pod list                                    # List running pods (default, like docker ps)
runpodctl pod list --all                              # List all pods including exited
runpodctl pod list --status exited                    # Filter by status (RUNNING, EXITED, etc.)
runpodctl pod list --since 24h                        # Pods created within last 24 hours
runpodctl pod list --created-after 2025-01-15         # Pods created after date
runpodctl pod get <pod-id>                            # Get pod details (includes SSH info)
runpodctl pod create --template-id runpod-torch-v21 --gpu-id "NVIDIA GeForce RTX 4090"  # Create from template
runpodctl pod create --image "runpod/pytorch:1.0.3-cu1281-torch291-ubuntu2404" --gpu-id "NVIDIA GeForce RTX 4090"  # Create with image
runpodctl pod create --compute-type cpu --image ubuntu:22.04  # Create CPU pod
runpodctl pod start <pod-id>                          # Start stopped pod
runpodctl pod stop <pod-id>                           # Stop running pod
runpodctl pod restart <pod-id>                        # Restart pod
runpodctl pod reset <pod-id>                          # Reset pod
runpodctl pod update <pod-id> --name "new"            # Update pod
runpodctl pod delete <pod-id>                         # Delete pod (aliases: rm, remove)
```

**List flags:** `--all` / `-a`, `--status`, `--since`, `--created-after`, `--name`, `--compute-type`
**Get flags:** `--include-machine`, `--include-network-volume`

**Create flags:** `--template-id` (required if no `--image`), `--image` (required if no `--template-id`), `--name`, `--gpu-id`, `--gpu-count`, `--compute-type`, `--ssh` (default true), `--container-disk-in-gb`, `--volume-in-gb`, `--volume-mount-path`, `--network-volume-id`, `--ports`, `--env`, `--cloud-type`, `--data-center-ids`, `--global-networking`, `--public-ip`

### Serverless (alias: sls)

```bash
runpodctl serverless list                             # List all endpoints
runpodctl serverless get <endpoint-id>                # Get endpoint details
runpodctl serverless create --name "x" --template-id "tpl_abc"  # Create endpoint
runpodctl serverless update <endpoint-id> --workers-max 5       # Update endpoint
runpodctl serverless delete <endpoint-id>             # Delete endpoint
```

**List flags:** `--include-template`, `--include-workers`
**Update flags:** `--name`, `--workers-min`, `--workers-max`, `--idle-timeout`, `--scaler-type` (QUEUE_DELAY or REQUEST_COUNT), `--scaler-value`
**Create flags:** `--name`, `--template-id`, `--gpu-id`, `--gpu-count`, `--compute-type`, `--workers-min`, `--workers-max`, `--network-volume-id`, `--data-center-ids`

### Templates (alias: tpl)

```bash
runpodctl template list                               # Official + community (first 10)
runpodctl template list --type official               # All official templates
runpodctl template list --type community              # Community templates (first 10)
runpodctl template list --type user                   # Your own templates
runpodctl template list --all                         # Everything including user
runpodctl template list --limit 50                    # Show 50 templates
runpodctl template search pytorch                     # Search for "pytorch" templates
runpodctl template search comfyui --limit 5           # Search, limit to 5 results
runpodctl template search vllm --type official        # Search only official
runpodctl template get <template-id>                  # Get template details (includes README, env, ports)
runpodctl template create --name "x" --image "img"    # Create template
runpodctl template create --name "x" --image "img" --serverless  # Create serverless template
runpodctl template update <template-id> --name "new"  # Update template
runpodctl template delete <template-id>               # Delete template
```

**List flags:** `--type` (official, community, user), `--limit`, `--offset`, `--all`
**Create flags:** `--name`, `--image`, `--container-disk-in-gb`, `--volume-in-gb`, `--volume-mount-path`, `--ports`, `--env`, `--docker-start-cmd`, `--docker-entrypoint`, `--serverless`, `--readme`

### Network Volumes (alias: nv)

```bash
runpodctl network-volume list                         # List all volumes
runpodctl network-volume get <volume-id>              # Get volume details
runpodctl network-volume create --name "x" --size 100 --data-center-id "US-GA-1"  # Create volume
runpodctl network-volume update <volume-id> --name "new"  # Update volume
runpodctl network-volume delete <volume-id>           # Delete volume
```

**Create flags:** `--name`, `--size`, `--data-center-id`

### Models

```bash
runpodctl model list                                  # List your models
runpodctl model list --all                            # List all models
runpodctl model list --name "llama"                   # Filter by name
runpodctl model list --provider "meta"                # Filter by provider
runpodctl model add --name "my-model" --model-path ./model  # Add model
runpodctl model remove --name "my-model"              # Remove model
```

### Registry (alias: reg)

```bash
runpodctl registry list                               # List registry auths
runpodctl registry get <registry-id>                  # Get registry auth
runpodctl registry create --name "x" --username "u" --password "p"  # Create registry auth
runpodctl registry delete <registry-id>               # Delete registry auth
```

### Info

```bash
runpodctl user                                        # Account info and balance (alias: me)
runpodctl gpu list                                    # List available GPUs
runpodctl gpu list --include-unavailable              # Include unavailable GPUs
runpodctl datacenter list                             # List datacenters (alias: dc)
runpodctl billing pods                                # Pod billing history
runpodctl billing serverless                          # Serverless billing history
runpodctl billing network-volume                      # Volume billing history
```

### SSH

```bash
runpodctl ssh info <pod-id>                           # Get SSH info (command + key, does not connect)
runpodctl ssh list-keys                               # List SSH keys
runpodctl ssh add-key                                 # Add SSH key
```

**Agent note:** `ssh info` returns connection details, not an interactive session. If interactive SSH is not available, execute commands remotely via `ssh user@host "command"`.

### File Transfer

```bash
runpodctl send <path>                                 # Send files (outputs code)
runpodctl receive <code>                              # Receive files using code
```

### Utilities

```bash
runpodctl doctor                                      # Diagnose and fix CLI issues
runpodctl update                                      # Update CLI
runpodctl version                                     # Show version
runpodctl completion                                  # Auto-detect shell and install completion
```

## URLs

### Pod URLs

Access exposed ports on your pod:

```
https://<pod-id>-<port>.proxy.runpod.net
```

Example: `https://abc123xyz-8888.proxy.runpod.net`

### Serverless URLs

```
https://api.runpod.ai/v2/<endpoint-id>/run        # Async request
https://api.runpod.ai/v2/<endpoint-id>/runsync    # Sync request
https://api.runpod.ai/v2/<endpoint-id>/health     # Health check
https://api.runpod.ai/v2/<endpoint-id>/status/<job-id>  # Job status
```

# agent-media (Apache-2.0)

Agent-first media toolkit for image, video, and audio processing. All commands return deterministic JSON output.
https://github.com/agntswrm/agent-media/blob/main/skills/agent-media/SKILL.md

Requires Node.js 18+. Run via `npx agent-media@latest` or `npm install -g agent-media`.

## Available Commands

### Image Commands
- `npx agent-media@latest image resize` - Resize an image
- `npx agent-media@latest image convert` - Convert image format
- `npx agent-media@latest image generate` - Generate image from text
- `npx agent-media@latest image edit` - Edit one or more images with text prompt
- `npx agent-media@latest image remove-background` - Remove image background
- `npx agent-media@latest image upscale` - Upscale image with AI super-resolution
- `npx agent-media@latest image extend` - Extend image canvas with padding
- `npx agent-media@latest image crop` - Crop image to dimensions around focal point

### Audio Commands
- `npx agent-media@latest audio extract` - Extract audio from video
- `npx agent-media@latest audio transcribe` - Transcribe audio to text

### Video Commands
- `npx agent-media@latest video generate` - Generate video from text or image

## Output Format

All commands return JSON to stdout:

```json
{
  "ok": true,
  "media_type": "image",
  "action": "resize",
  "provider": "local",
  "output_path": "output_123.webp",
  "mime": "image/webp",
  "bytes": 12345
}
```

On error:

```json
{
  "ok": false,
  "error": {
    "code": "INVALID_INPUT",
    "message": "input file not found"
  }
}
```

## Providers

- **local** - Default provider using Sharp (resize, convert, extend, crop) and Transformers.js (remove-background, upscale, transcribe)
- **fal** - fal.ai provider (generate, edit, remove-background, upscale, transcribe, video)
- **replicate** - Replicate API (generate, edit, remove-background, upscale, transcribe, video)
- **runpod** - Runpod API (generate, edit, video)
- **ai-gateway** - Vercel AI Gateway (generate, edit)

## Provider Selection

1. Explicit: `--provider <name>`
2. Auto-detect from environment variables
3. Fallback to local provider

## Environment Variables

- `AGENT_MEDIA_DIR` - Custom output directory
- `FAL_API_KEY` - Enable fal provider
- `REPLICATE_API_TOKEN` - Enable replicate provider
- `RUNPOD_API_KEY` - Enable runpod provider
- `AI_GATEWAY_API_KEY` - Enable ai-gateway provider

# a2go (MIT)

Use open weight models (LLM, image, audio) with open source agents on Mac, Linux, and Windows.
https://a2go.run · https://github.com/runpod-labs/a2go/blob/main/skills/a2go/SKILL.md

## Prerequisites

Requires the `a2go` CLI. Install from GitHub releases (includes SHA256 checksums for verification): https://github.com/runpod-labs/a2go/releases

## Quick start

```bash
a2go doctor                                              # One-time setup (checks Docker, GPU, pulls image)
a2go run --agent hermes --llm <repo>:<bits>bit          # Start with a model
a2go status                                              # Check running services
a2go stop                                                # Stop all
```

Pick a model value with `a2go models`; use the `repo:bits` value from the output.

## Commands

```bash
a2go run --agent <agent> --llm <repo>:<bits>bit [--image <repo>] [--audio <repo>:<bits>bit] [--engine <engine>]
a2go doctor                                              # Prereq check + image pull
a2go status                                              # Service health
a2go stop                                                # Stop containers
```

Agents: `hermes` (recommended) or `openclaw`.

## Engines

- **llama.cpp** — NVIDIA GPU (CUDA). Default on Linux/Windows. Uses GGUF models.
- **MLX** — Apple Silicon. Default on Mac. Uses MLX models.
- **wandler** — ONNX runtime. Works on all platforms (CUDA on Linux, CPU fallback). Uses ONNX models. Pass `--engine wandler` to use it.

The engine is auto-detected from the model when possible. Use `--engine` to override (e.g. `--engine wandler` for ONNX models).

## Ports

- **8000** — LLM API (direct model access, use for testing chat completions)
- **8080** — Web proxy / media server (TTS, STT, image gen, web UI)
- **8642** — Hermes Gateway (agent orchestration, not for direct API calls)
- **18789** — OpenClaw Gateway (agent orchestration, not for direct API calls)

For direct LLM testing use port **8000** (`/v1/chat/completions`). For TTS/STT use port **8080** (`/v1/audio/speech`, `/v1/audio/transcriptions`). The gateway ports (8642/18789) are for platform integrations.

## Models

```bash
a2go models                                # All models
a2go models --type llm                     # LLMs only
a2go models --engine wandler               # Wandler/ONNX models only
a2go models --os mac                       # Mac/MLX models only
a2go models --max-vram 24                  # Fits in 24GB GPU
a2go models --type llm --engine wandler    # Wandler LLMs only
```

Output: `type | engine | os | vram | context | repo:bits | name` — use `repo:bits` as the `--llm`/`--image`/`--audio` value.

## Docker

Image `runpod/a2go:latest`, configured via `A2GO_CONFIG` env var:

```json
{"agent":"openclaw", "engine":"wandler", "llm":"onnx-community/gemma-4-E4B-it-ONNX:4bit"}
```

Fields:
- `agent` — `hermes` or `openclaw` (required)
- `engine` — `llamacpp`, `mlx`, or `wandler`
- `llm` — model as `repo:bits` from `a2go models`
- `audio` — audio model (optional)
- `image` — image model (optional)
- `contextLength` — override context window (optional)

Env vars: `A2GO_AUTH_TOKEN` (gateway auth), `A2GO_API_KEY` (LLM API auth).

## Notes

- **Mac/Apple Silicon:** `a2go run` runs natively via MLX (no Docker). Wandler models also work on Mac.
- **Browse models visually:** https://a2go.run

# wandler (MIT)

transformers.js inference server with OpenAI-compatible API, written in TypeScript.
https://wandler.ai · https://github.com/runpod-labs/wandler/blob/main/skills/wandler/SKILL.md

`npm install -g wandler` or `npx wandler --llm <org/repo:precision>`

```bash
# LLM
wandler --llm onnx-community/gemma-4-E4B-it-ONNX:q4
# LLM on CPU with fp16
wandler --llm LiquidAI/LFM2.5-1.2B-Instruct-ONNX:fp16 --device cpu
# LLM + embeddings
wandler --llm onnx-community/Qwen3.5-0.8B-Text-ONNX:q4 --embedding Xenova/all-MiniLM-L6-v2:q8
# LLM + embeddings + STT
wandler --llm onnx-community/gemma-4-E4B-it-ONNX:q4 --embedding Xenova/all-MiniLM-L6-v2:q8 --stt onnx-community/whisper-tiny:q4
# custom port, auth, listen on all interfaces
wandler --llm LiquidAI/LFM2.5-1.2B-Instruct-ONNX:q4 --port 3000 --host 0.0.0.0 --api-key mysecret

# --llm <id>           LLM model
# --embedding <id>     Embedding model
# --stt <id>           STT model
# --device <type>      auto | webgpu | cpu | wasm (default: auto)
# --port <n>           Default: 8000
# --host <addr>        Default: 127.0.0.1
# --api-key <key>      Bearer auth (or env WANDLER_API_KEY)
# --hf-token <token>   HuggingFace token for gated models
# --cors-origin <o>    Allowed CORS origin (default: *)
# --max-tokens <n>     Max tokens per request (default: model's max context)
# --max-concurrent <n> Concurrent requests (default: 1)
# --timeout <ms>       Request timeout (default: 120000)
# --log-level <l>      debug | info | warn | error (default: info)
# --cache-dir <path>   Model cache directory (default: ~/.cache/huggingface)
# Precision suffixes:  q4 (default) | q8 | fp16 | fp32

# list all models from the wandler registry
# returns: type, size, precision, capabilities, repo:precision, name
# --type: llm | embedding | stt
wandler model ls
```

Server at `http://127.0.0.1:8000`.

## API (OpenAI-compatible)

- `POST /v1/chat/completions` — streaming + non-streaming
- `POST /v1/completions`
- `POST /v1/embeddings`
- `POST /v1/audio/transcriptions`
- `GET /v1/models`
- `POST /tokenize`
- `POST /detokenize`
- `GET /admin/metrics`
- `GET /health`

## Gotchas

- Tool calling disables true streaming — full response generated first, then sent as SSE.
- `stop` sequences only match on the last token. Multi-token stops won't match exactly.

# agnt-init (Apache-2.0)

Initialize a project for agents by creating AGENTS.md and symlinking CLAUDE.md to it.
https://github.com/agntswrm/agnt-init/blob/main/agnt-init/SKILL.md

- Find the project root (`.git`, `package.json`, `go.mod`, or current working directory)
- If `AGENTS.md` or `CLAUDE.md` already exist, ask the user before overwriting
- Create `AGENTS.md` with this exact content:

```markdown
<!-- Do not edit or remove this section -->
This document exists for non-obvious, error-prone shortcomings in the codebase, the model, or the tooling that an agent cannot figure out by reading the code alone. No architecture overviews, file trees, build commands, or standard behavior. When you encounter something that belongs here, first consider whether a code change could eliminate it and suggest that to the user. Only document it here if it can't be reasonably fixed.

---
```

- Symlink `CLAUDE.md` to `AGENTS.md`

# agnt-prune (Apache-2.0)

Prune AGENTS.md / CLAUDE.md down to only sharp edges and gotchas by removing everything inferable from the codebase.
https://github.com/agntswrm/agnt-prune/blob/main/agnt-prune/SKILL.md

## steps

- Find the project root (`.git`, `package.json`, `go.mod`, or current working directory)
- Look for `AGENTS.md` or `CLAUDE.md` (resolve symlinks). If neither exists, tell the user and stop
- Read the full file content
- Read the codebase to understand what can be inferred from the code itself
- Apply this pruning rule to the content:

> Remove everything from the file that can be inferred from the codebase, including high-level architecture descriptions, file trees, CLI usage, build commands, and examples of standard behavior. Keep only non-obvious, failure-prone decisions and hidden constraints that are not explicit in the code but would cause mistakes if misunderstood. The final file should read like a sharp-edges and gotchas document, not a project overview.

- Preserve the header comment if it exists:

```markdown
<!-- Do not edit or remove this section -->
This document exists for non-obvious, error-prone shortcomings in the codebase, the model, or the tooling that an agent cannot figure out by reading the code alone. No architecture overviews, file trees, build commands, or standard behavior. When you encounter something that belongs here, first consider whether a code change could eliminate it and suggest that to the user. Only document it here if it can't be reasonably fixed.

---
```

- If the header comment does not exist, add it at the top of the file
- Show the user a diff of what will be removed and ask for confirmation before writing
- Write the pruned content back to the same file

# commit (MIT)

Create git commits using conventional commits (angular style). Use only when the user explicitly asks to commit.
https://github.com/TimPietrusky/skills/blob/main/skills/commit/SKILL.md

create a git commit following conventional commits with angular style, everything lowercase.

**NEVER commit automatically** - only commit when the user explicitly asks you to.

## Mandatory process

1. **Check all changed files**: Run `git status --short` to see all modified, added, and deleted files
2. **Review actual changes**: Run `git diff --stat` to see file-level changes, then `git diff` for detailed changes
3. **Understand the full scope**: Read through the diffs to understand what was actually changed, not just what you remember
4. **Identify the primary purpose**: Determine the main goal/feature/fix that drove these changes
5. **Choose correct commit type**: `fix`, `feat`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `build`, `ci`
6. **Use the correct format**: `<type>(<scope>): <subject>`
7. **Concise subject**: Keep subject line concise and descriptive (under 72 characters when possible), use present tense ("add" not "added", "fix" not "fixed").
8. **Create comprehensive message**: The commit message should reflect all significant changes, not just the most recent ones

## Common mistakes to avoid

- Creating commit message based only on the last change you made
- Using wrong commit type (e.g., `refactor` when it's actually a `feat`)
- Ignoring major changes because they're not in the most recent edits
- Grouping unrelated changes into one commit (should be multiple commits)
