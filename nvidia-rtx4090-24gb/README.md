# NVIDIA GeForce RTX 4090 24GB

NVIDIA GeForce RTX 4090 with 24GB GDDR6X. The GPU is Ada Lovelace, CUDA compute capability 8.9 (`sm_89`). The host machine, operating system, driver, and system RAM have not been recorded yet.

No models, builds, server configs, or benchmark numbers have been recorded yet.

## Hardware

| | |
| --- | --- |
| Device | Desktop GPU |
| GPU | NVIDIA GeForce RTX 4090 (Ada Lovelace) |
| VRAM | 24GB GDDR6X |
| CUDA architecture | `sm_89` |
| System RAM | not recorded yet |
| OS | not recorded yet |
| Driver / CUDA | not recorded yet |
| Backend | CUDA |
| Role | not recorded yet |

## Suggested layout

```text
nvidia-rtx4090-24gb/
  README.md                 # this file
  models/<model>-<quant>.md # one copy of TEMPLATE.md per model and quant
  llama.cpp.md              # build commit, flags, and CUDA version
  llama-server.service      # systemd unit, once one is in use
  benchmarks/<run>.md       # optional pasted llama-bench output
```

## Models and quants

None recorded yet. Add a file under `models/` from [`TEMPLATE.md`](../TEMPLATE.md) and link it here.

| Model | Quant | Context | Backend | PP tok/s | TG tok/s | Memory | Date | Entry |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

## llama.cpp build

Not recorded yet.

When this is filled in, include the llama.cpp git commit, the CMake flags, `CMAKE_CUDA_ARCHITECTURES=89` (or the equivalent you passed), the CUDA toolkit and driver versions, the compiler, the OS, and the system RAM.

## Server config

Not recorded yet.

When this is filled in, include the `llama-server` launch command and any systemd unit: bind address, port, context (`-c`), GPU layers, flash attention, KV-cache type, and batch sizes.

## Benchmarks

None recorded yet.

Record prompt-processing tok/s and generation tok/s separately, with the command, the prompt length, the generation length, the context the server loaded, and memory use (`nvidia-smi` or the equivalent you used). Follow the measurement fields in the [repository conventions](../README.md#conventions).

## Notes and gotchas

None recorded yet.

## Other runtimes

No vLLM or other runtime has been recorded on this GPU yet. Use the same sections (build, server command, benchmark table) when one is added.
