# Qwen3.8-Flash-Next UD-IQ4_XS

CUDA llama.cpp on the Ubuntu RTX 4090 desktop with 128GB of system RAM, 2026-11-20. The build is llama.cpp tag `b12557` (`b94959a64d`). The server loads context 262144 with no MTP. Stock `b12557` runs this recipe with no source patch.

The production rate is ~40 t/s generation once the service is warm at 256K. A cold single request at 32K is a separate, slower log. An MTP trial the same day was slower still and is not the unit.

- **Date:** 2026-11-20
- **Hardware:** [nvidia-rtx4090-24gb-128gb](../README.md)
- **Runtime:** llama.cpp
- **Backend:** CUDA

## Model

- **Name:** Qwen3.8-Flash-Next
- **Source:** [unsloth/Qwen3.8-Flash-Next-GGUF](https://huggingface.co/unsloth/Qwen3.8-Flash-Next-GGUF), folder [`UD-IQ4_XS`](https://huggingface.co/unsloth/Qwen3.8-Flash-Next-GGUF/tree/main/UD-IQ4_XS)
- **Quant:** UD-IQ4_XS
- **Quant publisher:** Unsloth
- **File:** 4-split GGUF. `--model` points at the first shard, `Qwen3.8-Flash-Next-UD-IQ4_XS-00001-of-00004.gguf`. On the machine: `/home/ubuntu/llama.cpp/models/Qwen3.8-Flash-Next-GGUF/UD-IQ4_XS/Qwen3.8-Flash-Next-UD-IQ4_XS-00001-of-00004.gguf`.
- **Disk size:** ~93.7 GB across the four shards. 28.8 GB of that is the engram/PLE table, kept on disk with `mmap`. ~62 GB of experts are resident in RAM. A checksum was not recorded.

Download:

```
hf download unsloth/Qwen3.8-Flash-Next-GGUF --include "UD-IQ4_XS/*" --local-dir /home/ubuntu/llama.cpp/models/Qwen3.8-Flash-Next-GGUF
```

The model has 12 attention layers. A 256K q8_0 KV cache for those layers is ~3.2 GiB. The publisher's advertised maximum context was not recorded separately from the 262144 loaded below.

## Build

- **Commit / version:** tag `b12557`, commit [`b94959a64d011915d77a5381c7d4846dcda99530`](https://github.com/ggml-org/llama.cpp/commit/b94959a64d011915d77a5381c7d4846dcda99530) (2026-11-19 22:21:24 +0000). Official repository `https://github.com/ggml-org/llama.cpp.git`. The tag points at that commit.
- **GPU arch:** `sm_89`
- **Toolkit:** CUDA. The toolkit version was not recorded.
- **OS / kernel / driver:** Ubuntu desktop `AMD9900-ML-ubuntu`, AMD Ryzen 9 9950X (16c/32t), NVIDIA RTX 4090 24GB. Ubuntu release, kernel, NVIDIA driver, and CUDA toolkit version were not recorded.
- **Binary:** `/home/ubuntu/llama.cpp-flashnext/build/bin/llama-server`. That path is the checkout for this recipe (`/home/ubuntu/llama.cpp-flashnext`). Weights stay under `/home/ubuntu/llama.cpp/models/`.
- **Configure command:**

```
cmake -S . -B build -DGGML_CUDA=ON -DCMAKE_CUDA_ARCHITECTURES=89 -DCMAKE_BUILD_TYPE=Release && cmake --build build --config Release -j --target llama-server llama-cli llama-bench
```

Targets built: `llama-server`, `llama-cli`, `llama-bench`. Stock `b12557` serves the unit below. The MTP trial further down used a one-line `qwen4exp.cpp` workaround; that patch is not part of this build.

## Server

- **Context loaded:** 262144 (`--ctx-size 262144`, 256K)
- **Model max context:** not recorded
- **KV cache:** q8_0 / q8_0 (~3.2 GiB at 256K, 12 attention layers)
- **Offload:** partial. `--n-gpu-layers 99 --cpu-moe --moe-cache-mib 9000`. Experts are CPU-offloaded (~62 GB in RAM) with a 9000 MiB GPU MoE cache. The 28.8 GB engram/PLE table stays on disk via `mmap`.
- **Parallel slots:** 1
- **Threads:** 16
- **Sampling:** temp 1.0, top-p 0.95, top-k 20, min-p 0.0
- **Other flags:** `--flash-attn on`, `--jinja`, `--metrics`, `--batch-size 2048`, `--ubatch-size 512`, alias `qwen3.8-flash-next`. `CUDA_VISIBLE_DEVICES=0`.
- **Endpoint:** OpenAI-compatible API on port 8080, bound to `0.0.0.0`. Base URL `http://<host>:8080/v1`.

`mmap` stays enabled. Ignore llama.cpp's `--load-mode none` warning. `--load-mode none` would read the 28.8 GB engram table into RAM, on top of the ~62 GB of experts, inside a 128GB machine that also runs Hermes Agent and a desktop.

Launch command, as installed:

```
/home/ubuntu/llama.cpp-flashnext/build/bin/llama-server \
  --model /home/ubuntu/llama.cpp/models/Qwen3.8-Flash-Next-GGUF/UD-IQ4_XS/Qwen3.8-Flash-Next-UD-IQ4_XS-00001-of-00004.gguf \
  --alias qwen3.8-flash-next \
  --host 0.0.0.0 --port 8080 \
  --n-gpu-layers 99 --cpu-moe --moe-cache-mib 9000 \
  --ctx-size 262144 --flash-attn on \
  --cache-type-k q8_0 --cache-type-v q8_0 \
  --batch-size 2048 --ubatch-size 512 --threads 16 --parallel 1 \
  --jinja --temp 1.0 --top-p 0.95 --top-k 20 --min-p 0.0 \
  --metrics
```

Installed as `/etc/systemd/system/llama-server-flashnext.service`. The same recorded keys are [`../llama-server-flashnext.service`](../llama-server-flashnext.service):

```ini
# Installed on AMD9900-ML-ubuntu as /etc/systemd/system/llama-server-flashnext.service.
# Recorded keys are User, Environment, ExecStart, and TimeoutStartSec.
# Type, Group, Restart, and [Install] were not recorded.

[Service]
User=ubuntu
Environment="CUDA_VISIBLE_DEVICES=0"
ExecStart=/home/ubuntu/llama.cpp-flashnext/build/bin/llama-server \
  --model /home/ubuntu/llama.cpp/models/Qwen3.8-Flash-Next-GGUF/UD-IQ4_XS/Qwen3.8-Flash-Next-UD-IQ4_XS-00001-of-00004.gguf \
  --alias qwen3.8-flash-next \
  --host 0.0.0.0 --port 8080 \
  --n-gpu-layers 99 --cpu-moe --moe-cache-mib 9000 \
  --ctx-size 262144 --flash-attn on \
  --cache-type-k q8_0 --cache-type-v q8_0 \
  --batch-size 2048 --ubatch-size 512 --threads 16 --parallel 1 \
  --jinja --temp 1.0 --top-p 0.95 --top-k 20 --min-p 0.0 \
  --metrics
TimeoutStartSec=600
```

`Type`, `Group`, `Restart`, and `[Install]` were not recorded, so this copy is the verified keys rather than a guessed full unit.

## Benchmark

Three observations from 2026-11-20. The first is the service after it is warm. The second is one cold request just after startup. The third is the MTP trial, which is not the unit. Prompt-processing tok/s were not measured for any of them. Token counts for those logs were not recorded. No raw `llama-bench` table was pasted.

| Date | Method | Prompt tokens | Gen tokens | PP tok/s | TG tok/s | Memory | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2026-11-20 | service, warm GPU MoE cache + page cache, 256K, no MTP | not measured | not measured | not measured | ~40 | not measured | production unit, `--moe-cache-mib 9000`, `--parallel 1` |
| 2026-11-20 | cold single request right after startup, 32K, no MTP | not measured | not measured | not measured | 29.50 avg (31.3 steady) | not measured | `--moe-cache-mib 9000` |
| 2026-11-20 | `--spec-type draft-mtp`, 256K, `--moe-cache-mib 6000` | not measured | not measured | not measured | 23.72 avg (27.7 warmed up) | not measured | not the unit; 70% acceptance, mean length 3.10 |

- **Fit:** partial offload on a 24GB RTX 4090. Attention and a 9000 MiB MoE cache are on the GPU. ~62 GB of experts are in system RAM. The 28.8 GB engram/PLE table is mmap'd from disk. The host has 128GB of RAM, ~123 GiB usable, and also runs Hermes Agent and a desktop, so the model is budgeted at ~90 GB with ~30 GB of headroom. The q8_0 KV cache at 256K is ~3.2 GiB (12 attention layers).
- **Memory tool:** not recorded. The figures above are the recorded sizes and the RAM budget. No `nvidia-smi` or `free` paste was kept.
- **Raw output:** not recorded. The cold log was written down as 29.50 t/s average and 31.3 t/s steady. The MTP log was written down as 23.72 t/s average and 27.7 t/s warmed up. The warm service rate was written down as ~40 t/s.

## Client

- **Client:** The desktop also runs [Hermes Agent](https://github.com/NousResearch/hermes-agent). Which client produced the timings above was not recorded separately.
- **Base URL:** `http://<host>:8080/v1`. On the desktop, `<host>` is `127.0.0.1`. Alias `qwen3.8-flash-next`.
- **Tool calling / chat template:** `--jinja` is on, so the server applies the chat template in the GGUF. Tool-calling behavior was not recorded.

## Notes

### mmap

Keep `mmap` enabled. llama.cpp warns about `--load-mode none` on this load. Leave that warning alone. The 28.8 GB engram/PLE table has to stay on disk. Pulling it into RAM would land on top of the ~62 GB of experts, inside the ~90 GB model budget on a 128GB host (~123 GiB usable) that also runs Hermes Agent and the desktop.

### Why no MTP

`--spec-type draft-mtp` was tested on this machine and left out of the unit. The draft file was `mtp-Qwen3.8-Flash-Next-Q8_0.gguf`, with `--spec-draft-n-max 3`, `--moe-cache-mib 6000`, and context 256K. Loading that head on this tree took a one-line `qwen4exp.cpp` workaround for [ggml-org/llama.cpp#30860](https://github.com/ggml-org/llama.cpp/issues/30860).

Draft acceptance was 70% (mean length 3.10). Generation was still slower than the no-MTP service: 23.72 t/s average, 27.7 t/s warmed up, against ~40 t/s once the no-MTP service is warm at 256K. Verifying 4 tokens at once pulls ~3–4× more unique CPU-offloaded experts over DDR5 per step. The MTP head is ~3.9 GB and takes VRAM that the no-MTP unit gives to `--moe-cache-mib` (6000 MiB with the head, 9000 MiB without). Dropping MTP and giving that VRAM to the MoE cache was the faster configuration. The installed unit is stock `b12557` and does not carry the `qwen4exp.cpp` workaround.
