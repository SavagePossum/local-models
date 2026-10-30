# AMD Strix Halo (Ryzen AI Max+ 392)

Lenovo Legion 7a laptop with an AMD Ryzen AI Max+ 392 (Strix Halo) and a Radeon 8060S iGPU (40 compute units). The GPU target is `gfx1151`. Memory is 64GB unified LPDDR5X, shared by the CPU and the iGPU. The OS is Omarchy Linux (Arch-based) with the Limine bootloader. Setups on this page target Vulkan or ROCm.

One model is recorded: [Ornith-1.5-122B AD-IQ2_S-IQ2_XS](models/ornith-1.5-122b-ad-iq2_s-iq2_xs.md), Vulkan, llama.cpp `e0f142d5d` (12005), on 2026-10-30. `llama-bench` without MTP: prompt processing 472.35 / 274.15 t/s and generation 26.19 / 24.65 t/s at KV depth 0 / 32K. 128K depth is TODO. The server runs at context 131072 with MTP speculative decoding, for long-running work (research, refactoring, deep code review). Interactive use stays on Ornith-1.5-35B-A3B; the recorded 35B setup is the [AGX Orin entry](../nvidia-agx-orin-64gb/models/ornith-1.5-35b-a3b-q4_k_m.md).

## Hardware

| | |
| --- | --- |
| Machine | Lenovo Legion 7a |
| APU | AMD Ryzen AI Max+ 392 (Strix Halo) |
| iGPU | AMD Radeon 8060S, 40 CU |
| ROCm / LLVM target | `gfx1151` |
| Memory | 64GB unified LPDDR5X |
| OS | Omarchy Linux (Arch-based) |
| Bootloader | Limine |
| Backends | Vulkan, ROCm |
| Role | Laptop |

The Ornith run below is Vulkan (RADV). A ROCm run is not recorded yet.

## Suggested layout

```text
amd-strix-halo-392/
  README.md                 # this file
  models/<model>-<quant>.md # one copy of TEMPLATE.md per model and quant
  llama.cpp.md              # optional: extra build notes, once they outgrow this file
  llama-server.service      # systemd unit (placeholders; see the model entry)
  benchmarks/<run>.md       # optional pasted llama-bench output
```

## Models and quants

Unified memory is the fit limit. Record which backend (Vulkan or ROCm) produced each row; the two are not interchangeable.

The iGPU heap stays small until the UMA frame buffer is minimized and the TTM page limit is raised. After that change, Vulkan reported `Vulkan0: AMD Radeon 8060S Graphics (RADV STRIX_HALO) (59904 MiB, 59283 MiB free)`. The procedure is in the [model entry](models/ornith-1.5-122b-ad-iq2_s-iq2_xs.md) and under [Notes and gotchas](#notes-and-gotchas).

| Model | Quant | Context | Backend | PP tok/s | TG tok/s | Memory | Date | Entry |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Ornith-1.5-122B | AD-IQ2_S-IQ2_XS | 131072 server; bench at 0 and 32K | Vulkan `e0f142d5d` (12005) | 472.35 / 274.15 | 26.19 / 24.65 | 39.86 GiB model; Vulkan heap 59904 MiB | 2026-10-30 | [entry](models/ornith-1.5-122b-ad-iq2_s-iq2_xs.md) |

PP is `pp512` and TG is `tg128` at KV depth 0 / 32K, from `llama-bench`, with no MTP. The ± values are in the model entry. 128K depth (`-d 131072`) is TODO even though the server loads 131072. The memory column is the model size `llama-bench` printed and the unloaded Vulkan heap, not a resident reading.

One MTP chat on the server (not `llama-bench`) recorded 32.15 t/s generation and about 190.8 t/s prompt processing. Details, including the 23% comparison against the no-MTP `tg128`, are in the [model entry](models/ornith-1.5-122b-ad-iq2_s-iq2_xs.md).

## llama.cpp build

Recorded for the Ornith server.

- **Commit:** [`e0f142d5ddc1461ce721644d92af33db23545105`](https://github.com/ggml-org/llama.cpp/commit/e0f142d5ddc1461ce721644d92af33db23545105) (2026-10-30 04:30:05 +0000), build 12005. Short hash `e0f142d5d`.
- **GPU arch:** `gfx1151` (RADV `STRIX_HALO`). CMake does not pass an architecture flag.
- **Toolkit:** Vulkan, Arch `vulkan-radeon` (RADV). Mesa version not recorded.
- **OS:** Omarchy Linux, Limine. Kernel not recorded.

Packages:

```
sudo pacman -S --needed base-devel git cmake vulkan-radeon vulkan-icd-loader vulkan-headers vulkan-tools shaderc spirv-headers ccache python-pipx
```

`spirv-headers` is required. If CMake already failed without it, `rm -rf build` and configure again.

```
cmake -B build -DGGML_VULKAN=ON -DCMAKE_BUILD_TYPE=Release && cmake --build build --config Release -j
```

Check the device from the llama.cpp tree:

```
./build/bin/llama-server --list-devices
```

On this build, `-lm none` (load mode) replaces the older `-mmp` / mmap flags.

## Server config

[`llama-server.service`](llama-server.service) is the unit installed at `/etc/systemd/system/llama-server.service`. It serves Ornith-1.5-122B AD-IQ2_S-IQ2_XS on `0.0.0.0:8080` with context 131072, q8_0 KV cache, flash attention, `-lm none`, and MTP (`--spec-type draft-mtp --spec-draft-n-max 2`).

The committed unit says `USER` and `/home/USER/...`. On the laptop that was `alex`, `/home/alex/llama.cpp`, and `/home/alex/models/Ornith-1.5-122B-AD-IQ2_S-IQ2_XS.gguf`. `SupplementaryGroups=render video` is how the service opens the Vulkan device. From this folder, copy the unit, replace every `USER` in the installed file, then:

```
sudo cp llama-server.service /etc/systemd/system/llama-server.service
# Replace every USER in /etc/systemd/system/llama-server.service, then:
sudo systemctl daemon-reload && sudo systemctl enable --now llama-server
journalctl -u llama-server -f
```

Hermes Agent uses the OpenAI-compatible base URL `http://<host>:8080/v1`. A `curl` against `/v1/chat/completions` is in the [model entry](models/ornith-1.5-122b-ad-iq2_s-iq2_xs.md).

## Benchmarks

`llama-bench` on 2026-10-30, build `e0f142d5d` (12005), Vulkan RADV, no MTP. Depth 0 / 32K: prompt processing 472.35 ± 2.50 / 274.15 ± 1.09 t/s, generation 26.19 ± 0.05 / 24.65 ± 0.20 t/s (`pp512` and `tg128`). Flags: `-ngl 99 -fa 1 -ctk q8_0 -ctv q8_0 -lm none`. 128K depth is TODO.

One real chat with MTP (`--spec-draft-n-max 2`) measured 32.15 t/s generation (1,382 tokens, 42 s) and about 190.8 t/s prompt processing (355 tokens, 1.9 s). That is a single-request observation, separate from `llama-bench`. The model answered the car-wash question correctly (a car wash 20 m away; walk or drive). The command, both tables, and the comparison arithmetic are in the [model entry](models/ornith-1.5-122b-ad-iq2_s-iq2_xs.md).

## Notes and gotchas

- BIOS: set the UMA frame buffer to the minimum. The menu label and the resulting size were not written down.
- GTT/TTM limit. Create `/etc/modprobe.d/ttm.conf`:

```
options ttm pages_limit=15204352 page_pool_size=15204352
```

`15204352` × 4 KiB = 58 GiB. Then `sudo limine-update` and reboot. `sudo mkinitcpio -P` fails on this Omarchy install with `No presets found`. Fallback: add `ttm.pages_limit=15204352 ttm.page_pool_size=15204352` to `KERNEL_CMDLINE[default]` in `/etc/default/limine`, run `sudo limine-update`, and reboot. The modprobe file is the path that was used. After reboot the Vulkan heap was 59904 MiB, 59283 MiB free.

- `spirv-headers` has to be installed before the Vulkan CMake configure. A failed configure needs `rm -rf build` before the next one.
- This 122B server is the long-running job. Ornith-1.5-35B-A3B stays the interactive model. Other AtomicChat quant sizes for the 122B are listed in the [model entry](models/ornith-1.5-122b-ad-iq2_s-iq2_xs.md). Only AD-IQ2_S-IQ2_XS was run.

## Other runtimes

No vLLM or other runtime has been recorded on this laptop yet. Use the same sections (build, server command, benchmark table) when one is added.
