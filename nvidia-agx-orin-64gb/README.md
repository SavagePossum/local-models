# NVIDIA Jetson AGX Orin 64GB

NVIDIA Jetson AGX Orin 64GB module, used as a dedicated model host. Memory is 64GB unified LPDDR5, shared by the CPU and the GPU. The GPU is Ampere, CUDA compute capability 8.7 (`sm_87`). The OS stack is NVIDIA JetPack.

No models, builds, server configs, or benchmark numbers have been recorded yet.

## Hardware

| | |
| --- | --- |
| Device | NVIDIA Jetson AGX Orin 64GB |
| Memory | 64GB unified LPDDR5 |
| GPU architecture | Ampere, CUDA `sm_87` |
| OS | JetPack (release not recorded yet) |
| Backend | CUDA |
| Role | Dedicated model host |

## Suggested layout

```text
nvidia-agx-orin-64gb/
  README.md                 # this file
  models/<model>-<quant>.md # one copy of TEMPLATE.md per model and quant
  llama.cpp.md              # build commit, flags, and JetPack / CUDA versions
  llama-server.service      # systemd unit, once one is in use
  benchmarks/<run>.md       # optional pasted llama-bench output
```

## Models and quants

None recorded yet. Add a file under `models/` from [`TEMPLATE.md`](../TEMPLATE.md) and link it here.

Unified memory is the fit limit: there is no separate VRAM pool. Record how much of the 64GB the model, KV cache, and the rest of the system were using.

| Model | Quant | Context | Backend | PP tok/s | TG tok/s | Memory | Date | Entry |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

## llama.cpp build

Not recorded yet.

When this is filled in, include the llama.cpp git commit, the CMake flags, `CMAKE_CUDA_ARCHITECTURES=87` (or the equivalent you passed), the JetPack release, the CUDA version, and the power mode the board was in while you built or ran.

## Server config

Not recorded yet.

When this is filled in, include the `llama-server` launch command and any systemd unit: bind address, port, context (`-c`), GPU layers, flash attention, KV-cache type, and batch sizes. Note the Jetson power mode (`nvpmodel` / `jetson_clocks`) if it was set for the run.

## Benchmarks

None recorded yet.

Record prompt-processing tok/s and generation tok/s separately, with the command, the prompt length, the generation length, the context the server loaded, and memory use (`tegrastats` or the equivalent you used). Follow the measurement fields in the [repository conventions](../README.md#conventions).

## Notes and gotchas

None recorded yet.

## Other runtimes

No vLLM or other runtime has been recorded on this board yet. Use the same sections (build, server command, benchmark table) when one is added.
