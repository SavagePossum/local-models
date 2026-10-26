# AMD Strix Halo (Ryzen AI Max+ 392)

Lenovo Legion 7a laptop with an AMD Ryzen AI Max+ 392 (Strix Halo) and a Radeon 8060S iGPU. The GPU target is `gfx1151`. Memory is 64GB unified LPDDR5X, shared by the CPU and the iGPU. The OS is Omarchy Linux (Arch-based). Setups on this page target Vulkan or ROCm.

No models, builds, server configs, or benchmark numbers have been recorded yet.

## Hardware

| | |
| --- | --- |
| Machine | Lenovo Legion 7a |
| APU | AMD Ryzen AI Max+ 392 (Strix Halo) |
| iGPU | AMD Radeon 8060S |
| ROCm / LLVM target | `gfx1151` |
| Memory | 64GB unified LPDDR5X |
| OS | Omarchy Linux |
| Backends | Vulkan, ROCm |
| Role | Laptop |

## Suggested layout

```text
amd-strix-halo-392/
  README.md                 # this file
  models/<model>-<quant>.md # one copy of TEMPLATE.md per model and quant
  llama.cpp.md              # build commit, flags, and Mesa or ROCm versions
  llama-server.service      # systemd unit, once one is in use
  benchmarks/<run>.md       # optional pasted llama-bench output
```

## Models and quants

None recorded yet. Add a file under `models/` from [`TEMPLATE.md`](../TEMPLATE.md) and link it here.

Unified memory is the fit limit. Record which backend (Vulkan or ROCm) produced each row; the two are not interchangeable.

| Model | Quant | Context | Backend | PP tok/s | TG tok/s | Memory | Date | Entry |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

## llama.cpp build

Not recorded yet.

When this is filled in, include the llama.cpp git commit, the CMake flags, the GPU target (`gfx1151`), the Mesa or ROCm version, the compiler, and the kernel. Record any environment variable the build or the run required.

## Server config

Not recorded yet.

When this is filled in, include the `llama-server` launch command and any systemd unit: bind address, port, context (`-c`), GPU layers, flash attention, KV-cache type, and batch sizes. State whether the process used Vulkan or ROCm.

## Benchmarks

None recorded yet.

Record prompt-processing tok/s and generation tok/s separately, with the command, the prompt length, the generation length, the context the server loaded, and memory use. Follow the measurement fields in the [repository conventions](../README.md#conventions).

## Notes and gotchas

None recorded yet.

## Other runtimes

No vLLM or other runtime has been recorded on this laptop yet. Use the same sections (build, server command, benchmark table) when one is added.
