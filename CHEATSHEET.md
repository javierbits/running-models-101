# Running Models 101 — cheatsheet

One-page reference for Local AI Sessions Ep. 01. Model names, tags, and sizes
were checked on 2026-11-05. They go stale; check the model card and the
runtime's library.

## The stack

| Term | Plain meaning |
|---|---|
| Model | Weights + architecture + tokenizer/config |
| Weights | Learned numbers; a statistical memory, not a database |
| Architecture | Blueprint: layers, attention, routing, context limit |
| Tokenizer | Text ↔ token IDs; part of the model, not interchangeable |
| Chat template | Formatting for system/user/assistant turns |
| Runtime | Engine that loads weights and serves tokens |
| GGUF | Model container for llama.cpp-family runtimes |
| Safetensors | Raw tensor container for Transformers/vLLM |
| Context | Working memory for the conversation; measured in tokens |
| KV cache | Cached attention state; grows with context and batch |
| Quantization | Lower-precision weights; smaller and faster, but lossy |
| Offload | Which layers run on GPU versus CPU |
| Agent | Model + loop + tools + permissions |

A GGUF is not an app. Double-clicking it does nothing without a runtime. A
safetensors file is not a runnable model either; it needs the architecture
config, tokenizer, and a serving stack.

## Memory math

| Precision | Rough memory | Note |
|---|---|---|
| 4-bit | ~0.5 GB per 1B parameters | Safe default for local; real files vary |
| 8-bit | ~1 GB per 1B | More quality, more memory |
| 16-bit | ~2 GB per 1B | Original-ish precision; large |
| 32-bit | ~4 GB per 1B | Rare for inference |

Add 1-4 GB for context, KV cache, and runtime overhead. More context means more
memory. A model that fits at load can still fail at 32K context.

Examples:

| Model | Precision | Approx. size |
|---|---|---|
| LFM3-1.7B | Q4 | ~1.1 GB |
| Qwen3.5-4B | Q4 | ~3 GB |
| 8B dense | Q4 | ~4.7-5.5 GB |
| 8B dense | BF16 | ~16 GB |
| Qwen3.8-27B | Q4 | ~17 GB |
| Qwen3.8-27B | BF16 | ~54 GB |

## Model tiers

| Memory | Pick | Size | Use |
|---|---|---|---|
| Any / 2 GB | LFM3-1.7B Q4 | ~1.1 GB | First run; text-only; laptop or phone |
| 8 GB VRAM | Qwen3.5-4B Q4 | ~3 GB | Next step; fits with context |
| 8 GB VRAM, vision | Gemma 4 E4B Q4 | ~5 GB | Images and documents |
| 12 GB VRAM | Gemma 4 12B Q4 | ~7.5 GB | Small multimodal |
| 24 GB VRAM | Qwen3.8-27B Q4 | ~17 GB | Best all-round local model |
| 24 GB VRAM, fast | Gemma 4 26B-A4B Q4 | ~16 GB | MoE, multimodal |
| 32 GB VRAM | Qwen3.6-35B-A3B Q4 | ~22 GB | MoE, fast, more total knowledge |
| 64 GB+ unified | 27B/31B Q8, 35B-A3B Q8 | ~29-37 GB | Bigger quant, slower |

A bigger quant of a smaller model usually beats a tiny quant of a bigger model.
Pick the smallest model that does the task.

## Filename decoder

Example: `Qwen3.8-27B-Q4_K_M.gguf`

| Token | Meaning |
|---|---|
| `Qwen3.8-27B` | Family, version, parameter count |
| `A3B` | MoE with 3B active; total model memory is bigger |
| `E4B` | Effective 4B; not the same as a dense 4B |
| `it` | Instruction tuned; `base` is raw |
| `Q4_K_M` | 4-bit k-quant, medium; safe default |
| `Q5_K_M`, `Q6_K`, `Q8_0` | More bits, more memory, less loss |
| `IQ4_XS` | i-quant; similar size, more compute |
| `UD-` | Unsloth Dynamic; selective precision |
| `QAT` | Quantization-aware training; better at same size |
| `GGUF` | Container for Ollama, LM Studio, llama.cpp |
| `safetensors` | Original weights for Transformers/vLLM |
| `AWQ`, `GPTQ`, `FP8`, `NVFP4`, `EXL3`, `MLX` | Other formats, often hardware-specific |
| `MTP`, `DFlash`, `EAGLE` | Speculative decoding; runtime support required |

More examples:

- `Qwen3.6-35B-A3B-Q4_K_M.gguf` — 35B total, 3B active MoE. Fast compute,
  full-model memory.
- `gemma-4-E4B-it-Q4_K_M.gguf` — Gemma 4, effective 4B, instruction-tuned.
- `Llama-3.1-8B-Instruct-Q8_0.gguf` — 8B dense, 8-bit.
- `model-00001-of-00004.safetensors` — sharded original weights, not a GGUF.

## Runtime decision tree

| If you want... | Use | Caveat |
|---|---|---|
| Fastest first run, terminal, API | Ollama | Curated tags, defaults, less control |
| GUI, model browser, chat, server | LM Studio | Closed source, GUI-first |
| A desktop app with curated models | llama.app | New, less control |
| Maximum control, custom quants, unusual hardware | llama.cpp | Flags, setup, second step |
| Serving many users, throughput | vLLM | Not a laptop first run |

Install one. If you do not know which, Ollama. If you hate terminals, LM Studio
or llama.app. Do not install all four on day one.

## What breaks

| Symptom | First fix |
|---|---|
| OOM at load | Smaller model or quant, lower context, close other GPU apps |
| Loads but CPU-only | Wrong backend or driver, or offload set to CPU; check `ollama ps` and logs |
| Slow after it works | Spilling to system RAM, context too high, thermals, power mode |
| Context error | Lower context first; the KV cache is the usual culprit |
| Port conflict | Change the port on one server |
| Disk full | Check free space; `ollama list`, `ollama rm` |
| App cannot see the model | Each runtime manages its own store; don't mix folders |
| Gated model | Accept the license on Hugging Face, `hf auth login`, or use a quant mirror |
| Pull says "requires a newer version of Ollama" | Update Ollama from <https://ollama.com/download> |
| Dumb answers | Wrong chat template, too-low quant, too-small model, context too short, reasoning off |
| AMD/Intel problems | Check ROCm/Vulkan/SYCL support; llama.cpp or LM Studio may work where Ollama does not |
| Mac memory pressure | Lower context, close apps, watch unified memory |

NVIDIA is the path of least resistance. AMD and Intel work, but check the
support matrix before buying hardware for a specific model.

## What not to say

- "Local is always private." The inference is local; the app can still log.
- "Open weights means open source." It does not; the license is separate.
- "This model is better." Better at what, on which memory, at which quant?
- "You need a 4090." You need enough memory for the model you picked.
- "Quantization is free." It is a quality trade-off.
- "Just max the context." The KV cache will eat your memory.
- "Cloud is bad." It is a different tool, not a moral failing.
- "It's easy." It is 30 minutes of work the first time.
- One benchmark number without a task is noise.

## Licenses

- Apache-2.0 and MIT are easy.
- Gemma and Llama use community licenses with conditions.
- LFM3 uses Liquid's LFM1.0 license: free under $10M annual revenue, check above
  that.
- "Open weights" is not the same as OSI open source. Read the license before
  commercial use.

## Acronyms

| Term | Meaning |
|---|---|
| MoE | Mixture of experts; total parameters vs active parameters |
| RAM | System memory |
| VRAM | GPU memory |
| KV | Key/value attention cache |
| OOM | Out of memory |
| tok/s | Tokens per second |
| QAT | Quantization-aware training |
| AWQ/GPTQ | Post-training quantization formats for GPU serving |
| FP8/NVFP4 | Low-precision GPU formats |
| EXL3 | ExLlamaV3 quantization format |
| MLX | Apple Silicon model format/runtime |
| MTP/DFlash/EAGLE | Speculative decoding draft heads |
| HF | Hugging Face |

## Security

- Local inference stays on your machine. The app around it can still log.
- Bind the API to localhost. Do not expose the port to the LAN or internet
  without authentication.
- Agent tool output is untrusted input. A web page or file can contain prompt
  injection.
- Give agents least privilege: read-only by default, explicit approval for
  writes and destructive commands, and a sandbox for shell access.
- Cap the loop. Agents can burn tokens and time repeating the same mistake.
