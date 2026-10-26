# AMD Radeon RX 7900 XTX 24GB

Fedora desktop with an AMD Radeon RX 7900 XTX (24GB GDDR6) and 64GB of system RAM. Setups on this page target ROCm/HIP or Vulkan. The RDNA 3 ROCm target for this card is `gfx1100`.

One bench comparison is recorded: [Qwen3.8-27B Q4_K_M](models/qwen3.8-27b-q4_k_m.md), two ROCm builds of llama.cpp measured on 2026-10-26. No server config yet.

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
  llama-server.service      # systemd unit, once one is in use
  benchmarks/<run>.md       # optional pasted llama-bench output
```

## Models and quants

| Model | Quant | Context | Backend | PP tok/s | TG tok/s | Memory | Date | Entry |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Qwen3.8-27B | Q4_K_M | bench only | ROCm `df03399b8` (2026-09-10) | 1012.58 / 596.43 | 34.80 / 31.84 | 15.92 GiB model; 24560 MiB VRAM | 2026-10-26 | [entry](models/qwen3.8-27b-q4_k_m.md) |
| Qwen3.8-27B | Q4_K_M | bench only | ROCm `8b28c2394` (2026-10-26) | 1081.42 / 752.52 | 35.61 / 32.50 | 15.92 GiB model; 24560 MiB VRAM | 2026-10-26 | [entry](models/qwen3.8-27b-q4_k_m.md) |

PP is `pp512` and TG is `tg128` at KV depth 0 / 32K. 128K and 256K depths were not measured. `llama-bench` excludes MTP drafting. There is no server context yet. The memory column is the model size and the device VRAM `llama-bench` printed, not a resident-memory reading.

## llama.cpp build

Two ROCm builds, compared in the [Qwen3.8-27B entry](models/qwen3.8-27b-q4_k_m.md).

- **Old:** [`df03399b8`](https://github.com/ggml-org/llama.cpp/commit/df03399b885831b2a1603b3abb0d8c156808e363) (2026-09-10 11:25:40 -0700). CMake flags unknown.
- **New:** [`8b28c2394`](https://github.com/ggml-org/llama.cpp/commit/8b28c2394324465bf7727089e7eaf7aec9442d6a) (build 11898, 2026-10-26 11:38:55 +0100). Intended config:

```
cmake -B build -DGGML_HIP=ON -DAMDGPU_TARGETS=gfx1100 -DGGML_HIP_ROCWMMA_FATTN=ON -DCMAKE_BUILD_TYPE=Release
```

The device line was `AMD Radeon RX 7900 XTX, gfx1100, VMM: no, Wave Size: 32, VRAM: 24560 MiB`. The ROCm version, Fedora release, and kernel were not recorded.

## Server config

TODO. No `llama-server` command or systemd unit has been recorded on this machine.

## Benchmarks

`llama-bench` on 2026-10-26, both builds, same command. Depth 0 / 32K: the newer build is +6.8% / +26.2% on prompt processing and +2.3% / +2.1% on generation. 128K and 256K depths were not measured. `llama-bench` excludes MTP drafting. The command, both result sets, and the percentage arithmetic are in the [model entry](models/qwen3.8-27b-q4_k_m.md).

## Notes and gotchas

- The new build's CMake line is the intended config. The old build's flags are unknown, so the comparison is between two checkouts, not one flipped option.
- Hugging Face source for `Qwen3.8-27B-Q4_K_M.gguf` is TODO.

## Other runtimes

No vLLM or other runtime has been recorded on this machine yet. Use the same sections (build, server command, benchmark table) when one is added.
