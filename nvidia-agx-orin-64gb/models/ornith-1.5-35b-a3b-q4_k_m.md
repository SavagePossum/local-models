# Ornith-1.5-35B-A3B Q4_K_M

Serving configuration observed on the AGX Orin on 2026-10-26. `llama-bench` numbers from that day are below. The bench does not include MTP speculative decoding.

- **Date:** 2026-10-26
- **Hardware:** [nvidia-agx-orin-64gb](../README.md)
- **Runtime:** llama.cpp
- **Backend:** CUDA

## Model

- **Name:** Ornith-1.5-35B-A3B
- **Source:** [bartowski/Ornith-1.5-35B-A3B-GGUF](https://huggingface.co/bartowski/Ornith-1.5-35B-A3B-GGUF) (`Ornith-1.5-35B-A3B-Q4_K_M.gguf`). Base weights: [ornith-ai/Ornith-1.5-35B-A3B](https://huggingface.co/ornith-ai/Ornith-1.5-35B-A3B).
- **Quant:** Q4_K_M
- **Quant publisher:** bartowski (imatrix quantization of the ornith-ai base model)
- **File:** `/mnt/data/models/Ornith-1.5-35B-A3B-Q4_K_M.gguf`
- **Disk size:** 21,864,081,056 bytes (21.86 GB). File mtime on the Orin was 2026-08-27.

The filename and byte size match [bartowski's Q4_K_M file](https://huggingface.co/bartowski/Ornith-1.5-35B-A3B-GGUF/blob/main/Ornith-1.5-35B-A3B-Q4_K_M.gguf) exactly. The official ornith-ai GGUF repo publishes a different file, `Ornith-1.5-35B-Q4_K_M.gguf` (21,713,463,040 bytes), so that download is not this setup.

GGUF metadata on the bartowski repo reports `context_length` 262144 for this architecture (`qwen35moe`). No checksum was recorded on the Orin.

## Build

- **Commit / version:** [`704485942ab54bbbbf1f241b3550ffba35f5f37e`](https://github.com/ggml-org/llama.cpp/commit/704485942ab54bbbbf1f241b3550ffba35f5f37e) (2026-08-11 11:47:57 +0200). `git log -1 --format="%h %ci"` on the Orin printed `704485942 2026-08-11 11:47:57 +0200`.
- **GPU arch:** `sm_87` (AGX Orin). The CMake architecture flag used for this build was not recorded.
- **Toolkit:** CUDA, via `LD_LIBRARY_PATH=/usr/local/cuda/lib64`. The CUDA toolkit version was not recorded.
- **OS / kernel / driver:** L4T R36.4.7. `/etc/nv_tegra_release` reads:

```
# R36 (release), REVISION: 4.7, GCID: 42132812, BOARD: generic, EABI: aarch64, DATE: Thu Sep 18 22:54:44 UTC 2025
# KERNEL_VARIANT: oot
```

The JetPack meta-package version was not queried. This note records the L4T release from that file and does not map it to a JetPack number.

- **Binary:** `/home/nvidia/src/llama.cpp/build/bin/llama-server`
- **Configure command:** not recorded

Power mode for the running server: `nvpmodel -m 0`, then `jetson_clocks`. The unit runs both before `llama-server` because `jetson_clocks` does not persist across reboot.

## Server

- **Context loaded:** 262144 (`--ctx-size 262144`)
- **Model max context:** 262144 (GGUF `context_length` on the source repo)
- **KV cache:** f16 / f16
- **Offload:** `--n-gpu-layers 99`
- **Parallel slots:** 1
- **Sampling:** temp 0.6, top-p 0.95, top-k 20
- **Other flags:** flash attention on, `--cache-reuse 256`, `--jinja`, `--reasoning-preserve`, MTP speculative decoding (`--spec-type draft-mtp --spec-draft-n-max 2`), `--metrics`, batch 2048, ubatch 512, 6 threads
- **Endpoint:** OpenAI-compatible API on port 8080, bound to `0.0.0.0`. Base URL `http://<orin-host>:8080/v1`. Alias `ornith-1.5-35b-a3b`.

Installed on the board as `/etc/systemd/system/llama-server.service`. The same unit is in this repo at [`configs/llama-server.service`](../configs/llama-server.service):

```ini
[Unit]
Description=llama.cpp server (Ornith-1.5-35B-A3B)
After=network-online.target
Wants=network-online.target

[Service]
Type=exec
User=nvidia
Group=nvidia
Environment=LD_LIBRARY_PATH=/usr/local/cuda/lib64

# Re-apply max clocks after reboot (jetson_clocks does not persist)
ExecStartPre=+/usr/sbin/nvpmodel -m 0
ExecStartPre=+/usr/bin/jetson_clocks

ExecStart=/home/nvidia/src/llama.cpp/build/bin/llama-server \
  --model /mnt/data/models/Ornith-1.5-35B-A3B-Q4_K_M.gguf \
  --alias ornith-1.5-35b-a3b \
  --host 0.0.0.0 \
  --port 8080 \
  --n-gpu-layers 99 \
  --ctx-size 262144 \
  --flash-attn on \
  --cache-type-k f16 \
  --cache-type-v f16 \
  --batch-size 2048 \
  --ubatch-size 512 \
  --threads 6 \
  --parallel 1 \
  --cache-reuse 256 \
  --jinja \
  --reasoning-preserve \
  --temp 0.6 \
  --top-p 0.95 \
  --top-k 20 \
  --spec-type draft-mtp \
  --spec-draft-n-max 2 \
  --metrics

# 29 GB model + 5 GB KV allocation takes a while on first start
TimeoutStartSec=900
Restart=on-failure
RestartSec=15
LimitMEMLOCK=infinity
LimitNOFILE=65536
OOMScoreAdjust=-500

[Install]
WantedBy=multi-user.target
```

`ExecStartPre=+/...` runs those two commands as root. The server process itself runs as `nvidia`.

## Benchmark

Measured 2026-10-26 with `llama-bench`. The server was stopped. Clocks were the same as the unit: `nvpmodel -m 0`, then `jetson_clocks`.

Depth 0 / 32K / 128K: prompt processing 631 / 528 / 327 t/s, generation 38.3 / 33.4 / 24.1 t/s. Those are `pp512` and `tg128` at KV depth 0, 32768, and 131072, rounded from the table below. 256K depth was not measured. The server itself is started at context 262144.

`llama-bench` does not run `--spec-type draft-mtp`. Served generation speed with MTP speculative decoding (`--spec-draft-n-max 2`) may differ from these tg128 numbers. The unit's `--metrics` endpoint is the place to record that; those rates are not here.

Command:

```
~/src/llama.cpp/build/bin/llama-bench -m /mnt/data/models/Ornith-1.5-35B-A3B-Q4_K_M.gguf -ngl 99 -fa 1 -ctk f16 -ctv f16 -t 6 -b 2048 -ub 512 -p 512 -n 128 -d 0,32768,131072 -r 2
```

Raw output:

```
ggml_cuda_init: found 1 CUDA devices (Total VRAM: 62852 MiB): Device 0: Orin, compute capability 8.7, VMM: yes
| model | size | params | backend | ngl | threads | fa | test | t/s |
| qwen35moe 35B.A3B Q4_K - Medium | 20.35 GiB | 35.51 B | CUDA | 99 | 6 | 1 | pp512 | 631.11 ± 0.76 |
| ... | tg128 | 38.26 ± 0.10 |
| ... | pp512 @ d32768 | 528.05 ± 2.22 |
| ... | tg128 @ d32768 | 33.38 ± 0.10 |
| ... | pp512 @ d131072 | 326.88 ± 0.32 |
| ... | tg128 @ d131072 | 24.05 ± 0.06 |
build: 704485942 (10367)
```

| Date | Method | Prompt tokens | Gen tokens | PP tok/s | TG tok/s | Memory | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2026-10-26 | `llama-bench -d 0 -r 2` | 512 | 128 | 631.11 ± 0.76 | 38.26 ± 0.10 | not measured during the bench | depth 0 |
| 2026-10-26 | `llama-bench -d 32768 -r 2` | 512 | 128 | 528.05 ± 2.22 | 33.38 ± 0.10 | not measured during the bench | depth 32K |
| 2026-10-26 | `llama-bench -d 131072 -r 2` | 512 | 128 | 326.88 ± 0.32 | 24.05 ± 0.06 | not measured during the bench | depth 128K |
| 2026-10-26 | `free -g` while serving | | | | | ~37 GB used of 61 GB total | swap unused; separate from the bench |

- **Fit:** the bench loaded the model with `-ngl 99` on CUDA (`fa` 1, f16 KV). It reported one device, Orin, compute capability 8.7, total VRAM 62852 MiB. `build: 704485942 (10367)` matches the llama.cpp checkout in this entry. While the server was up, `free -g` reported about 37 GB used of 61 GB total and swap unused (0 of 30 GB). Per-layer offload was not logged.
- **Memory tool:** `free -g` for the serving row. The bench did not print a resident-memory figure.

`free -g` while serving:

```
               total        used        free      shared  buff/cache   available
Mem:              61          37           1           0          22          22
Swap:             30           0          30
```

`free -g` prints whole gigabytes. The module is specified as 64GB unified memory; the command reported 61 GB total and about 37 GB used while this server was up.

## Client

- **Client:** Hermes Agent (Nous Research)
- **Base URL:** `http://<orin-host>:8080/v1` (OpenAI-compatible, port 8080)
- **Tool calling / chat template:** not recorded. The unit enables `--jinja` and `--reasoning-preserve` for that client.

## Notes

The unit comment says "29 GB model + 5 GB KV allocation". The file actually loaded is the Q4_K_M GGUF at 21.86 GB. The comment likely dates from an earlier Q6 model. The same directory still has `Qwen3.6-35B-A3B-UD-Q6_K.gguf` at 29,308,320,736 bytes (29.3 GB), which matches the "29 GB" figure. That file is not what this unit serves. A third file, `NVIDIA-Nemotron-3.5-Lightning-30B-A3B-Q4_K_M.gguf` (25,430,738,944 bytes), is also on disk and is not part of this entry. The "5 GB KV" half of the comment was not measured separately.

`TimeoutStartSec=900` is there because the first start, with a 262144 context, takes long enough to trip a default systemd timeout.
