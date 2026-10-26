# local-models

Public, reproducible setups for running local LLMs on specific GPUs and single-board computers. Each top-level folder is one machine. The notes record which models and quantizations fit, how the runtime was built, how the server was launched, and the prompt-processing and generation speeds that were actually measured on that hardware.

The intended workload is agentic: tool calling, multi-turn sessions, and long context, served through an OpenAI-compatible HTTP endpoint. [Hermes Agent](https://github.com/NousResearch/hermes-agent) (Nous Research) is the client these setups are exercised with most often. Any other client that speaks the same API can use the same server.

Model weights stay out of git. Entries link to the file that was run.

## Who this is for

Developers who already have one of these devices and want a tested starting point, rather than a generic "recommended quant" that was timed on different hardware.

The numbers that matter here are the ones that decide whether a model is usable as an agent:

- Does this quantization fit, and at what context length?
- How fast is prompt processing on a long, tool-heavy transcript?
- How fast is token generation once the prompt is ingested?
- How much memory did that configuration actually use?

A result from a different GPU, a different context length, or a different KV-cache type does not answer those questions. Use the folder that matches your hardware, and treat the recorded command as part of the result.

## Find your hardware

Folder names are kebab-case: vendor, product, and the memory size or SKU that bounds which models fit.

| Folder | Hardware | Memory | OS | Backends |
| --- | --- | --- | --- | --- |
| [`amd-rx7900xtx-24gb`](amd-rx7900xtx-24gb/README.md) | AMD Radeon RX 7900 XTX on a desktop | 24GB GDDR6, 64GB system RAM | Fedora | ROCm/HIP, Vulkan |
| [`nvidia-agx-orin-64gb`](nvidia-agx-orin-64gb/README.md) | NVIDIA Jetson AGX Orin, dedicated model host | 64GB unified | L4T R36.4.7 | CUDA (`sm_87`) |
| [`amd-strix-halo-392`](amd-strix-halo-392/README.md) | Lenovo Legion 7a, Ryzen AI Max+ 392, Radeon 8060S | 64GB unified LPDDR5X | Omarchy Linux | Vulkan, ROCm (`gfx1151`) |
| [`nvidia-rtx4090-24gb`](nvidia-rtx4090-24gb/README.md) | NVIDIA GeForce RTX 4090 | 24GB GDDR6X, 64GB system RAM | Ubuntu | CUDA (`sm_89`) |

Open the hardware README and match the OS, driver stack, and backend before trusting a number. Two machines with the same GPU and different system RAM, drivers, or offload settings will not match.

Two folders have measurements from 2026-10-26: [Ornith-1.5-35B-A3B Q4_K_M](nvidia-agx-orin-64gb/models/ornith-1.5-35b-a3b-q4_k_m.md) on the AGX Orin, and [Qwen3.8-27B Q4_K_M](amd-rx7900xtx-24gb/models/qwen3.8-27b-q4_k_m.md) on the RX 7900 XTX (two ROCm builds). The Strix Halo and RTX 4090 folders have no entries yet.

## Reproduce a setup

1. Open the folder for your device and pick a model entry (or the hardware README, while a folder still has no entries).
2. Match the recorded OS family, GPU driver, and backend (CUDA, ROCm/HIP, or Vulkan). If you have to diverge, write down the delta next to your own numbers.
3. Check out the recorded runtime commit and rebuild with the recorded flags and GPU architecture (`sm_XX` or `gfxXXXX`).
4. Download the same model file from the recorded source. Same publisher, same quantization, same filename or revision.
5. Start the server with the recorded command: context length, GPU offload, KV-cache type, flash attention, and batch sizes.
6. Run the recorded benchmark command. Compare prompt-processing tok/s, generation tok/s, memory, and the prompt length used for the measurement.

## Repository layout

```text
README.md                  # this file: purpose, conventions, contributing
TEMPLATE.md                # copy-paste form for one model + quant + runtime
<hardware-folder>/
  README.md                # the machine, plus an index of what has been tried
  models/
    <model>-<quant>.md     # one filled-in copy of TEMPLATE.md
```

Server unit files, launch scripts, and pasted benchmark logs can live next to the hardware README once they exist. Link them from that README. Keep secrets out of committed units (tokens, Tailscale auth keys, private hostnames you do not want published).

Suggested hardware-folder names stay specific enough to search for: `amd-rx7900xtx-24gb`, `nvidia-agx-orin-64gb`, `amd-strix-halo-392`, `nvidia-rtx4090-24gb`. A new device gets a new folder. A different memory configuration of the same GPU gets its own folder when the difference changes which models fit.

## What a hardware README contains

Each hardware README describes the machine, then keeps these sections, even while they are empty:

- **Hardware.** GPU or SoC, VRAM or unified memory, system RAM, OS, backend, and the role of the machine (desktop, laptop, dedicated model host).
- **Models and quants tried.** An index of entries under `models/`. One row per model, quantization, and runtime. No row until someone has run it.
- **llama.cpp build.** Commit, CMake (or other) flags, GPU architecture, compiler, and the ROCm, CUDA, or Vulkan version used to compile.
- **Server config.** The `llama-server` launch command and any systemd unit: bind address, port, context, and the flags that change fit and speed.
- **Benchmarks.** Measured prompt-processing and generation tok/s, the command that produced them, context length, and memory use.
- **Notes and gotchas.** Crashes, kernel or driver pins, environment variables, thermals, and anything else required to reproduce the run.
- **Other runtimes.** The same kind of notes for vLLM or another server, once someone has actually run it on that machine.

Leave a section empty until there is a real result. Do not copy numbers in from another folder or from a vendor table.

## Conventions

Record a result so someone else can rerun it and compare. One entry is one model, one quantization, one runtime build, and one server configuration. A different quant, a different KV-cache type, or a newer llama.cpp commit is a new entry or a new dated row. Keep the old row.

### Model

- **Name.** The model id people search for, including parameter count (`Qwen2.5-32B-Instruct`, `Llama-3.3-70B-Instruct`).
- **Source link.** The Hugging Face repo, direct file URL, or other page you downloaded from.
- **Quantization.** The method and variant you ran (`Q4_K_M`, `Q8_0`, `IQ4_XS`, `FP16`, AWQ, GPTQ, FP8), plus the quant publisher if they are not the original lab.
- **File identity.** Exact GGUF filename, repo revision, or checksum when the file is published. Disk size is worth recording so a partial download is obvious.

### Runtime build

- **Runtime.** `llama.cpp`, `vLLM`, or whatever actually served the port.
- **Commit or version.** Full git SHA for llama.cpp (or the image tag / package version for another runtime), and the date of that commit if you have it.
- **Backend.** CUDA, ROCm/HIP, Vulkan, or CPU, including the toolkit version (CUDA, ROCm, Vulkan SDK / Mesa).
- **GPU architecture.** The target you compiled for, such as `sm_89`, `sm_87`, `gfx1100`, or `gfx1151`.
- **Build flags.** The CMake invocation or equivalent, plus any environment variables required at build or run time (`AMDGPU_TARGETS`, `CMAKE_CUDA_ARCHITECTURES`, `HSA_OVERRIDE_GFX_VERSION`, and similar).

### Server configuration

- **Launch command.** The full command, copied as it was run. For llama.cpp that is usually `llama-server` with `-m`, `-c`, `-ngl`, batch (`-b`), micro-batch (`-ub`), flash attention, KV-cache type (`-ctk` / `-ctv` or `--cache-type-k` / `--cache-type-v`), threads, and `--host` / `--port`.
- **Context length loaded.** The `-c` / `max_model_len` the server was started with. Also note the model's advertised maximum if you loaded less than that.
- **KV cache.** Type per key and value (`f16`, `q8_0`, `q4_0`, …). This changes both the context that fits and the speed.
- **Offload.** GPU layers, split mode, and whether the model was fully on GPU, partially offloaded, or CPU-only.
- **systemd.** The unit contents, or a path to the unit file in the hardware folder, when the machine starts the server that way.

### Measurements

Date the run (`YYYY-MM-DD`). Report prompt-processing and generation separately.

| Field | What to write |
| --- | --- |
| Method | The tool and command (`llama-bench -p 512 -n 128`, server timings, client-side). |
| Prompt tokens | Tokens ingested for the prompt-processing figure (often `pp512`, but record the real length). |
| Generation tokens | Tokens generated for the generation figure (often `tg128`). |
| PP tok/s | Prompt-processing tokens per second. |
| TG tok/s | Generation tokens per second. |
| Memory | VRAM, unified memory, or system RAM, and the tool (`nvidia-smi`, `rocm-smi`, `tegrastats`, `free`). |
| Fit | Fully on GPU, partial offload, or CPU. Note OOM or swap if that is how the run ended. |

For agentic use, add at least one prompt-processing figure at a long context you actually care about (for example several thousand to tens of thousands of tokens), not only a 512-token sample. State whether the figure is a single stream. Parallel slots change the result; record `-np` / `--parallel` when it is not 1.

Paste the command and the tool's raw summary in the entry when you have them. Write `not measured` for a field you skipped. A blank cell is easy to misread as zero.

### Client notes

Name the client (Hermes Agent, or another OpenAI-compatible client), the base URL it used, and whether tool calling and the chat template behaved. Server flags such as `--jinja` belong in the launch command.

## Contributing

Open a pull request. Results land in the hardware folder they were measured on.

1. Copy [`TEMPLATE.md`](TEMPLATE.md) to `<hardware-folder>/models/<model>-<quant>.md`. Use lowercase kebab-case, and include the quant in the filename so two files for the same model do not collide.
2. Fill in the fields in [Conventions](#conventions). Those details are what make two runs comparable: model source, quant, loaded context, runtime commit, build flags, server command, date, prompt-processing tok/s, generation tok/s, and memory.
3. Add a row to the models table in that folder's `README.md` and link the entry. If you built llama.cpp or wrote a unit file, update those sections (or link the new file) in the same README.
4. New device: add a kebab-case folder and a `README.md` with the sections listed above. Describe the hardware you ran on. Leave benchmark tables empty until you have a measurement.
5. A newer driver or a newer commit is a new dated row. Leave the previous measurement in place.

Measure on the named hardware. Do not transpose tok/s from another GPU, from a cloud instance, or from a model's model card. If a run failed, that is still useful: record the command, the error, and the memory high-water mark.

## Template

Copy [`TEMPLATE.md`](TEMPLATE.md) for a per-model entry. The same skeleton is below.

~~~markdown
# <Model name> <quant>

- **Date:** YYYY-MM-DD
- **Hardware:** <hardware-folder>
- **Runtime:** llama.cpp | vLLM | <other>
- **Backend:** CUDA | ROCm/HIP | Vulkan | CPU

## Model

- **Name:**
- **Source:** <https://…>
- **Quant:** <Q4_K_M | Q8_0 | IQ4_XS | AWQ | GPTQ | FP16 | …>
- **Quant publisher:** <lab or repo, if different from the source>
- **File:** <exact filename, revision, or checksum>
- **Disk size:**

## Build

- **Commit / version:** <full git SHA or package version>
- **GPU arch:** <sm_XX or gfxXXXX>
- **Toolkit:** <CUDA / ROCm / Mesa version>
- **OS / kernel / driver:**
- **Configure command:**

```
<cmake or other build command, with the flags you used>
```

## Server

- **Context loaded:** <-c / max_model_len>
- **Model max context:**
- **KV cache:** <k type> / <v type>
- **Offload:** <fully on GPU | N layers | CPU>
- **Endpoint:** <http://host:port/v1>

```
<full launch command, or the systemd unit>
```

## Benchmark

| Date | Method | Prompt tokens | Gen tokens | PP tok/s | TG tok/s | Memory | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| YYYY-MM-DD | <command> | | | | | | |

- **Fit:**
- **Raw output:**

```
<paste llama-bench or server summary here>
```

## Client

- **Client:** <Hermes Agent or other>
- **Tool calling / chat template:**

## Notes
~~~
