# Qwen3.8-27B Q4_K_M

`llama-bench` comparison of two llama.cpp ROCm builds on the RX 7900 XTX, measured 2026-10-26. The same command was used for both builds. No server config was recorded. `llama-bench` does not run MTP drafting.

- **Date:** 2026-10-26
- **Hardware:** [amd-rx7900xtx-24gb](../README.md)
- **Runtime:** llama.cpp
- **Backend:** ROCm

## Model

- **Name:** Qwen3.8-27B
- **Source:** TODO. The Hugging Face repo for this GGUF was not recorded.
- **Quant:** Q4_K_M
- **Quant publisher:** TODO
- **File:** `/home/alex/models/Qwen3.8-27B-Q4_K_M.gguf`
- **Disk size:** `llama-bench` reports 15.92 GiB and 27.32 B parameters. The on-disk byte size was not recorded separately.
- **Bench label:** `qwen35 27B Q4_K - Medium`

## Build

Two ROCm builds of the same bench command. The new build's CMake line is the intended config. The old build's flags were not recorded.

### Old: `df03399b8` (2026-09-10)

- **Commit / version:** [`df03399b885831b2a1603b3abb0d8c156808e363`](https://github.com/ggml-org/llama.cpp/commit/df03399b885831b2a1603b3abb0d8c156808e363) (2026-09-10 11:25:40 -0700). The short hash from the machine was `df03399b8`.
- **GPU arch:** `gfx1100`
- **Toolkit:** ROCm. The ROCm version was not recorded.
- **OS / kernel / driver:** Fedora. Release, kernel, and driver were not recorded.
- **Configure command:** unknown

### New: `8b28c2394` (build 11898, 2026-10-26)

- **Commit / version:** [`8b28c2394324465bf7727089e7eaf7aec9442d6a`](https://github.com/ggml-org/llama.cpp/commit/8b28c2394324465bf7727089e7eaf7aec9442d6a) (2026-10-26 11:38:55 +0100), build 11898. The short hash from the machine was `8b28c2394`.
- **GPU arch:** `gfx1100`
- **Toolkit:** ROCm. The ROCm version was not recorded.
- **OS / kernel / driver:** Fedora. Release, kernel, and driver were not recorded.
- **Configure command:** intended config for this ROCm build:

```
cmake -B build -DGGML_HIP=ON -DAMDGPU_TARGETS=gfx1100 -DGGML_HIP_ROCWMMA_FATTN=ON -DCMAKE_BUILD_TYPE=Release
```

`llama-bench` reported the device as `AMD Radeon RX 7900 XTX, gfx1100, VMM: no, Wave Size: 32, VRAM: 24560 MiB`.

## Server

TODO. No launch command or systemd unit was recorded for this model.

- **Context loaded:** TODO
- **Model max context:** TODO
- **KV cache:** f16 / f16 in the bench (`-ctk f16 -ctv f16`). The server cache type was not recorded.
- **Offload:** `-ngl 99` in the bench. A server `-ngl` was not recorded.
- **Parallel slots:** TODO
- **Endpoint:** TODO

## Benchmark

Both builds, server not part of this measurement. Command:

```
llama-bench -m Qwen3.8-27B-Q4_K_M.gguf -ngl 99 -fa 1 -ctk f16 -ctv f16 -p 512 -n 128 -d 0,32768 -r 2
```

The model file used is `/home/alex/models/Qwen3.8-27B-Q4_K_M.gguf`. The command above is what was run; it passes the filename rather than that absolute path.

Depth 0 / 32K only. 128K and 256K depths were not measured. `llama-bench` excludes MTP drafting, so a served run with speculative decoding can differ from these tg128 numbers.

| Test | Old `df03399b8` | New `8b28c2394` | Change |
| --- | --- | --- | --- |
| pp512 | 1012.58 ± 31.92 | 1081.42 ± 12.73 | +6.8% |
| tg128 | 34.80 ± 0.08 | 35.61 ± 0.05 | +2.3% |
| pp512 @ d32768 | 596.43 ± 5.59 | 752.52 ± 1.06 | +26.2% |
| tg128 @ d32768 | 31.84 ± 0.00 | 32.50 ± 0.02 | +2.1% |

Change is `(new − old) / old`, rounded to one decimal place from the reported means:

- pp512: `(1081.42 − 1012.58) / 1012.58` = +6.798% → +6.8%
- tg128: `(35.61 − 34.80) / 34.80` = +2.328% → +2.3%
- pp512 @ d32768: `(752.52 − 596.43) / 596.43` = +26.171% → +26.2%
- tg128 @ d32768: `(32.50 − 31.84) / 31.84` = +2.073% → +2.1%

| Date | Method | Prompt tokens | Gen tokens | PP tok/s | TG tok/s | Memory | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2026-10-26 | `llama-bench`, old `df03399b8`, `-d 0 -r 2` | 512 | 128 | 1012.58 ± 31.92 | 34.80 ± 0.08 | not measured | depth 0 |
| 2026-10-26 | `llama-bench`, old `df03399b8`, `-d 32768 -r 2` | 512 | 128 | 596.43 ± 5.59 | 31.84 ± 0.00 | not measured | depth 32K |
| 2026-10-26 | `llama-bench`, new `8b28c2394`, `-d 0 -r 2` | 512 | 128 | 1081.42 ± 12.73 | 35.61 ± 0.05 | not measured | depth 0 |
| 2026-10-26 | `llama-bench`, new `8b28c2394`, `-d 32768 -r 2` | 512 | 128 | 752.52 ± 1.06 | 32.50 ± 0.02 | not measured | depth 32K |

- **Fit:** the bench requested `-ngl 99` on ROCm. The card reported 24560 MiB VRAM. Resident memory during the run was not recorded. Per-layer offload was not logged.
- **Memory tool:** none for this run. The 15.92 GiB and 24560 MiB figures are what `llama-bench` printed for the model and the device.
- **Raw output:** the owner recorded the device line, the model label, and the four rates per build above. A full `llama-bench` table was not pasted.

## Client

- **Client:** not recorded
- **Base URL:** TODO
- **Tool calling / chat template:** TODO. No server was configured in this entry.

## Notes

The old and new checkouts are 46 days apart. Only the new build's CMake flags are known, so the +26.2% pp512 at depth 32K is the difference between those two binaries, not a single-flag result. `-DGGML_HIP_ROCWMMA_FATTN=ON` is part of the new build's intended config. The old build may or may not have had it.

Repeats were `-r 2`. The ± values are the ones `llama-bench` printed.
