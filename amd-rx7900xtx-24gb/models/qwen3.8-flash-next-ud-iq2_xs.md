# Qwen3.8-Flash-Next UD-IQ2_XS

ROCm llama.cpp on the RX 7900 XTX desktop, measured and served 2026-11-11. Replaces the previous default [Qwen3.8-27B Q4_K_M](qwen3.8-27b-q4_k_m.md) as the always-on `llama-server` on port 8080. MoE experts and the PLE / n-gram table are offloaded; attention stays on the GPU. MTP was tried and dropped (stock llama.cpp could not load Unsloth shared MTP heads).

- **Date:** 2026-11-11
- **Hardware:** [amd-rx7900xtx-24gb](../README.md)
- **Runtime:** llama.cpp
- **Backend:** ROCm/HIP

## Model

- **Name:** Qwen3.8-Flash-Next
- **Source:** [unsloth/Qwen3.8-Flash-Next-GGUF](https://huggingface.co/unsloth/Qwen3.8-Flash-Next-GGUF), folder [`UD-IQ2_XS`](https://huggingface.co/unsloth/Qwen3.8-Flash-Next-GGUF/tree/main/UD-IQ2_XS)
- **Quant:** UD-IQ2_XS (Unsloth Dynamic)
- **Quant publisher:** Unsloth
- **File:** three shards; point `-m` at the first:

  - `Qwen3.8-Flash-Next-UD-IQ2_XS-00001-of-00003.gguf`
  - `Qwen3.8-Flash-Next-UD-IQ2_XS-00002-of-00003.gguf` (~47 GB)
  - `Qwen3.8-Flash-Next-UD-IQ2_XS-00003-of-00003.gguf` (~29 GB, PLE / engram table)

  On the machine: `/home/alex/models/Qwen3.8-Flash-Next-GGUF/UD-IQ2_XS/`

- **Disk size:** ~76 GB across the three shards (Hugging Face LFS sizes; not re-hashed locally)
- **Download:**

```
hf download unsloth/Qwen3.8-Flash-Next-GGUF \
  --include "UD-IQ2_XS/*" \
  --local-dir ~/models/Qwen3.8-Flash-Next-GGUF
```

Model max context is 262144. This host loads **131072**. `-c 142000` loaded but aborted under the built-in WebUI (systemd `status=6/ABRT`).

## Build

- **Commit / version:** not recorded as a full SHA. Rebuilt from a current `~/llama.cpp` checkout on 2026-11-11 after the earlier ROCm build `8b28c2394` (11898). Binary installed to `/opt/llama.cpp/bin/llama-server`.
- **GPU arch:** `gfx1100`
- **Toolkit:** ROCm. Version not recorded.
- **OS / kernel / driver:** Fedora. Release, kernel, and driver not recorded.
- **Configure command:**

```
cmake -B build -DGGML_HIP=ON -DAMDGPU_TARGETS=gfx1100 \
  -DGGML_HIP_ROCWMMA_FATTN=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release -j$(nproc) \
  --target llama-server llama-cli llama-bench
sudo install -D -m 755 build/bin/llama-server /opt/llama.cpp/bin/llama-server
```

Wipe a stale `build/` first if CMakeCache points at another tree (`llama.cpp-new` vs `llama.cpp`).

## Server

- **Context loaded:** 131072
- **Model max context:** 262144
- **KV cache:** default (not overridden in the unit)
- **Offload:** partial. `--n-cpu-moe 30` keeps MoE experts on CPU; `--override-tensor per_layer_token_embd=CPU` pins the PLE table to host; `--load-mode mmap` leaves that table pageable from NVMe. Do **not** combine `-ngl 99` with `--fit on` here: `-ngl 99` aborts fit and can try to allocate ~43 GB on the 24 GB card.
- **Parallel slots:** 1
- **Endpoint:** `http://0.0.0.0:8080/v1` (alias `qwen3.8-flash-next`)
- **Sampling (thinking defaults):** `temp 1.0`, `top_p 0.95`, `top_k 20`, `min_p 0.0`

Launch command (same flags as the unit):

```
/opt/llama.cpp/bin/llama-server \
  -m /home/alex/models/Qwen3.8-Flash-Next-GGUF/UD-IQ2_XS/Qwen3.8-Flash-Next-UD-IQ2_XS-00001-of-00003.gguf \
  --alias qwen3.8-flash-next \
  --host 0.0.0.0 --port 8080 \
  -c 131072 --parallel 1 -fa on \
  --fit on --fit-target 2048 \
  --load-mode mmap \
  --override-tensor per_layer_token_embd=CPU \
  --n-cpu-moe 30 \
  --jinja \
  --temp 1.0 --top-p 0.95 --top-k 20 --min-p 0.0 \
  --threads-http 8 \
  --no-reasoning-preserve
```

systemd unit: [`../llama-server.service`](../llama-server.service) (installed as `/etc/systemd/system/llama-server.service`).

`--fit on` still prints `tensor_buft_overrides already set by user, abort`; that is expected with `--override-tensor` / `--n-cpu-moe`. Keep the offload flags.

## Benchmark

Server slot timings from interactive runs on 2026-11-11 (not `llama-bench`). Single stream. MTP off.

| Date | Method | Prompt tokens | Gen tokens | PP tok/s | TG tok/s | Memory | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2026-11-11 | `llama-server` slot timings, `-c 65536` | 393 | 355 | 50.31 | 20.04 | not measured | first validated Flash run |
| 2026-11-11 | `llama-server` slot timings, `-c 131072` | 766 | 372 | 49.69 | 19.46 | not measured | production context |

- **Fit:** partial GPU + system RAM + mmap PLE. Fully on GPU is not possible for this quant on 24 GB.
- **Memory tool:** not recorded for these runs. RSS under mmap stays low until pages are touched; watch `free` / `rocm-smi` under load.
- **Raw output (128K run, abbreviated):**

```
prompt eval time =   15414.39 ms /   766 tokens (   20.12 ms per token,    49.69 tokens per second)
       eval time =   19062.71 ms /   372 tokens (   51.38 ms per token,    19.46 tokens per second)
      total time =   34477.10 ms /  1138 tokens
```

Compared with the earlier full-VRAM [Qwen3.8-27B Q4_K_M](qwen3.8-27b-q4_k_m.md) bench (~35 tg/s at depth 0), Flash-Next is slower on this card but is the larger MoE model.

## Client

- **Client:** llama.cpp built-in WebUI and OpenAI-compatible `/v1` clients
- **Base URL:** `http://<host>:8080/v1`
- **Model id:** `qwen3.8-flash-next`
- **Tool calling / chat template:** `--jinja` enabled. Reasoning preserve disabled via `--no-reasoning-preserve`.

Smoke test:

```
curl -s http://127.0.0.1:8080/v1/models | jq .
curl -N http://127.0.0.1:8080/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{"model":"qwen3.8-flash-next","messages":[{"role":"user","content":"Say OK"}],"max_tokens":32,"stream":true}'
```

## Notes

- **Context ceiling on this box:** 131072 is the stable production value. 142000 loaded once but the service ABRTed when driven from the WebUI.
- **WebUI vs API:** `/v1` streaming via `curl` is reliable. The built-in webpage can show `Stream connection lost and could not be resumed` on this slow offload path even when the server is fine; prefer an external OpenAI-compatible client, or keep `--threads-http 8`.
- **Do not use `-ngl 99` with this recipe.** It disables `--fit` and previously tried to allocate ~44520 MiB on device 0.
- **MTP:** Unsloth `shared-Q8_0` MTP failed with `token_embd.weight not found` on this stock ROCm build. Non-shared MTP / Unsloth MTP fork not pursued; left off.
- **Port conflict:** stop any previous 27B unit before binding `:8080`.
- **`--include` for hf download:** use `UD-IQ2_XS/*` (path glob), not the bare folder name.
