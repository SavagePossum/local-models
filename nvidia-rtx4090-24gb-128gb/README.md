# NVIDIA GeForce RTX 4090 24GB, 128GB system RAM

Ubuntu desktop `AMD9900-ML-ubuntu` with an NVIDIA GeForce RTX 4090 (24GB GDDR6X, Ada Lovelace, `sm_89`), an AMD Ryzen 9 9950X (16 cores / 32 threads), and 128GB of system RAM (~123 GiB usable). The machine also runs Hermes Agent and a desktop session, so the model is budgeted at ~90 GB with ~30 GB of headroom.

The 64GB Ubuntu RTX 4090 host is a separate page, [`nvidia-rtx4090-24gb`](../nvidia-rtx4090-24gb/README.md). This page is the 128GB desktop. Speeds and flags here were measured on this machine. The UD-IQ4_XS recipe keeps ~62 GB of experts in RAM, which is why the two hosts are separate folders.

One model is in service: [Qwen3.8-Flash-Next UD-IQ4_XS](models/qwen3.8-flash-next-ud-iq4_xs.md). llama.cpp tag `b12557`, CUDA, no MTP, context 262144. Generation once the service is warm (GPU MoE cache and page cache) is ~40 t/s at 256K. A cold single request right after startup, at 32K with `--moe-cache-mib 9000`, logged 29.50 t/s average (31.3 t/s steady).

## Hardware

| | |
| --- | --- |
| Machine | `AMD9900-ML-ubuntu` |
| Device | Desktop GPU |
| GPU | NVIDIA GeForce RTX 4090 (Ada Lovelace) |
| VRAM | 24GB GDDR6X |
| CUDA architecture | `sm_89` |
| CPU | AMD Ryzen 9 9950X (16c/32t) |
| System RAM | 128GB (~123 GiB usable) |
| Model budget | ~90 GB, with ~30 GB headroom (Hermes Agent and the desktop also run) |
| OS | Ubuntu (release not recorded) |
| Driver / CUDA | not recorded |
| Backend | CUDA |
| Role | Desktop that also runs local models |

## Suggested layout

```text
nvidia-rtx4090-24gb-128gb/
  README.md                          # this file
  models/<model>-<quant>.md          # one copy of TEMPLATE.md per model and quant
  llama-server-flashnext.service     # unit installed on this desktop
  llama.cpp.md                       # optional: extra build notes, once they outgrow this file
  benchmarks/<run>.md                # optional pasted llama-bench output
```

## Models and quants

System RAM is the fit limit for this quant. The 24GB GPU holds attention, the KV cache, and a MoE cache. Experts stay in RAM. The engram/PLE table stays on disk.

| Model | Quant | Context | Backend | PP tok/s | TG tok/s | Memory | Date | Entry |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Qwen3.8-Flash-Next | UD-IQ4_XS | 262144 | CUDA `b12557` | not measured | ~40 | ~62 GB experts in RAM; 28.8 GB engram mmap'd; ~3.2 GiB KV | 2026-11-20 | [entry](models/qwen3.8-flash-next-ud-iq4_xs.md) |

TG is generation once the service is warm at 256K, single stream (`--parallel 1`), no MTP. Prompt processing was not measured. The memory column is the recorded split of the ~93.7 GB on-disk model (experts, engram table, KV), not an `nvidia-smi` reading. The cold 32K log (29.50 t/s average, 31.3 t/s steady) and the slower MTP trial are in the [model entry](models/qwen3.8-flash-next-ud-iq4_xs.md).

## llama.cpp build

Recorded for the Flash-Next server. Stock tag `b12557`. This no-MTP recipe needs no source patch.

- **Commit:** [`b94959a64d011915d77a5381c7d4846dcda99530`](https://github.com/ggml-org/llama.cpp/commit/b94959a64d011915d77a5381c7d4846dcda99530) (2026-11-19 22:21:24 +0000), tag [`b12557`](https://github.com/ggml-org/llama.cpp/commit/b94959a64d011915d77a5381c7d4846dcda99530). Official tree `https://github.com/ggml-org/llama.cpp.git`.
- **Binary:** `/home/ubuntu/llama.cpp-flashnext/build/bin/llama-server`
- **GPU arch:** `sm_89` (`CMAKE_CUDA_ARCHITECTURES=89`)
- **Toolkit:** CUDA. The toolkit version was not recorded.
- **OS:** Ubuntu desktop `AMD9900-ML-ubuntu`. Release, kernel, and NVIDIA driver were not recorded.

The checkout that produced the binary is `/home/ubuntu/llama.cpp-flashnext`. Weights live under `/home/ubuntu/llama.cpp/models/`.

```
cmake -S . -B build -DGGML_CUDA=ON -DCMAKE_CUDA_ARCHITECTURES=89 -DCMAKE_BUILD_TYPE=Release && cmake --build build --config Release -j --target llama-server llama-cli llama-bench
```

## Server config

[`llama-server-flashnext.service`](llama-server-flashnext.service) is the unit installed at `/etc/systemd/system/llama-server-flashnext.service`. It serves Qwen3.8-Flash-Next UD-IQ4_XS on `0.0.0.0:8080` with context 262144, q8_0 KV cache, flash attention, `--cpu-moe`, and `--moe-cache-mib 9000`. Recorded unit keys: `User=ubuntu`, `Environment="CUDA_VISIBLE_DEVICES=0"`, `TimeoutStartSec=600`, and the `ExecStart` in that file. MTP is off. `mmap` stays on.

The command, the `mmap` warning, and the MTP comparison are in the [model entry](models/qwen3.8-flash-next-ud-iq4_xs.md).

## Benchmarks

2026-11-20, no MTP. Warm service at 256K (GPU MoE cache and page cache): ~40 t/s generation. Cold single request right after startup, 32K, `--moe-cache-mib 9000`: 29.50 t/s average, 31.3 t/s steady. Prompt processing was not measured. No `nvidia-smi` or `free` paste was kept.

An MTP trial on the same day (`--spec-type draft-mtp`, `mtp-Qwen3.8-Flash-Next-Q8_0.gguf`, `--spec-draft-n-max 3`, `--moe-cache-mib 6000`, 256K) accepted 70% of drafts (mean length 3.10) and logged 23.72 t/s average, 27.7 t/s warmed up. That configuration is not the unit. The comparison is in the [model entry](models/qwen3.8-flash-next-ud-iq4_xs.md).

## Notes and gotchas

- Keep `mmap` enabled. The 28.8 GB engram/PLE table stays on disk. llama.cpp's `--load-mode none` warning is the one to ignore here: `--load-mode none` would pull that table into the 128GB RAM budget, on top of the ~62 GB of experts already resident.
- `--n-gpu-layers 99 --cpu-moe --moe-cache-mib 9000` is the production offload. The GPU holds attention (12 layers), the q8_0 KV cache (~3.2 GiB at 256K), and a 9000 MiB MoE cache. Experts stay in RAM.
- The MTP trial was slower at 70% draft acceptance. Verifying 4 tokens at once pulls ~3–4× more unique CPU-offloaded experts over DDR5 per step, and the ~3.9 GB MTP head takes VRAM away from `--moe-cache-mib` (6000 MiB with the head, 9000 MiB without). The unit is stock `b12557` with MTP left off. The trial used a one-line `qwen4exp.cpp` workaround for [ggml-org/llama.cpp#30860](https://github.com/ggml-org/llama.cpp/issues/30860).
- Hermes Agent and the desktop share this host. The model budget is ~90 GB, leaving ~30 GB.

## Other runtimes

No vLLM or other runtime has been recorded on this machine yet. Use the same sections (build, server command, benchmark table) when one is added.
