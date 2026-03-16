**Serving Qwen3‑Next‑80B‑A3B‑Instruct (FP8) on Windows 11 WSL2 with vLLM and FlashInfer — A Real‑World Tale**

*Score: 86 upvotes (Reddit r/LocalLLaMA, community post)*  

---

### TL;DR
To run **Qwen3‑Next‑80B‑A3B‑Instruct FP8** on a **Blackwell** GPU under **WSL2**, use **PyTorch 2.8.0 (cu128)**, **vLLM 0.10.2**, and **FlashInfer ≥ 0.3.1** (preferably 0.3.1). Pin the nightly **cu128** vLLM image, expose `/dev/dxg` and `/usr/lib/wsl/lib`, and run a small `run.sh` that installs the precise userspace stack and starts the OpenAI‑compatible server. This configuration reaches **80 tokens per second** on a single stream, leveraging both FlashInfer and CUDA graphs — a setup that contradicts common advice from some AI assistants.

---

### Introduction

The Reddit post from r/LocalLLaMA details a successful deployment of the massive **Qwen3‑Next‑80B‑A3B‑Instruct** model in **FP8 format** on a **Blackwell‑based RTX PRO 6000** (96 GB VRAM) via **WSL2**, **vLLM**, and **FlashInfer**. The author, a member of the community, shares a step‑by‑step Docker configuration that resolves several known pitfalls, including the “FlashInfer requires sm75+” crash, and achieves **80 tok/s** throughput.

Below is a technical blog article summarizing the post’s claims and instructions, presented in its own words and with attention to the numbers and terminology used.

---

### Community‑Reported Requirements

According to the post, the following stack must be pinned to avoid crashes and ensure compatibility:

- **PyTorch 2.8.0 (cu128)** – essential for cu128 support on WSL2
- **vLLM 0.10.2** – nightly cu128 container
- **FlashInfer ≥ 0.3.0** – version **0.3.1** is recommended to bypass the sm75 requirement
- **Transformers (main)** – kept up to date

The post stresses that **FlashInfer must be upgraded** because the default version triggers a crash on Blackwell GPUs. With FlashInfer 0.3.1, the system enables **CUDA graphs** and delivers **80 tokens per second** in a single stream.

---

### Hardware and Software Environment

| Component | Specification |
|----------|----------------|
| OS | Windows 11 + WSL2 (Ubuntu) |
| GPU | RTX PRO 6000 **Blackwell** (96 GB) |
| Serving engine | **vLLM** OpenAI‑compatible server |
| Model | `TheClusterDev/Qwen3-Next-80B-A3B-Instruct-FP8-Dynamic` (80 B total, ~3 B activated per token) |

Key points from the post:

- Despite the **MoE (Mixture of Experts)** nature with only ~3 B activated weights per token, **the full 80 B weights must fit in VRAM** because FP8 does not fully reduce the memory footprint.
- The model requires **~75 GiB** of VRAM on the Blackwell card.
- The **dynamic version** of the model is mandatory for vLLM compatibility.

---

### Docker Command – Community‑Tested

The author shares a Docker run command that incorporates all required parameters:

```bash
docker run --rm --name vllm-qwen \
  --gpus all \
  --ipc=host \
  -p 8000:8000 \
  --entrypoint bash \
  --device /dev/dxg \
  -v /usr/lib/wsl/lib:/usr/lib/wsl/lib:ro \
  -e LD_LIBRARY_PATH="/usr/lib/wsl/lib:$LD_LIBRARY_PATH" \
  -e HUGGING_FACE_HUB_TOKEN="$HF_TOKEN" \
  -e HF_TOKEN="$HF_TOKEN" \
  -e VLLM_ATTENTION_BACKEND=FLASHINFER \
  -v "$HOME/.cache/huggingface:/root/.cache/huggingface" \
  -v "$HOME/.cache/torch:/root/.cache/torch" \
  -v "$HOME/.triton:/root/.triton" \
  -v /data/models/qwen3_next_fp8:/models \
  -v "$PWD/run-vllm-qwen.sh:/run.sh:ro" \
  lmcache/vllm-openai:latest-nightly-cu128 \
  -lc '/run.sh'
```

#### Why Each Flag Matters

- **`--device /dev/dxg`** – Expose the WSL GPU device node so the container can access the NVIDIA driver.
- **`-v /usr/lib/wsl/lib:/usr/lib/wsl/lib:ro`** – Mount the WSL CUDA stub directory, providing `libcuda.so.1` for PyTorch’s `dlopen`.
- **`-e LD_LIBRARY_PATH=...`** – Prepend the WSL library path so the container finds `libcuda.so.1`.
- **`-p 8000:8000`** – Bind the OpenAI-compatible API port.
- **`--entrypoint bash -lc '/run.sh'`** – Executes the custom script that installs dependencies and starts vLLM.

---

### The `run.sh` Script

While the full script is truncated in the original post, its purpose is clear: it installs **PyTorch 2.8.0 (cu128)**, **FlashInfer 0.3.1**, and any other runtime dependencies, then launches vLLM with the correct arguments. The script avoids relying on pre‑installed system packages that may differ across WSL2 installations.

---

### Performance and Observations

With the above configuration, the author reports:

- **80 tokens per second** throughput on a single stream.
- Successful use of **CUDA graphs** and **FlashInfer**, contradicting advice from some AI assistants that recommend **disabling** these features.
- A clear warning: **do not rely on Claude or ChatGPT** for guidance on this stack, as they may suggest suboptimal or incompatible settings.

---

### Summary

This community‑contributed guide demonstrates that, with precise version pinning and careful WSL2 GPU exposure, it is possible to serve a massive **80 B FP8 MoE model** on a **Blackwell** GPU via **vLLM** and **FlashInfer**. The key steps are:

1. Use **PyTorch 2.8.0 (cu128)** and **vLLM nightly cu128** image.
2. Install **FlashInfer 0.3.1** or later.
3. Mount `/dev/dxg` and `/usr/lib/wsl/lib` to expose the WSL CUDA environment.
4. Run a dedicated `run.sh` to ensure consistent dependencies.
5. Expect **~75 GiB** VRAM usage despite FP8 compression.

The post, with its **86 upvotes**, offers one of the most detailed, working configurations available for this niche deployment.

---

> **Disclaimer**: This article summarizes unverified community information from a Reddit post (r/LocalLLaMA). The details are presented as‑is and may require further validation for production use.
