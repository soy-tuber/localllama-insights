**RTX Pro 6000 Blackwell vLLM Benchmark: 120B Model Performance Analysis**  
*Unverified Community Insight – Reddit Score: 173 Upvotes*

This post from the *r/LocalLLaMA* community presents a detailed performance profile of a 120-billion-parameter model running on NVIDIA’s RTX Pro 6000 Blackwell workstation, using vLLM 0.11.0. With 96GB of VRAM, the system demonstrates compelling results for large-scale local inference. While unconfirmed by official benchmarks, the data comes from a reputed local AI enthusiast and carries 89 comments discussing its practical implications — suggesting a high level of community engagement and perceived credibility.

---

### Hardware & Software Setup

- **GPU:** NVIDIA RTX Pro 6000 Blackwell Workstation Edition (96GB VRAM)  
- **CUDA:** 13.0  
- **Driver:** 580.82.09  
- **Framework:** vLLM 0.11.0  
- **Model:** `openai/gpt-oss-120b` (120B parameters)  
- **Source:** [Hugging Face – gpt-oss-120b](https://huggingface.co/openai/gpt-oss-120b)

The benchmark was executed using the [vllm-benchmark-suite](https://github.com/notaDestroyer/vllm-benchmark-suite), a community-driven tool for evaluating LLM inference performance under varying workloads.

---

### Test Scenarios

Two primary test profiles were evaluated:

1. **Short Output (500 tokens)**  
   - Context lengths: 1K–128K  
   - Concurrency: 1–20 users  

2. **Extended Output (1000–2000 tokens)**  
   - Same context and concurrency ranges  

Performance was measured in tokens per second (tok/s), with attention to throughput, latency, power consumption, and batch efficiency.

---

### Key Findings

**Peak Performance (500-token output):**

- **1051 tok/s** achieved at 1 user with 1K context  
- **300–476 tok/s** sustained across 20 concurrent users, regardless of context length  
- **TTFT (Time to First Token):** 200–400ms at low concurrency; increases to **2000–3000ms** at 20 users  
- **Average latency:** 2.6 seconds (1 user) → **30.2 seconds** (20 users) at 128K context  

These results indicate strong linear scaling up to moderate concurrency, with predictable latency growth under load.

**Extended Output (1000–2000 tokens):**

- **1016 tok/s** peak throughput — only a minor (~3%) drop from 500-token output  
- Higher latencies due to longer decoding phases  
- **Power draw:** 300–600W depending on system load  
- **Batch scaling efficiency:** Excellent at 2–5 users; remains usable up to 10 users  

The model demonstrates stable performance even under extended token generation, with minimal efficiency loss.

---

### Observations on Blackwell Architecture

- **Linear scaling** observed up to approximately **5 concurrent users**  
- **GPU clock stability:** Maintained at 2800+ MHz under sustained load  
- **Inter-token latency:** Remains sub-50ms ("INSTANT") for most configurations  
- **Context length behavior:** Throughput roughly halves with every **32K increase** in context length  

The 96GB VRAM capacity proves sufficient to avoid memory swapping, even at maximum context (128K) and concurrency.

---

### TL;DR

For users running 100B+ parameter models locally, the **RTX Pro 6000 Blackwell** delivers **production-grade throughput** with **excellent multi-user scalability**. While power consumption is moderate (300–600W), the system’s compute density and VRAM headroom make it a compelling choice for enterprise-grade local inference workloads — especially when paired with vLLM’s efficient attention implementation.

---

*Note: This analysis is based on unverified community data from r/LocalLLaMA (173 upvotes, 89 comments). Always validate performance with your own hardware and software stack.*
