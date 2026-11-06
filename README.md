# Running Models 101

Companion repository for **Local AI Sessions Ep. 01 — Running Models 101**,
first run 2026-11-06 with [@javierbits](https://x.com/javierbits) and
[@BHIIKTOR](https://x.com/BHIIKTOR).

The goal: one runtime, one model that fits, one task. Start with the smallest
model that does the job, not the biggest one that fits.

## Start here

- [CHEATSHEET.md](CHEATSHEET.md) — concepts, model-name decoder, memory tiers,
  failure table, and the things not to say.
- [COMMANDS.md](COMMANDS.md) — copy-paste commands for Ollama, LM Studio,
  llama.cpp, OpenAI-compatible APIs, and GPU checks.

## First run

macOS or Linux:

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama run lfm3:1.7b
```

Windows: install from <https://ollama.com/download>, then:

```powershell
ollama run lfm3:1.7b
```

Ask it something, then check where it is running:

```bash
ollama ps
```

The first pull is about 1.1 GB. On a laptop with no GPU it runs on CPU, slower.
On a GPU, `ollama ps` should show 100% GPU. If it is slow, check memory before
changing flags. If the pull says it requires a newer version of Ollama, update
from <https://ollama.com/download>.

The API is OpenAI-compatible at `http://localhost:11434/v1`. See
[COMMANDS.md](COMMANDS.md) for curl, context settings, LM Studio, and
llama.cpp.

## The one rule

Pick the smallest model that does the task, not the biggest one that fits.

| Precision | Rough memory |
|---|---|
| 4-bit | ~0.5 GB per 1B parameters |
| 8-bit | ~1 GB per 1B |
| 16-bit | ~2 GB per 1B |
| 32-bit | ~4 GB per 1B |

Add 1-4 GB for context and runtime overhead. A 1.7B at Q4 is about 1.1 GB. A
27B at Q4 is about 17 GB. More context means more memory.

## What we cover

- What a model is: weights, architecture, tokenizer, chat template, runtime.
- The memory budget and model tiers.
- How to read a model filename and quant label.
- Ollama, LM Studio, llama.cpp, and vLLM.
- First run, GPU verification, context, and the local API.
- What breaks and how to diagnose it.
- Local vs cloud, licenses, privacy, and agents.

## Repository layout

```text
README.md       this file
CHEATSHEET.md   concepts, model decoder, tiers, failures
COMMANDS.md     copy-paste commands
LICENSE         MIT
```

## Notes

- Model names, tags, and sizes were checked on 2026-11-05. They go stale. Check
  the model card and the runtime's model library before relying on a name.
- Local inference is not automatically private. The app around it can still log.
- Open weights is not the same as open source. The license is separate.
- NVIDIA is the path of least resistance. AMD and Intel work, but check the
  runtime's support matrix first.
- Benchmarks are not a task. One tokens-per-second number without a workload is
  noise.

## License

MIT. See [LICENSE](LICENSE).
