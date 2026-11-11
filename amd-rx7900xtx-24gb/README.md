# AMD Radeon RX 7900 XTX 24GB

Fedora desktop with an AMD Radeon RX 7900 XTX (24GB GDDR6) and 64GB of system RAM. Setups on this page target ROCm/HIP or Vulkan. The RDNA 3 ROCm target for this card is `gfx1100`.

Default server (2026-11-11): [Qwen3.8-Flash-Next UD-IQ2_XS](models/qwen3.8-flash-next-ud-iq2_xs.md) at context 131072 with MoE/PLE offload (~50 PP / ~19–20 TG tok/s). Earlier bench-only entry: [Qwen3.8-27B Q4_K_M](models/qwen3.8-27b-q4_k_m.md).

## Hardware

| | |
| --- | --- |
| Device | Desktop GPU |
| GPU | AMD Radeon RX 7900 XTX (RDNA 3) |
| VRAM | 24GB GDDR6 |
| ROCm / LLVM target | `gfx1100` |
| System RAM | 64GB |
| OS | Fedora (release not recorded yet) |
| Backends | ROCm/HIP, Vulkan |
| Role | Desktop that also runs local models |

## Suggested layout

```text
amd-rx7900xtx-24gb/
  README.md                 # this file
  models/<model>-<quant>.md # one copy of TEMPLATE.md per model and quant
  llama.cpp.md              # build commit, flags, and toolkit versions
  llama-server.service      # systemd unit in use
  benchmarks/<run>.md       # optional pasted llama-bench output
```

## Models and quants

| Model | Quant | Context | Backend | PP tok/s | TG tok/s | Memory | Date | Entry |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Qwen3.8-Flash-Next | UD-IQ2_XS | 131072 | ROCm (2026-11-11 build) | ~50 | ~19–20 | partial offload; PLE mmap + `--n-cpu-moe 30` | 2026-11-11 | [entry](models/qwen3.8-flash-next-ud-iq2_xs.md) |
| Qwen3.8-27B | Q4_K_M | bench only | ROCm `df03399b8` (2026-09-10) | 1012.58 / 596.43 | 34.80 / 31.84 | 15.92 GiB model; 24560 MiB VRAM | 2026-10-26 | [entry](models/qwen3.8-27b-q4_k_m.md) |
| Qwen3.8-27B | Q4_K_M | bench only | ROCm `8b28c2394` (2026-10-26) | 1081.42 / 752.52 | 35.61 / 32.50 | 15.92 GiB model; 24560 MiB VRAM | 2026-10-26 | [entry](models/qwen3.8-27b-q4_k_m.md) |

Flash-Next PP/TG are `llama-server` slot timings on short interactive prompts (not `llama-bench`). 27B PP is `pp512` and TG is `tg128` at KV depth 0 / 32K. `llama-bench` excludes MTP drafting.

## llama.cpp build

Flash-Next production binary: rebuilt 2026-11-11 from a current `~/llama.cpp` ROCm tree, installed to `/opt/llama.cpp/bin/llama-server`. Full git SHA not recorded.

```
cmake -B build -DGGML_HIP=ON -DAMDGPU_TARGETS=gfx1100 \
  -DGGML_HIP_ROCWMMA_FATTN=ON -DCMAKE_BUILD_TYPE=Release
```

Earlier 27B comparison used two builds; details in the [Qwen3.8-27B entry](models/qwen3.8-27b-q4_k_m.md):

- **Old:** [`df03399b8`](https://github.com/ggml-org/llama.cpp/commit/df03399b885831b2a1603b3abb0d8c156808e363) (2026-09-10).
- **New:** [`8b28c2394`](https://github.com/ggml-org/llama.cpp/commit/8b28c2394324465bf7727089e7eaf7aec9442d6a) (build 11898, 2026-10-26).

The device line was `AMD Radeon RX 7900 XTX, gfx1100, VMM: no, Wave Size: 32, VRAM: 24560 MiB`. The ROCm version, Fedora release, and kernel were not recorded.

## Server config

Production unit: [`llama-server.service`](llama-server.service). Serves Flash-Next UD-IQ2_XS on `0.0.0.0:8080` with context 131072, `--n-cpu-moe 30`, PLE on CPU via `--override-tensor`, mmap load, `--parallel 1`, `--threads-http 8`, `--no-reasoning-preserve`.

```
sudo systemctl daemon-reload
sudo systemctl enable --now llama-server.service
```

## Benchmarks

- **Flash-Next (2026-11-11):** interactive server timings ~50 PP / ~19–20 TG tok/s at 65K–128K. Details in the [model entry](models/qwen3.8-flash-next-ud-iq2_xs.md).
- **27B (2026-10-26):** `llama-bench` on two ROCm builds. Depth 0 / 32K: newer build +6.8% / +26.2% PP and +2.3% / +2.1% TG. Details in the [27B entry](models/qwen3.8-27b-q4_k_m.md).

## Notes and gotchas

- Do not pass `-ngl 99` with this Flash-Next offload recipe; it aborts `--fit` and can OOM (~43 GB alloc on a 24 GB card).
- Stable context is 131072. 142000 loaded but ABRTed under the built-in WebUI.
- Built-in WebUI may drop SSE on this slow path; `/v1` via curl or an external client is reliable.
- Unsloth shared MTP needs a special llama.cpp build; not enabled here.
- Hugging Face source for the older `Qwen3.8-27B-Q4_K_M.gguf` is still TODO.

## Other runtimes

No vLLM or other runtime has been recorded on this machine yet. Use the same sections (build, server command, benchmark table) when one is added.
