# NVIDIA Jetson AGX Orin 64GB

NVIDIA Jetson AGX Orin 64GB module, used as a dedicated model host. Memory is 64GB unified LPDDR5, shared by the CPU and the GPU. The GPU is Ampere, CUDA compute capability 8.7 (`sm_87`). The board is running L4T R36.4.7. The JetPack meta-package version was not recorded.

One model is in service: [Ornith-1.5-35B-A3B Q4_K_M](models/ornith-1.5-35b-a3b-q4_k_m.md). `llama-bench` on 2026-10-26, without MTP: at KV depth 0 / 32K / 128K, prompt processing was 631 / 528 / 327 t/s and generation was 38.3 / 33.4 / 24.1 t/s. 256K depth was not measured.

## Hardware

| | |
| --- | --- |
| Device | NVIDIA Jetson AGX Orin 64GB |
| Memory | 64GB unified LPDDR5 |
| GPU architecture | Ampere, CUDA `sm_87` |
| OS | L4T R36.4.7 (JetPack package version not recorded) |
| Backend | CUDA |
| Role | Dedicated model host |

## Suggested layout

```text
nvidia-agx-orin-64gb/
  README.md                         # this file
  models/<model>-<quant>.md         # one copy of TEMPLATE.md per model and quant
  configs/llama-server.service      # unit installed on the board
  llama.cpp.md                      # optional: CMake flags, once captured
  benchmarks/<run>.md               # optional pasted llama-bench output
```

## Models and quants

Unified memory is the fit limit: there is no separate VRAM pool. Record how much of the 64GB the model, KV cache, and the rest of the system were using.

| Model | Quant | Context | Backend | PP tok/s | TG tok/s | Memory | Date | Entry |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Ornith-1.5-35B-A3B | Q4_K_M | 262144 | CUDA | 631 / 528 / 327 | 38.3 / 33.4 / 24.1 | ~37 GB of 61 GB (`free -g`) | 2026-10-26 | [entry](models/ornith-1.5-35b-a3b-q4_k_m.md) |

PP and TG are `pp512` and `tg128` at KV depth 0 / 32K / 128K (`llama-bench -d 0,32768,131072`, server stopped, `nvpmodel -m 0` and `jetson_clocks`). This bench does not include `--spec-type draft-mtp`, so served generation speed may differ. 256K depth was not measured. The memory column is `free -g` while the server was up, not a bench reading.

## llama.cpp build

Recorded for the Ornith server, from the checkout on the board. CMake flags and the CUDA toolkit version were not captured.

- **Commit:** [`704485942ab54bbbbf1f241b3550ffba35f5f37e`](https://github.com/ggml-org/llama.cpp/commit/704485942ab54bbbbf1f241b3550ffba35f5f37e) (2026-08-11 11:47:57 +0200)
- **Binary:** `/home/nvidia/src/llama.cpp/build/bin/llama-server`
- **GPU arch:** `sm_87`
- **OS:** L4T R36.4.7 (`/etc/nv_tegra_release`: R36 revision 4.7, GCID 42132812, 2025-09-18, `KERNEL_VARIANT: oot`)
- **Power mode while serving:** `nvpmodel -m 0`, then `jetson_clocks`

## Server config

[`configs/llama-server.service`](configs/llama-server.service) is the unit installed at `/etc/systemd/system/llama-server.service`. It serves Ornith-1.5-35B-A3B Q4_K_M on `0.0.0.0:8080` with context 262144, f16 KV cache, flash attention, and MTP speculative decoding (`--spec-draft-n-max 2`). Flags, the memory reading, and the 29 GB comment mismatch are written up in the [model entry](models/ornith-1.5-35b-a3b-q4_k_m.md).

## Benchmarks

`llama-bench` on 2026-10-26, server stopped, after `nvpmodel -m 0` and `jetson_clocks`. Depth 0 / 32K / 128K: prompt processing 631 / 528 / 327 t/s, generation 38.3 / 33.4 / 24.1 t/s (`pp512` and `tg128`). `llama-bench` excludes MTP speculative decoding, so served generation with `--spec-type draft-mtp` may differ. 256K depth was not measured. The command and the raw table are in the [model entry](models/ornith-1.5-35b-a3b-q4_k_m.md).

While the Ornith server was up, `free -g` reported about 37 GB used of 61 GB total, with swap unused.

## Notes and gotchas

- The systemd comment says "29 GB model". The Q4_K_M file in service is 21.86 GB. The comment likely dates from an earlier Q6 model. See the [model entry](models/ornith-1.5-35b-a3b-q4_k_m.md).
- `jetson_clocks` does not persist across reboot. The unit re-applies `nvpmodel -m 0` and `jetson_clocks` as root before each start.
- First start uses `TimeoutStartSec=900`.

## Other runtimes

No vLLM or other runtime has been recorded on this board yet. Use the same sections (build, server command, benchmark table) when one is added.
