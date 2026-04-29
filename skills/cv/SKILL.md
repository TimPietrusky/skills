---
name: cv
description: Tim Pietrusky's cv of shipped tools as skills (agent-media, wandler, a2go, runpodctl, agnt-prune, unsloth, agnt-init, commit).
metadata:
  author: Tim Pietrusky
license: MIT
---

Tools as skills shipped by Tim Pietrusky.

# agent-media (Apache-2.0)

Image, video, and audio toolkit. All commands return deterministic JSON output.
https://github.com/agntswrm/agent-media/blob/main/skills/agent-media/SKILL.md

Requires Node.js 18+. Run via `npx agent-media@latest` or `npm install -g agent-media`.

## Commands

### Image

```bash
npx agent-media@latest image resize --in photo.jpg --width 800 --height 600 --out ./resized --provider local # resize — at least one of --width/--height required
npx agent-media@latest image convert --in photo.png --format webp --quality 95 --out ./converted --provider local # convert — --format png|jpg|webp; --quality 1-100 (lossy only, default 80)
npx agent-media@latest image generate --prompt "a red robot in a forest" --width 1024 --height 768 --count 1 --out ./generated --provider ai-gateway # generate — text-to-image (requires fal|replicate|runpod|ai-gateway, no local)
npx agent-media@latest image edit --in template.png person.jpg --prompt "place the person into the template" --aspect-ratio 1:1 --resolution 2K --model fal-ai/nano-banana-pro/edit --out ./edited --provider fal # edit — image-to-image; --in accepts multiple paths to combine images
npx agent-media@latest image remove-background --in portrait.jpg --resolution 2048x2048 --out ./nobg --provider fal # remove-background — --resolution only honored by fal Dynamic model
npx agent-media@latest image upscale --in photo.jpg --scale 4 --model nightmareai/real-esrgan --out ./upscaled --provider replicate # upscale — --scale 2|4 (local always 4x); replicate supports 2-10x; --model overrides provider default
npx agent-media@latest image extend --in photo.jpg --padding 50 --color "#FFFFFF" --dpi 300 --out ./extended # extend — solid-color padding; --color also flattens transparency
npx agent-media@latest image crop --in photo.jpg --width 800 --height 600 --focus-x 20 --focus-y 30 --dpi 300 --out ./cropped --provider local # crop — --focus-x/--focus-y 0-100 (50=center)
```

### Audio

```bash
npx agent-media@latest audio extract --in video.mp4 --format mp3 --out ./audio # extract — --format mp3|wav (default mp3); local only, bundled ffmpeg
npx agent-media@latest audio transcribe --in podcast.mp3 --language en --out ./transcripts --provider runpod # transcribe — --language fixes lang (else auto-detect); --diarize/--speakers requires fal|replicate
```

### Video

```bash
npx agent-media@latest video generate --prompt "a cat walking in a garden" --in portrait.png --duration 10 --resolution 1080p --fps 25 --audio --model lightricks/ltx-video --out ./videos --provider replicate # generate — text-to-video; pass --in to animate a static image; --audio enables audio track
```

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

- **local** (default, no key) — Sharp: resize, convert, extend, crop. Transformers.js: remove-background, upscale, transcribe.
- **fal** — `FAL_API_KEY`. generate, edit, remove-background, upscale, transcribe, video.
- **replicate** — `REPLICATE_API_TOKEN`. generate, edit, remove-background, upscale, transcribe, video.
- **runpod** — `RUNPOD_API_KEY`. generate, edit, video, transcribe.
- **ai-gateway** — `AI_GATEWAY_API_KEY`. generate, edit.

Selection: explicit `--provider <name>` → auto-detect from env vars → local fallback.

## Environment Variables

- `AGENT_MEDIA_DIR` — custom output directory
- `FAL_API_KEY` / `REPLICATE_API_TOKEN` / `RUNPOD_API_KEY` / `AI_GATEWAY_API_KEY` — provider auth

# wandler (MIT)

transformers.js inference server with OpenAI-compatible API, written in TypeScript.
https://wandler.ai · https://github.com/runpod-labs/wandler/blob/main/skills/wandler/SKILL.md

`npm install -g wandler` or `npx wandler --llm <org/repo:precision>`

```bash
wandler --llm onnx-community/gemma-4-E4B-it-ONNX:q4 # LLM
wandler --llm LiquidAI/LFM2.5-1.2B-Instruct-ONNX:fp16 --device cpu # LLM on CPU with fp16
wandler --llm onnx-community/Qwen3.5-0.8B-Text-ONNX:q4 --embedding Xenova/all-MiniLM-L6-v2:q8 # LLM + embeddings
wandler --llm onnx-community/gemma-4-E4B-it-ONNX:q4 --embedding Xenova/all-MiniLM-L6-v2:q8 --stt onnx-community/whisper-tiny:q4 # LLM + embeddings + STT
wandler --llm LiquidAI/LFM2.5-1.2B-Instruct-ONNX:q4 --port 3000 --host 0.0.0.0 --api-key mysecret # custom port, auth, listen on all interfaces
```

- `--llm <id>` — LLM model (env `WANDLER_LLM`)
- `--backend <name>` — `wandler` | `transformersjs` (default: `wandler`; env `WANDLER_BACKEND`)
- `--embedding <id>` — embedding model (env `WANDLER_EMBEDDING`)
- `--stt <id>` — STT model (env `WANDLER_STT`)
- `--device <type>` — `auto` | `cuda` | `coreml` | `dml` | `webgpu` | `cpu` | `wasm` (default: `auto`)
- `--port <n>` — default: 8000
- `--host <addr>` — default: `127.0.0.1`
- `--api-key <key>` — bearer auth (env `WANDLER_API_KEY`)
- `--hf-token <token>` — HuggingFace token for gated models
- `--cors-origin <o>` — allowed CORS origin (default: `*`)
- `--max-tokens <n>` — max tokens per request (default: loaded model context)
- `--max-concurrent <n>` — concurrent requests (default: 1)
- `--timeout <ms>` — request timeout (default: 120000)
- `--log-level <l>` — `debug` | `info` | `warn` | `error` (default: `info`)
- `--quiet` — suppress non-error startup/profile logs (env `WANDLER_QUIET`)
- `--cache-dir <path>` — model cache directory (default: HF cache)
- `--prefill-chunk-size <n>` — `auto` | `auto:<mb>` | `0`/`off` | integer chunk size (auto uses a 640MB WebGPU attention budget; other backends use 1024)
- `--prefix-cache <mode>` — `true` | `false` (default: `true`; env `WANDLER_PREFIX_CACHE`)
- `--prefix-cache-entries <n>` — prefix KV cache entries (default: 2)
- `--prefix-cache-min-tokens <n>` — minimum prefix tokens to cache (default: 512)
- `--warmup-tokens <n>` — approximate startup warmup prompt tokens (default: 0)
- `--warmup-max-new-tokens <n>` — startup warmup max new tokens (default: 8)

Precision suffixes: `q4` (default) | `q8` | `fp16` | `fp32`.

Server at `http://127.0.0.1:8000`.

## list models

```bash
wandler model ls # all models from the wandler registry
wandler model ls --type llm # filter by type: llm | embedding | stt
```

Returns: `type | size | precision | capabilities | repo:precision | name`.

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

## gotchas

- Tool calling disables true streaming — full response generated first, then sent as SSE.
- `stop` sequences only match on the last token. Multi-token stops won't match exactly.

# a2go (MIT)

Use open weight models (LLM, image, audio) with open source agents on Mac, Linux, and Windows.
https://a2go.run · https://github.com/runpod-labs/a2go/blob/main/skills/a2go/SKILL.md

## Prerequisites

Requires the `a2go` CLI. Install from GitHub releases (includes SHA256 checksums for verification): https://github.com/runpod-labs/a2go/releases

## Quick start

```bash
a2go doctor # one-time setup (checks Docker, GPU, pulls image)
a2go run --agent hermes --llm <repo>:<bits>bit # start with a model
a2go status # check running services
a2go stop # stop all
```

Pick a model value with `a2go models`; use the `repo:bits` value from the output.

## Commands

```bash
a2go run --agent <agent> --llm <repo>:<bits>bit [--image <repo>] [--audio <repo>:<bits>bit] [--engine <engine>]
a2go doctor # prereq check + image pull
a2go status # service health
a2go stop # stop containers
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
a2go models # all models
a2go models --type llm # LLMs only
a2go models --engine wandler # wandler/ONNX models only
a2go models --os mac # mac/MLX models only
a2go models --max-vram 24 # fits in 24GB GPU
a2go models --type llm --engine wandler # wandler LLMs only
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

# runpodctl (Apache-2.0)

Manage GPU pods, serverless endpoints, templates, network volumes, and models on Runpod.
https://github.com/runpod/skills/blob/main/runpodctl/SKILL.md

> **Spelling:** "Runpod" (capital R). Command is `runpodctl` (lowercase).

## Install

```bash
curl -sSL https://cli.runpod.net | bash # any platform (official installer)
brew install runpod/runpodctl/runpodctl # macOS (Homebrew)
mkdir -p ~/.local/bin && curl -sL https://github.com/runpod/runpodctl/releases/latest/download/runpodctl-darwin-all.tar.gz | tar xz -C ~/.local/bin # macOS (manual — universal binary)
mkdir -p ~/.local/bin && curl -sL https://github.com/runpod/runpodctl/releases/latest/download/runpodctl-linux-amd64.tar.gz | tar xz -C ~/.local/bin # linux
Invoke-WebRequest -Uri https://github.com/runpod/runpodctl/releases/latest/download/runpodctl-windows-amd64.zip -OutFile runpodctl.zip; Expand-Archive runpodctl.zip -DestinationPath $env:LOCALAPPDATA\runpodctl; [Environment]::SetEnvironmentVariable('Path', $env:Path + ";$env:LOCALAPPDATA\runpodctl", 'User') # windows (PowerShell)
```

Ensure `~/.local/bin` is on your `PATH` (add `export PATH="$HOME/.local/bin:$PATH"` to `~/.bashrc` or `~/.zshrc`).

## Quick start

```bash
runpodctl doctor # first time setup (API key + SSH)
runpodctl gpu list # see available GPUs
runpodctl template search pytorch # find a template
runpodctl pod create --template-id runpod-torch-v21 --gpu-id "NVIDIA GeForce RTX 4090" # create from template
runpodctl pod list # list your pods
```

API key: https://runpod.io/console/user/settings

## Commands

### Pods

```bash
runpodctl pod list # list running pods (default, like docker ps)
runpodctl pod list --all # list all pods including exited
runpodctl pod list --status exited # filter by status (RUNNING, EXITED, etc.)
runpodctl pod list --since 24h # pods created within last 24 hours
runpodctl pod list --created-after 2025-01-15 # pods created after date
runpodctl pod get <pod-id> # get pod details (includes SSH info)
runpodctl pod create --template-id runpod-torch-v21 --gpu-id "NVIDIA GeForce RTX 4090" # create from template
runpodctl pod create --image "runpod/pytorch:1.0.3-cu1281-torch291-ubuntu2404" --gpu-id "NVIDIA GeForce RTX 4090" # create with image
runpodctl pod create --compute-type cpu --image ubuntu:22.04 # create CPU pod
runpodctl pod start <pod-id> # start stopped pod
runpodctl pod stop <pod-id> # stop running pod
runpodctl pod restart <pod-id> # restart pod
runpodctl pod reset <pod-id> # reset pod
runpodctl pod update <pod-id> --name "new" # update pod
runpodctl pod delete <pod-id> # delete pod (aliases: rm, remove)
```

**List flags:** `--all` / `-a`, `--status`, `--since`, `--created-after`, `--name`, `--compute-type`
**Get flags:** `--include-machine`, `--include-network-volume`

**Create flags:** `--template-id` (required if no `--image`), `--image` (required if no `--template-id`), `--name`, `--gpu-id`, `--gpu-count`, `--compute-type`, `--ssh` (default true), `--container-disk-in-gb`, `--volume-in-gb`, `--volume-mount-path`, `--network-volume-id`, `--ports`, `--env`, `--cloud-type`, `--data-center-ids`, `--global-networking`, `--public-ip`

### Serverless (alias: sls)

```bash
runpodctl serverless list # list all endpoints
runpodctl serverless get <endpoint-id> # get endpoint details
runpodctl serverless create --name "x" --template-id "tpl_abc" # create endpoint
runpodctl serverless update <endpoint-id> --workers-max 5 # update endpoint
runpodctl serverless delete <endpoint-id> # delete endpoint
```

**List flags:** `--include-template`, `--include-workers`
**Update flags:** `--name`, `--workers-min`, `--workers-max`, `--idle-timeout`, `--scaler-type` (QUEUE_DELAY or REQUEST_COUNT), `--scaler-value`
**Create flags:** `--name`, `--template-id`, `--gpu-id`, `--gpu-count`, `--compute-type`, `--workers-min`, `--workers-max`, `--network-volume-id`, `--data-center-ids`

### Templates (alias: tpl)

```bash
runpodctl template list # official + community (first 10)
runpodctl template list --type official # all official templates
runpodctl template list --type community # community templates (first 10)
runpodctl template list --type user # your own templates
runpodctl template list --all # everything including user
runpodctl template list --limit 50 # show 50 templates
runpodctl template search pytorch # search for "pytorch" templates
runpodctl template search comfyui --limit 5 # search, limit to 5 results
runpodctl template search vllm --type official # search only official
runpodctl template get <template-id> # get template details (includes README, env, ports)
runpodctl template create --name "x" --image "img" # create template
runpodctl template create --name "x" --image "img" --serverless # create serverless template
runpodctl template update <template-id> --name "new" # update template
runpodctl template delete <template-id> # delete template
```

**List flags:** `--type` (official, community, user), `--limit`, `--offset`, `--all`
**Create flags:** `--name`, `--image`, `--container-disk-in-gb`, `--volume-in-gb`, `--volume-mount-path`, `--ports`, `--env`, `--docker-start-cmd`, `--docker-entrypoint`, `--serverless`, `--readme`

### Network Volumes (alias: nv)

```bash
runpodctl network-volume list # list all volumes
runpodctl network-volume get <volume-id> # get volume details
runpodctl network-volume create --name "x" --size 100 --data-center-id "US-GA-1" # create volume
runpodctl network-volume update <volume-id> --name "new" # update volume
runpodctl network-volume delete <volume-id> # delete volume
```

**Create flags:** `--name`, `--size`, `--data-center-id`

### Models

```bash
runpodctl model list # list your models
runpodctl model list --all # list all models
runpodctl model list --name "llama" # filter by name
runpodctl model list --provider "meta" # filter by provider
runpodctl model add --name "my-model" --model-path ./model # add model
runpodctl model remove --name "my-model" # remove model
```

### Registry (alias: reg)

```bash
runpodctl registry list # list registry auths
runpodctl registry get <registry-id> # get registry auth
runpodctl registry create --name "x" --username "u" --password "p" # create registry auth
runpodctl registry delete <registry-id> # delete registry auth
```

### Info

```bash
runpodctl user # account info and balance (alias: me)
runpodctl gpu list # list available GPUs
runpodctl gpu list --include-unavailable # include unavailable GPUs
runpodctl datacenter list # list datacenters (alias: dc)
runpodctl billing pods # pod billing history
runpodctl billing serverless # serverless billing history
runpodctl billing network-volume # volume billing history
```

### SSH

```bash
runpodctl ssh info <pod-id> # get SSH info (command + key, does not connect)
runpodctl ssh list-keys # list SSH keys
runpodctl ssh add-key # add SSH key
```

**Agent note:** `ssh info` returns connection details, not an interactive session. If interactive SSH is not available, execute commands remotely via `ssh user@host "command"`.

### File Transfer

```bash
runpodctl send <path> # send files (outputs code)
runpodctl receive <code> # receive files using code
```

### Utilities

```bash
runpodctl doctor # diagnose and fix CLI issues
runpodctl update # update CLI
runpodctl version # show version
runpodctl completion # auto-detect shell and install completion
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
https://api.runpod.ai/v2/<endpoint-id>/run # async request
https://api.runpod.ai/v2/<endpoint-id>/runsync # sync request
https://api.runpod.ai/v2/<endpoint-id>/health # health check
https://api.runpod.ai/v2/<endpoint-id>/status/<job-id> # job status
```

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

## References

- `references/config-reference.md` — every config field with type, default, and description
- `assets/lora-text-train.yaml` — copy-paste config template for LoRA text fine-tuning (most common)
- `assets/full-finetune.yaml` — copy-paste config template for full fine-tuning (no LoRA, more VRAM)
- `assets/vision-lora-train.yaml` — copy-paste config template for vision model LoRA

## gotchas

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
