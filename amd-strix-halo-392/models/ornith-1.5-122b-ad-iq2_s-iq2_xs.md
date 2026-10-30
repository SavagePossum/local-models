# Ornith-1.5-122B AD-IQ2_S-IQ2_XS

Vulkan llama.cpp on the Strix Halo laptop, 2026-10-30. The build is llama.cpp `e0f142d5d` (12005). `llama-bench` below has no speculative decoding. The server loads a 131072 context and turns on MTP. A 128K-depth bench is TODO.

This server is for long-running work where a reply can take a while: research, refactoring, and deep code review. Interactive use stays on Ornith-1.5-35B-A3B. The 35B setup recorded in this repo is the [AGX Orin entry](../../nvidia-agx-orin-64gb/models/ornith-1.5-35b-a3b-q4_k_m.md). No 35B run on this laptop is recorded here.

- **Date:** 2026-10-30
- **Hardware:** [amd-strix-halo-392](../README.md)
- **Runtime:** llama.cpp
- **Backend:** Vulkan (RADV)

## Model

- **Name:** Ornith-1.5-122B
- **Source:** [AtomicChat/Ornith-1.5-122B-GGUF](https://huggingface.co/AtomicChat/Ornith-1.5-122B-GGUF), file [`Ornith-1.5-122B-AD-IQ2_S-IQ2_XS.gguf`](https://huggingface.co/AtomicChat/Ornith-1.5-122B-GGUF/blob/main/Ornith-1.5-122B-AD-IQ2_S-IQ2_XS.gguf). Base weights: [ornith-ai/Ornith-1.5-122B](https://huggingface.co/ornith-ai/Ornith-1.5-122B).
- **Quant:** AD-IQ2_S-IQ2_XS
- **Quant publisher:** AtomicChat. The model card describes `AD-` as an Atomic Dynamic layout named for the two largest tensor groups (`ffn_down`, then `ffn_up`).
- **File:** `Ornith-1.5-122B-AD-IQ2_S-IQ2_XS.gguf` (one shard). On the laptop it lived at `/home/alex/models/Ornith-1.5-122B-AD-IQ2_S-IQ2_XS.gguf`.
- **Disk size:** 42,811,236,416 bytes (42.81 GB). That is the Hugging Face LFS size of this filename. `llama-bench` reported the loaded model as 39.86 GiB. The file was not re-hashed on the laptop.
- **Published SHA-256 (Hugging Face LFS):** `5ae43765722dc077c5f10f1c94a380b7a8dbe2e1706d1bf9ab7c7ac6eaf8e1d4`

The publisher's model card lists context length 262144. This file's GGUF metadata was not dumped locally. The server below loads 131072.

This GGUF keeps the MTP layers. The bench leaves them unused. The server enables them with `--spec-type draft-mtp --spec-draft-n-max 2`.

Nearby AtomicChat quants of the same model are listed under [Notes](#notes). Only this file was run.

## Host preparation

The iGPU does not see the full 64GB until the UMA frame buffer is minimized and the kernel GTT/TTM limit is raised. Do this before trusting a Vulkan heap size.

1. In the BIOS, set the UMA frame buffer to the minimum. The exact menu name and the resulting megabyte value were not written down.

2. Create `/etc/modprobe.d/ttm.conf`:

```
options ttm pages_limit=15204352 page_pool_size=15204352
```

`15204352` pages × 4 KiB = 58 GiB.

3. Apply it and reboot. Omarchy uses Limine. `sudo mkinitcpio -P` fails here with `No presets found`. The command that worked is:

```
sudo limine-update
```

Then reboot.

Fallback, if that module option is not picked up: append the same limits to `KERNEL_CMDLINE[default]` in `/etc/default/limine`, as kernel parameters `ttm.pages_limit=15204352 ttm.page_pool_size=15204352`. Run `sudo limine-update` again and reboot. The modprobe file plus `limine-update` is the path that was used.

4. After reboot, Vulkan reported:

```
Vulkan0: AMD Radeon 8060S Graphics (RADV STRIX_HALO) (59904 MiB, 59283 MiB free)
```

59904 MiB is 58.5 GiB. That line is what `./build/bin/llama-server --list-devices` prints. The 59283 MiB free figure is the unloaded device, not memory use while the model is resident.

## Build

- **Commit / version:** [`e0f142d5ddc1461ce721644d92af33db23545105`](https://github.com/ggml-org/llama.cpp/commit/e0f142d5ddc1461ce721644d92af33db23545105) (2026-10-30 04:30:05 +0000), build 12005. The short hash from the machine was `e0f142d5d`.
- **GPU arch:** `gfx1151`. The CMake line does not set an architecture. RADV selects the device at runtime and reported `RADV STRIX_HALO`.
- **Toolkit:** Vulkan via the Arch packages below (`vulkan-radeon`, RADV). The Mesa version was not recorded.
- **OS / kernel / driver:** Omarchy Linux (Arch-based), Limine bootloader, kernel version not recorded. iGPU: AMD Radeon 8060S, 40 CU.
- **Binary:** `/home/alex/llama.cpp/build/bin/llama-server` on the machine this was taken from. The unit in this folder uses `/home/USER/llama.cpp` so another account can substitute its own home directory.
- **Configure command:**

Packages:

```
sudo pacman -S --needed base-devel git cmake vulkan-radeon vulkan-icd-loader vulkan-headers vulkan-tools shaderc spirv-headers ccache python-pipx
```

`spirv-headers` is required. CMake fails without it. If that failure already happened, remove the build tree and configure again:

```
rm -rf build
```

`ccache` and `python-pipx` were installed with the rest. No further use of either was recorded.

From a llama.cpp checkout at the commit above:

```
cmake -B build -DGGML_VULKAN=ON -DCMAKE_BUILD_TYPE=Release && cmake --build build --config Release -j
```

Check the device:

```
./build/bin/llama-server --list-devices
```

The expected device line is the Vulkan string in [Host preparation](#host-preparation).

On this build, load mode replaces the older `-mmp` / mmap flags. The runs below pass `-lm none`.

## Server

- **Context loaded:** 131072 (`-c 131072`)
- **Model max context:** 262144 on the publisher's model card. Not confirmed from this file's GGUF metadata.
- **KV cache:** q8_0 / q8_0
- **Offload:** `-ngl 99`
- **Parallel slots:** 1
- **Load mode:** `-lm none`
- **Flash attention:** on (`-fa on` in the unit, `-fa 1` in the bench)
- **Sampling:** temp 0.6, top-p 0.95, top-k 20, min-p 0
- **Other flags:** `--jinja`, `--cache-reuse 256`, batch 2048, ubatch 512, `--metrics`, MTP (`--spec-type draft-mtp --spec-draft-n-max 2`)
- **Endpoint:** OpenAI-compatible API on port 8080, bound to `0.0.0.0`. Base URL `http://<host>:8080/v1`. No `--alias` was set.

Installed as `/etc/systemd/system/llama-server.service`. The same unit, with `alex` replaced by `USER`, is [`../llama-server.service`](../llama-server.service):

```ini
[Unit]
Description=llama.cpp server - Ornith-1.5-122B IQ2_S (Vulkan, 128K)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=USER
Group=USER
SupplementaryGroups=render video
WorkingDirectory=/home/USER/llama.cpp
ExecStart=/home/USER/llama.cpp/build/bin/llama-server \
  -m /home/USER/models/Ornith-1.5-122B-AD-IQ2_S-IQ2_XS.gguf \
  -c 131072 -ngl 99 -fa on -lm none \
  -ctk q8_0 -ctv q8_0 \
  -b 2048 -ub 512 --parallel 1 --cache-reuse 256 \
  --jinja --temp 0.6 --top-p 0.95 --top-k 20 --min-p 0 \
  --metrics --host 0.0.0.0 --port 8080 \
  --spec-type draft-mtp --spec-draft-n-max 2
Restart=on-failure
RestartSec=10
TimeoutStartSec=600
LimitMEMLOCK=infinity

[Install]
WantedBy=multi-user.target
```

`SupplementaryGroups=render video` is what gives the service access to the Vulkan device. On this laptop `USER` was `alex`, the working directory was `/home/alex/llama.cpp`, and the model path was `/home/alex/models/Ornith-1.5-122B-AD-IQ2_S-IQ2_XS.gguf`.

From this folder, install the unit, replace every `USER` in the installed file, then start it:

```
sudo cp llama-server.service /etc/systemd/system/llama-server.service
# Replace every USER in /etc/systemd/system/llama-server.service, then:
sudo systemctl daemon-reload && sudo systemctl enable --now llama-server
journalctl -u llama-server -f
```

`TimeoutStartSec=600` is the start budget in the unit. A separate timing of the cold start was not recorded.

## Benchmark

`llama-bench` on build `e0f142d5d` (12005), Vulkan RADV, no speculative decoding. The model line reported 39.86 GiB. Flags that were recorded: `-ngl 99 -fa 1 -ctk q8_0 -ctv q8_0 -lm none`. The four test names are `llama-bench`'s `pp512` and `tg128` at KV depth 0 and 32768. The repetition count and the bench batch sizes were not recorded. A full raw table was not pasted.

```
./build/bin/llama-bench \
  -m /home/USER/models/Ornith-1.5-122B-AD-IQ2_S-IQ2_XS.gguf \
  -ngl 99 -fa 1 -ctk q8_0 -ctv q8_0 -lm none \
  -p 512 -n 128 -d 0,32768
```

`-p 512 -n 128 -d 0,32768` is the command that produces the recorded test names. `-r` is omitted because it was not written down. Add `-d 131072` when the 128K row below is filled in. Stop the server first if it has the model loaded.

| Test | t/s |
| --- | --- |
| pp512 | 472.35 ± 2.50 |
| tg128 | 26.19 ± 0.05 |
| pp512 @ d32768 | 274.15 ± 1.09 |
| tg128 @ d32768 | 24.65 ± 0.20 |
| pp512 @ d131072 | TODO |
| tg128 @ d131072 | TODO |

| Date | Method | Prompt tokens | Gen tokens | PP tok/s | TG tok/s | Memory | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2026-10-30 | `llama-bench`, depth 0, no MTP | 512 | 128 | 472.35 ± 2.50 | 26.19 ± 0.05 | not measured | `-lm none`, q8_0 KV |
| 2026-10-30 | `llama-bench`, depth 32768, no MTP | 512 | 128 | 274.15 ± 1.09 | 24.65 ± 0.20 | not measured | `-lm none`, q8_0 KV |
| TODO | `llama-bench`, depth 131072, no MTP | 512 | 128 | TODO | TODO | TODO | server loads 131072; this depth was not benched |
| 2026-10-30 | one `llama-server` chat, MTP `--spec-draft-n-max 2` | 355 | 1382 | ~190.8 | 32.15 | not measured | single request; see below |

- **Fit:** the bench requested `-ngl 99` on Vulkan. `--list-devices` reported one device, `AMD Radeon 8060S Graphics (RADV STRIX_HALO)`, heap 59904 MiB, 59283 MiB free while unloaded. `llama-bench` printed the model size as 39.86 GiB. Resident memory during the bench was not recorded.
- **Memory tool:** none for the run. The heap figures are the device line above.

### MTP, one chat request

These rates are a single `llama-server` request with `--spec-type draft-mtp --spec-draft-n-max 2`. They are separate from the `llama-bench` rows. `llama-bench` does not run MTP drafting.

Recorded for that request: generation 32.15 t/s, 1,382 tokens, 42 s; prompt processing about 190.8 t/s, 355 tokens, 1.9 s. This page reports those figures as written down.

The operator compared that generation rate with the no-MTP `tg128` of 26.19 t/s and called it about 23% faster:

```
(32.15 - 26.19) / 26.19 = 22.8%
```

The 32.15 t/s figure is one chat at a 355-token prompt. The 26.19 t/s figure is `llama-bench` `tg128` at depth 0. The comparison is those two recorded rates.

Sanity check on the same setup: the model answered the car-wash question correctly (a car wash is 20 m away; walk or drive). The reply text was not saved.

## Client

- **Client:** [Hermes Agent](https://github.com/NousResearch/hermes-agent) (OpenAI-compatible)
- **Base URL:** `http://<host>:8080/v1`. On the laptop itself, `<host>` is `127.0.0.1`.
- **Tool calling / chat template:** `--jinja` is on, so the server applies the chat template in the GGUF. Tool-calling behavior was not recorded.

Confirm the process is up, then send one chat completion. The unit sets sampling, so the body does not repeat it. No `--alias` was set; take `id` from `/v1/models` if the client requires a model name.

```
curl -sS http://127.0.0.1:8080/v1/models

curl -sS http://127.0.0.1:8080/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{
    "messages": [
      {"role": "user", "content": "A car wash is 20 m away. Should I walk or drive?"}
    ]
  }'
```

Point Hermes Agent at the same base URL, `http://<host>:8080/v1`.

## Notes

- The 128K-depth `llama-bench` rows (`pp512` and `tg128` at `-d 131072`) are TODO. The server already loads `-c 131072`. No number for that depth was measured.
- `-lm none` is load mode. On this llama.cpp build it replaces the older `-mmp` / mmap flags.
- `sudo mkinitcpio -P` does not regenerate the boot image on this Omarchy install (`No presets found`). Use `sudo limine-update`.
- ROCm was not used for this run.

Other AtomicChat quants of Ornith-1.5-122B in the same size band, for reference. None of these were run on this laptop. Byte sizes are the Hugging Face LFS sizes. GB is bytes / 1e9, shown to two decimals. `AD-IQ3_S-IQ3_XXS` is published as two shards; the GB figure recorded with this setup for that quant was 54.28, and the shards sum to 54.29 GB at two decimals. The other four match the GB figures recorded here.

| Quant | File | Bytes | GB |
| --- | --- | --- | --- |
| AD-IQ2_XXS-IQ1_M | `Ornith-1.5-122B-AD-IQ2_XXS-IQ1_M.gguf` | 34,959,499,328 | 34.96 |
| AD-IQ2_XS-IQ2_XXS | `Ornith-1.5-122B-AD-IQ2_XS-IQ2_XXS.gguf` | 39,187,357,760 | 39.19 |
| AD-IQ2_S-IQ2_XS | `Ornith-1.5-122B-AD-IQ2_S-IQ2_XS.gguf` | 42,811,236,416 | 42.81 |
| AD-IQ3_XXS-IQ2_S | `Ornith-1.5-122B-AD-IQ3_XXS-IQ2_S.gguf` | 47,643,074,624 | 47.64 |
| AD-IQ3_S-IQ3_XXS | `Ornith-1.5-122B-AD-IQ3_S-IQ3_XXS-00001-of-00002.gguf` plus `-00002-of-00002` | 54,286,852,352 | 54.29 |

The first shard of `AD-IQ3_S-IQ3_XXS` is 44,741,990,560 bytes and the second is 9,544,861,792 bytes.
