# Commands

Copy-paste starting points for Local AI Sessions Ep. 01. Commands target
current Ollama and llama.cpp builds. Model tags may require a recent Ollama;
update if a pull asks for a newer version. Verify tags with `ollama list` and
the model card before relying on a name.

## Ollama

### Install

macOS or Linux:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Windows: download from <https://ollama.com/download>.

If the CLI cannot connect, start the service in a separate terminal:

```bash
ollama serve
```

On Linux the installer usually starts a systemd service. Check it with
`systemctl status ollama`.

### Pull and run

```bash
# first run, about 1.1 GB
ollama run lfm3:1.7b

# next step if the machine has the memory
ollama run qwen3.5:4b

# vision, for an 8 GB machine
ollama pull gemma4:e4b
```

`ollama run` pulls the model if it is missing. Type a prompt inside the REPL;
`/bye` exits.

### Inspect and clean

```bash
ollama list
ollama ps
ollama stop lfm3:1.7b
ollama rm lfm3:1.7b
```

`ollama ps` shows the loaded model and whether it is on GPU or CPU. `ollama
stop` unloads it; `ollama rm` deletes the weights.

### Set context explicitly

Start at 8192. Higher context uses more memory.

```bash
cat > Modelfile <<'EOF'
FROM lfm3:1.7b
PARAMETER num_ctx 8192
EOF

ollama create lfm3-8k -f Modelfile
ollama run lfm3-8k
```

### OpenAI-compatible API

```bash
curl -s http://localhost:11434/v1/models
```

```bash
curl -s http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "lfm3:1.7b",
    "messages": [{"role": "user", "content": "hello"}],
    "stream": false
  }'
```

Endpoint: `http://localhost:11434/v1`. Keep it bound to localhost. If you expose
it to the LAN, put authentication in front.

### Logs and ports

```bash
# Linux service logs
journalctl -u ollama -f

# macOS logs
tail -f ~/.ollama/logs/server.log

# who owns the port
lsof -i :11434
```

## LM Studio

1. Download from <https://lmstudio.ai>.
2. Search for `LFM3 1.7B GGUF` or open the Hugging Face repo
   `LiquidAI/LFM3-1.7B-GGUF`.
3. Pick `Q4_K_M`, download, and load it.
4. Set context to 8192 and GPU offload as high as the machine allows.
5. Start the local server on port 1234.
6. Endpoint: `http://localhost:1234/v1`.

LM Studio manages its own model folder. Do not point Ollama or llama.cpp at it.

## llama.cpp

### Build

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build --config Release -j
```

GPU build flags:

```bash
# NVIDIA
cmake -B build -DGGML_CUDA=ON

# macOS Metal (usually enabled by default)
cmake -B build -DGGML_METAL=ON

# AMD ROCm
cmake -B build -DGGML_HIP=ON

# Vulkan fallback
cmake -B build -DGGML_VULKAN=ON
```

### Serve a GGUF

```bash
./build/bin/llama-server \
  -hf LiquidAI/LFM3-1.7B-GGUF:Q4_K_M \
  --ctx-size 8192 \
  --port 8080
```

On Windows the binary is usually
`build/bin/Release/llama-server.exe`. Newer unified builds may expose `llama
serve` instead of `llama-server`. Endpoint: `http://localhost:8080/v1`.

### Chat in the terminal

```bash
./build/bin/llama-cli \
  -hf LiquidAI/LFM3-1.7B-GGUF:Q4_K_M \
  -c 8192
```

## vLLM

vLLM loads the original safetensors repo, not GGUF. It is for serving
throughput, not a laptop first run.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install vllm

vllm serve LiquidAI/LFM3-1.7B --max-model-len 8192
```

Endpoint: `http://localhost:8000/v1`.

## GPU checks

```bash
# what the runtime actually loaded
ollama ps

# NVIDIA
nvidia-smi
nvidia-smi --query-gpu=name,memory.total,memory.used --format=csv

# AMD, if installed
rocm-smi

# Linux hardware
lspci | grep -Ei 'vga|3d|display'
```

macOS:

```bash
sudo powermetrics --samplers gpu_power -n 1
system_profiler SPDisplaysDataType
```

Activity Monitor > Window > GPU History is the quick visual check.

## Gated models

Accept the license on Hugging Face first, then:

```bash
pip install -U huggingface_hub
hf auth login
```

Or use a known quant mirror. Check who quantized it and read the model card.

## Troubleshooting commands

```bash
# port conflicts
lsof -i :11434
lsof -i :1234
lsof -i :8080

# disk space and model sizes
df -h
ollama list

# runtime logs
journalctl -u ollama -f          # Linux
tail -f ~/.ollama/logs/server.log # macOS
```

Symptom to fix:

| Symptom | First fix |
|---|---|
| OOM at load | Smaller model or quant, lower context, close other GPU apps |
| Loads but CPU-only | Wrong backend or driver, or offload set to CPU; check `ollama ps` and logs |
| Slow after it works | Spilling to system RAM, context too high, thermals, power mode |
| Context error | Lower context first; the KV cache is the usual culprit |
| Port conflict | Change the port on one server |
| Disk full | Check free space; `ollama list`, `ollama rm` |
| App cannot see the model | Each runtime manages its own store; don't mix folders |
| Gated model | Accept the license, `hf auth login`, or use a quant mirror |
| Pull says "requires a newer version of Ollama" | Update Ollama from <https://ollama.com/download> |
| Dumb answers | Wrong chat template, too-low quant, too-small model, context too short |
| AMD/Intel problems | Check ROCm/Vulkan/SYCL support; llama.cpp or LM Studio may work where Ollama does not |
| Mac memory pressure | Lower context, close apps, watch unified memory |

NVIDIA is the path of least resistance. AMD and Intel work, but check the
runtime's support matrix before buying hardware for a specific model.
