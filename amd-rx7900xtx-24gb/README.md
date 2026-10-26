# AMD Radeon RX 7900 XTX 24GB

Fedora desktop with an AMD Radeon RX 7900 XTX (24GB GDDR6) and 64GB of system RAM. Setups on this page target ROCm/HIP or Vulkan. The RDNA 3 ROCm target for this card is `gfx1100`.

No models, builds, server configs, or benchmark numbers have been recorded yet.

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

None recorded yet. Add a file under `models/` from [`TEMPLATE.md`](../TEMPLATE.md) and link it here.

| Model | Quant | Context | Backend | PP tok/s | TG tok/s | Memory | Date | Entry |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

## llama.cpp build

Not recorded yet.

When this is filled in, include the llama.cpp git commit, the CMake flags, the GPU target (`gfx1100`), the ROCm or Vulkan (Mesa) version, the compiler, and the Fedora release and kernel.

## Server config

Not recorded yet.

When this is filled in, include the `llama-server` launch command and any systemd unit: bind address, port, context (`-c`), GPU layers, flash attention, KV-cache type, and batch sizes.

## Benchmarks

None recorded yet.

Record prompt-processing tok/s and generation tok/s separately, with the command, the prompt length, the generation length, the context the server loaded, and memory use (`rocm-smi` or the equivalent you used). Follow the measurement fields in the [repository conventions](../README.md#conventions).

## Notes and gotchas

None recorded yet.

## Other runtimes

No vLLM or other runtime has been recorded on this machine yet. Use the same sections (build, server command, benchmark table) when one is added.
