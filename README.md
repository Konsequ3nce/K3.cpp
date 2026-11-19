# K3 — CPU-first inference engine

**Your CPU is faster than you think. Run big models without a big GPU.**

K3 (`k3.cpp`) is a fork of [ik_llama.cpp](https://github.com/ikawrakow/ik_llama.cpp) for people who run LLMs on the CPU: laptops, desktops, workstations and old servers. It works on any modern x86 CPU and gets the most from multi-socket NUMA machines, where it keeps every socket working from its own local memory. A GPU is optional and only takes on the small parts where it clearly helps.

> 🚧 **Status: early preview.** Code, builds and full docs are on the way. This page shows where it is going.

## Why

Most local-inference tools treat the CPU as a fallback for when the model doesn't fit on the GPU. But system RAM is cheap and plentiful, and a CPU with the right engine runs models at speeds people don't expect. K3 is built around that: CPU first, tuned end to end.

## Faster on every CPU

- **Adaptive suffix speculation:** drafts text that is already in the context, such as code you're editing or a file you pasted, then checks it in one pass. It needs no draft model or extra memory, and it sizes each draft by how well recent ones were accepted.
- **Prompt-cache-friendly chat templates:** follow-up turns prefill only your new message instead of the whole conversation. On Gemma 4 that is 322 → 25 tokens.
- **Tuned build:** clang + OpenMP, 4–7% faster than a stock MSVC build on Windows, with threads that stay hot between tokens.
- **Repacked weights:** weights are rearranged at load time for faster CPU matrix maths.

## Even faster on NUMA machines

- **NUMA mirror, including on Windows:** each socket gets its own copy of the weights and reads only its local RAM, with no cross-socket traffic.
- **NUMA-aware MoE experts:** expert matmuls use the local copy on every socket, worth +11% to +22% on MoE models.

## Early numbers

These are decode speeds on coding tasks, CPU only, no GPU, on a 2016 dual-socket server (2 × Xeon E5-2697 v4, DDR3-1866). The "Without K3 features" column is the same build with speculation turned off.

| Model | Without K3 features | **K3** |
|---|---|---|
| Gemma 4 26B-A4B (MoE) | ~31.7 tok/s | **53.6 tok/s** |
| Flint (MoE, 256 experts) | ~30.7 tok/s | **39.5 tok/s** |
| Spark-X2.6 1.7B (dense) | ~70 tok/s | **88–98 tok/s** |

Edit-style tasks gain the most: one bug-fix task ran at 126.7 tok/s. Numbers for single-socket desktops and laptops are coming.

## On the roadmap

- Benchmarks and tuned presets for common desktop and laptop CPUs.
- One-command setup that detects your cores, sockets and memory and picks the settings.
- Light GPU assist: put only the hottest experts and the draft head on a small GPU and keep everything else in RAM.
- Separate MTP draft heads, loaded next to unmodified model files.
- Very large MoE models (200B+) on high-memory machines.

## Credits

K3 is based on [ik_llama.cpp](https://github.com/ikawrakow/ik_llama.cpp) by ikawrakow, itself a fork of [llama.cpp](https://github.com/ggml-org/llama.cpp) by Georgi Gerganov and contributors. Both are MIT-licensed, and so is K3.

---

*Codename: Konsequ3nce.*
