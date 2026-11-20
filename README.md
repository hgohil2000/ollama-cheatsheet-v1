# Ollama & `qwen3.8:27b-mlx` Complete Terminal Cheatsheet

The Ollama cheatsheet is generic but for this specific AI model (i.e. qwen3.8:27b-mlx), the MLX is Apple's open-source machine learning framework built specifically for Apple silicon, letting you run and fine-tune AL models locally without copying data back and forth between CPU and GPU memory.  Because it uses unified memory and dedicated chip accelerators, it often runs models faster and more memory-efficiently on Macs than cross-platform tools, though it only works on Apple hardware.

---

## 1. Core Ollama CLI Commands

| Command | Alias | Description | Example |
| :--- | :--- | :--- | :--- |
| `ollama serve` | `ollama start` | Starts the Ollama REST API server and inference engine in the foreground (`127.0.0.1:11434`). | `ollama serve` |
| `ollama run` | — | Pulls (if missing) and starts an interactive session, or executes a single-shot prompt. | `ollama run qwen3.8:27b-mlx` |
| `ollama pull` | — | Downloads or updates a model from the Ollama library or Hugging Face without running it. | `ollama pull qwen3.8:27b-mlx` |
| `ollama push` | — | Uploads a locally created or tagged model to your remote Ollama registry namespace. | `ollama push user/qwen3.8-custom:v1` |
| `ollama list` | `ollama ls` | Lists all models stored locally on disk with their ID, size, and modified date. | `ollama list` |
| `ollama ps` | — | Shows models currently loaded in RAM/Unified Memory, active context size, and unload timer. | `ollama ps` |
| `ollama stop` | — | Immediately unloads a running model from memory without stopping the Ollama server. | `ollama stop qwen3.8:27b-mlx` |
| `ollama show` | — | Inspects metadata, capabilities, parameters, prompt templates, and Modelfile details. | `ollama show qwen3.8:27b-mlx` |
| `ollama create` | — | Builds a new custom model variant from a `Modelfile`. | `ollama create qwen-32k -f Modelfile` |
| `ollama cp` | — | Creates a lightweight local alias/copy of a model without duplicating weight blobs on disk. | `ollama cp qwen3.8:27b-mlx qwen-backup` |
| `ollama rm` | — | Deletes one or more models from local disk storage. | `ollama rm mistral:latest codellama:7b` |
| `ollama launch` | — | Zero-config launcher to connect AI coding tools (Claude Code, OpenCode, Cline, Codex) to Ollama. | `ollama launch claude` |
| `ollama signin` | — | Authenticates your CLI with `ollama.com` for pushing models or accessing Ollama Cloud models. | `ollama signin` |
| `ollama signout` | — | Signs out of your `ollama.com` account on the local machine. | `ollama signout` |
| `ollama help` | `ollama <cmd> -h` | Displays help documentation and flags for any CLI command. | `ollama help run` |
| `ollama --version` | `ollama -v` | Displays the installed Ollama CLI and server version numbers. | `ollama --version` |

### Common CLI Execution Patterns
```bash
# Single-shot non-interactive execution
ollama run qwen3.8:27b-mlx "Explain Gated DeltaNet attention in two sentences."

# Pipe file or log output directly into the model
cat error.log | ollama run qwen3.8:27b-mlx "Identify the root cause of these errors."

# Redirect input from a file and save output to a file
ollama run qwen3.8:27b-mlx "Summarize this document in 50 words." < input.txt > summary.txt

# Analyze a local image (native multimodal vision support)
ollama run qwen3.8:27b-mlx "Summarize the architecture diagram in ./diagram.png"

# Launch an AI coding assistant wired directly to qwen3.8:27b-mlx
ollama launch claude --model qwen3.8:27b-mlx

# Pull a GGUF model directly from Hugging Face
ollama pull hf.co/bartowski/Llama-3.2-3B-Instruct-GGUF:Q4_K_M

# Quantize an F16/F32 GGUF model during creation
ollama create my-model -f ./Modelfile --quantize q4_K_M
```

---

## 2. Inspecting `qwen3.8:27b-mlx` (Checking Defaults & Capabilities)

| What You Want to Check | Terminal Command | What It Reveals |
| :--- | :--- | :--- |
| **Capabilities & Thinking Levels** | `ollama show qwen3.8:27b-mlx` | Architecture (`27.8B` params), native context (`262144`), and supported thinking levels (`low`, `medium`, `high`, `xhigh`). |
| **Exact Default Thinking Level** | `curl -s http://localhost:11434/api/show -d '{"model":"qwen3.8:27b-mlx"}' \| grep -o '"thinking":{[^}]*}'` | Returns `"thinking":{"values":["low","medium","high","xhigh"],"default":"medium"}`. |
| **Baked-In Runtime Parameters** | `ollama show qwen3.8:27b-mlx --parameters` | Displays default `temperature`, `top_p`, `top_k`, and `stop` tokens. |
| **Prompt & Thinking Template** | `ollama show qwen3.8:27b-mlx --template` | Shows the Go template handling `<think>` blocks, tool calls, and image tokens. |
| **Full Modelfile Recipe** | `ollama show qwen3.8:27b-mlx --modelfile` | Outputs the complete build file (`FROM`, `PARAMETER`, `TEMPLATE`, `SYSTEM`, `LICENSE`). |
| **System Prompt Only** | `ollama show qwen3.8:27b-mlx --system` | Prints the default system prompt baked into the model. |
| **Model License** | `ollama show qwen3.8:27b-mlx --license` | Prints the Apache 2.0 license text included with the model. |
| **Extended Tensor Metadata** | `ollama show qwen3.8:27b-mlx --verbose` | Prints extended layer counts, quantization details, and tensor shapes. |
| **Live Memory & Active Context** | `ollama ps` | Displays live Unified Memory footprint (~16–19 GB), active `CONTEXT` size, and unload countdown. |

---

## 3. Startup CLI Flags (`ollama run`, `ollama create`, `ollama launch`)

### `ollama run` Flags
| Flag | Valid Values / Syntax | Description & Behavior |
| :--- | :--- | :--- |
| `--think` | `low`, `medium`, `high`, `xhigh`, `true`, `false` | Controls reasoning effort. Unrecognized values fall back to `medium`. `false` disables thinking. |
| `--hidethinking` | Boolean flag | Runs reasoning internally at the active `--think` level while hiding `<think>` output in the terminal. |
| `--format` | `json` | Enforces valid JSON output. |
| `--keepalive` | Duration (`10m`, `2h`, `0`, `-1`) | Controls how long the model stays in memory after exiting (`5m` default; `-1` = forever; `0` = immediate unload). |
| `--verbose` (`-v`) | Boolean flag | Prints token generation speed (`tokens/s`), prompt evaluation time, and model load duration. |
| `--nowordwrap` | Boolean flag | Disables automatic word wrapping in terminal output (useful for wide tables/code or piping). |
| `--insecure` | Boolean flag | Allows connecting to private/self-hosted registries over HTTP or unverified TLS. |
| `--truncate` | `true` / `false` | *(Embedding models)* Truncates inputs exceeding context length (default `true`). |
| `--dimensions` | Integer (e.g., `1024`) | *(Embedding models)* Truncates output embedding vectors to the specified dimension. |
| `--ask` / `--choice` / `--level` | String / Key=Value | *(Decision models)* Evaluates yes/no, categorical choices, or scoring levels from piped input. |
| `--voice` | Voice name (e.g., `Zoey`) | *(Speech models)* Selects the TTS voice for spoken audio output. |

### `ollama create`, `ollama launch`, `ollama pull` & `ollama push` Flags
| Flag | Command | Description |
| :--- | :--- | :--- |
| `-f` / `--file` | `ollama create` | Path to the `Modelfile` (defaults to `./Modelfile`). |
| `-q` / `--quantize` | `ollama create` | Quantizes FP16/FP32 weights during build (e.g., `q4_K_M`, `q8_0`). |
| `--model` | `ollama launch` | Specifies which Ollama model to bind to the launched coding tool (e.g., `qwen3.8:27b-mlx`). |
| `--config` | `ollama launch` | Writes configuration for the coding integration without immediately launching it. |
| `--insecure` | `ollama push` / `pull` | Allows pushing/pulling over HTTP or unverified TLS certificates. |

### Thinking Flag Examples for `qwen3.8:27b-mlx`
```bash
# Launch with maximum reasoning depth
ollama run qwen3.8:27b-mlx --think=xhigh

# Launch with high reasoning, but only print the final answer
ollama run qwen3.8:27b-mlx --think=high --hidethinking

# Disable thinking completely for lowest latency
ollama run qwen3.8:27b-mlx --think=false

# Keep the 18 GB model pinned in Unified Memory for 1 hour with speed stats
ollama run qwen3.8:27b-mlx --think=medium --keepalive 1h --verbose
```

---

## 4. Interactive Session Slash Commands (Inside `ollama run`)

Type these commands directly inside an active `ollama run` chat session:

### Thinking & Output Formatting
* `/set think <level>` — Switch thinking level mid-chat (`low`, `medium`, `high`, `xhigh`).
* `/set think` — Enable thinking at the model's default level (`medium`).
* `/set nothink` — Disable thinking completely for instant replies.
* `/set verbose` / `/set quiet` — Toggle token/sec performance telemetry after each response.
* `/set format json` / `/set noformat` — Toggle strict JSON output mode.
* `/set nowordwrap` / `/set wordwrap` — Toggle terminal line wrapping.

### Session Inspection & State Management
* `/set parameter <name> <value>` — Update any runtime parameter (e.g., `/set parameter num_ctx 32768`).
* `/set system <prompt>` — Override the system prompt for the current session.
* `/set history` / `/set nohistory` — Enable or disable saving prompt input history to disk (`~/.ollama/history`).
* `/show info` — Display active model metadata and context details.
* `/show parameters` — Show active runtime parameters for the current session.
* `/show system` — Print the currently active system prompt.
* `/show template` — Print the active prompt template.
* `/show modelfile` — Print the active Modelfile.
* `/show license` — Print the model's license.
* `/save <new-model-name>` — Save your current session's `system` prompt and `parameter` changes as a new local model.
* `/load <model-name>` — Switch to a different local model without exiting the CLI.
* `/clear` — Clear conversation history and reset the KV context cache.
* `"""` — Open and close multi-line input blocks.
* `/?` or `/help` — List interactive slash commands and keyboard shortcuts.
* `/bye` — Exit the interactive session (or press `Ctrl + D`).

---

## 5. All Updatable Runtime Parameters (`Modelfile` & `/set parameter`)

Apply these via `/set parameter <name> <value>` mid-chat, `PARAMETER <name> <value>` in a `Modelfile`, or in the `"options"` JSON object via the REST API:

| Parameter | Type | Ollama Default | Qwen 3.8 Recommended | Description & Effect on `qwen3.8:27b-mlx` |
| :--- | :--- | :--- | :--- | :--- |
| `num_ctx` | `int` | `4096` (or auto-scaled) | `16384` – `65536` (Max: `262144`) | Context window size in tokens. Raise explicitly for large codebases or documents. |
| `temperature` | `float` | `0.8` | `1.0` (Thinking) / `0.7` (No-think) / `0.2` (Code) | Sampling randomness. Qwen 3.8 recommends `1.0` during `--think` to prevent reasoning loops. |
| `top_p` | `float` | `0.9` | `0.95` (Thinking) / `0.80` (No-think) | Nucleus sampling cumulative probability cutoff. |
| `top_k` | `int` | `40` | `20` | Restricts token sampling to the top K candidates (`20` per Qwen 3.8 model card). |
| `min_p` | `float` | `0.0` | `0.0` – `0.05` | Minimum token probability threshold relative to the most likely token. |
| `repeat_penalty` | `float` | `1.0` | `1.0` – `1.05` | Penalizes repeated token sequences. Keep `<= 1.05` so `<think>` blocks do not degrade. |
| `repeat_last_n` | `int` | `64` | `64` – `256` | Number of past tokens scanned for repetition (`0` = off, `-1` = full `num_ctx`). |
| `num_predict` | `int` | `-1` | `-1` (unlimited) | Max tokens to generate per turn (`-1` = unlimited). Low values can cut off responses mid-`<think>`. |
| `seed` | `int` | `0` | `0` (random) or fixed int (e.g., `42`) | Deterministic random number seed for reproducible outputs. |
| `stop` | `string` | Model default | e.g., `"<\|im_end\|>"` | Custom stop sequence(s) that halt token generation immediately. |
| `draft_num_predict` | `int` | `4` | `0` – `4` | Number of speculative decoding draft tokens (`0` disables speculative drafting). |

---

## 6. Server & MLX Environment Variables

Set globally on macOS via `launchctl setenv <VAR> <VALUE>` (followed by restarting the Ollama app), in `~/.zshrc`, or in Linux `systemd` units:

| Environment Variable | Default | Example Value | What It Controls |
| :--- | :--- | :--- | :--- |
| `OLLAMA_HOST` | `127.0.0.1:11434` | `0.0.0.0:11434` | Network interface and port the Ollama server binds to. |
| `OLLAMA_MODELS` | `~/.ollama/models` | `/Volumes/ExternalSSD/ollama` | Custom directory path where downloaded model weights are stored. |
| `OLLAMA_CONTEXT_LENGTH` | Auto (`4k`/`32k`/`256k`) | `32768` | Global default `num_ctx` across all models and API clients. |
| `OLLAMA_FLASH_ATTENTION` | `0` | `1` | Enables Flash Attention to reduce Unified Memory/VRAM usage on long contexts. |
| `OLLAMA_KV_CACHE_TYPE` | `f16` | `q8_0` or `q4_0` | Quantizes the KV cache (requires Flash Attention) to fit larger context windows in 24–32 GB Macs. |
| `OLLAMA_KEEP_ALIVE` | `5m` | `30m` or `-1` | Global idle duration before unloading models from memory. |
| `OLLAMA_MAX_LOADED_MODELS` | Auto | `1` | Max models loaded simultaneously (set to `1` on 24–32 GB Macs to prevent swap thrashing). |
| `OLLAMA_NUM_PARALLEL` | Auto | `1` or `2` | Max concurrent requests per loaded model (each slot multiplies KV cache memory). |
| `OLLAMA_MAX_QUEUE` | `512` | `128` | Maximum number of queued incoming requests before returning 503 busy errors. |
| `OLLAMA_ORIGINS` | Localhost | `*` or `http://localhost:3000` | Comma-separated list of allowed CORS origins for browser clients. |
| `OLLAMA_SCHED_SPREAD` | `0` | `1` | Spreads model layers across multiple GPUs instead of packing onto one. |
| `OLLAMA_GPU_OVERHEAD` | `0` | `1073741824` | Reserves a specified number of bytes of VRAM per GPU for non-inference tasks. |
| `OLLAMA_LOAD_TIMEOUT` | `5m` | `10m` | Maximum time allowed for a model to load before aborting. |
| `OLLAMA_MAX_TRANSFER_STREAMS` | `4` | `8` | Number of parallel download/upload streams during `pull` and `push`. |
| `OLLAMA_NO_CLOUD` | `0` | `1` | Disables Ollama Cloud remote inference and web search integrations. |
| `OLLAMA_NOPRUNE` | `0` | `1` | Prevents Ollama from automatically pruning unused model blobs on startup. |
| `OLLAMA_NOHISTORY` | `0` | `1` | Disables saving interactive CLI prompt history to disk. |
| `OLLAMA_DEBUG` | `0` | `1` | Enables verbose server debug logging. |

---

## 7. Hardware & Ready-to-Use Profiles for `qwen3.8:27b-mlx`

### Apple Silicon Hardware & Architecture Profile
* **Ollama Version:** Requires `v0.32.12` or newer.
* **Architecture:** 27.8B dense parameters (16 full-attention layers + 48 Gated DeltaNet linear layers; KV cache uses ~64 KB/token).
* **Disk Size:** ~18 GB (4-bit MLX build).
* **Unified Memory:** **24 GB minimum** (~16–19 GB used by weights + overhead); **32 GB+ recommended** for 32K+ context windows.

### Custom Modelfile 1: Deep Reasoning & Coding (`qwen3.8-think`)
Save as `Modelfile.think` and build with `ollama create qwen3.8-think -f Modelfile.think`:
```dockerfile
FROM qwen3.8:27b-mlx
PARAMETER num_ctx 32768
PARAMETER temperature 1.0
PARAMETER top_p 0.95
PARAMETER top_k 20
PARAMETER min_p 0.0
PARAMETER repeat_penalty 1.0
```
Run with: `ollama run qwen3.8-think --think=high`

### Custom Modelfile 2: Fast Non-Thinking & Tool Use (`qwen3.8-fast`)
Save as `Modelfile.fast` and build with `ollama create qwen3.8-fast -f Modelfile.fast`:
```dockerfile
FROM qwen3.8:27b-mlx
PARAMETER num_ctx 32768
PARAMETER temperature 0.7
PARAMETER top_p 0.80
PARAMETER top_k 20
PARAMETER min_p 0.0
PARAMETER repeat_penalty 1.05
```
Run with: `ollama run qwen3.8-fast --think=false`

### Recommended `~/.zshrc` Aliases
```bash
alias qwen-low="ollama run qwen3.8:27b-mlx --think=low"
alias qwen-med="ollama run qwen3.8:27b-mlx --think=medium"
alias qwen-high="ollama run qwen3.8:27b-mlx --think=high"
alias qwen-xhigh="ollama run qwen3.8:27b-mlx --think=xhigh"
alias qwen-fast="ollama run qwen3.8:27b-mlx --think=false"
```
