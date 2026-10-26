# <Model name> <quant>

One measured configuration. Copy this file to `<hardware-folder>/models/<model>-<quant>.md` and replace every placeholder. Leave a field as `not measured` when you did not record it. See the conventions in [README.md](README.md) for what makes two runs comparable.

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
- **Parallel slots:** <1, or the value you set>
- **Endpoint:** <http://host:port/v1>

```
<full launch command>
```

systemd unit, if the host starts the server this way:

```ini
# <hardware-folder>/llama-server.service
# Replace every placeholder. Do not commit tokens or private keys.

[Service]
ExecStart=<llama-server or other binary> -m <model-path> -c <context> -ngl <layers> --host <bind> --port <port>
```

## Benchmark

| Date | Method | Prompt tokens | Gen tokens | PP tok/s | TG tok/s | Memory | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| YYYY-MM-DD | <command> | | | | | | |

- **Fit:** <fully on GPU | partial offload | CPU | OOM>
- **Memory tool:** <nvidia-smi | rocm-smi | tegrastats | free | …>
- **Raw output:**

```
<paste llama-bench or server summary here>
```

## Client

- **Client:** <Hermes Agent or another OpenAI-compatible client>
- **Base URL:**
- **Tool calling / chat template:** <what worked, what failed>

## Notes

<Driver pins, environment variables, thermals, crashes, and anything else required to reproduce the run.>
