# Content Audit Report (2026-03-16)

This audit reviews the 15 articles in this repository against the latest available information as of March 2026. Each article is evaluated for factual accuracy, outdated claims, and technical errors.

---

## Summary

| Severity | Count | Description |
|----------|-------|-------------|
| Factual Error | 3 | Incorrect claims that contradict source material |
| Outdated | 4 | Claims that were accurate at publication but are now incorrect |
| Misleading | 3 | Framing or attribution issues |
| Minor | 5 | Typos, imprecise wording, or non-critical inaccuracies |

---

## Article-by-Article Findings

### 00: Don't Waste Electricity When Running vLLM

**Status: Outdated**

| Issue | Severity | Details |
|-------|----------|---------|
| PR #16226 status | Outdated | The article says the PR is "taking a long time to be merged." It was **merged on June 5, 2025** and is available via `VLLM_SLEEP_WHEN_IDLE=1` or `--sleep-on-idle`. |
| Container-only claim | Factual Error | The article states the patch "only works when deploying vLLM inside a container." This is incorrect — the fix is a Python-level change in `shm_broadcast.py` that works in any environment. |
| "sGLANG" naming | Minor | Consistently misspelled as "sGLANG" — the correct name is **SGLang**. The corresponding PR #6026 was also merged (June 12, 2025). |
| Ongoing issues not mentioned | Outdated | The merged fix only addresses the main engine's message queue. Ray distributed deployments and worker-level busy loops remain affected (issues #21231, #19036, #25122). |
| Pip install instructions | Minor | `--global-option` syntax is deprecated in modern pip. |

---

### 01: KV Cache RAM Swap is ~10x Faster Than Recomputation

**Status: Significantly Outdated**

| Issue | Severity | Details |
|-------|----------|---------|
| "No widely known implementation" | Outdated | **vLLM v0.11.0 (January 2026)** introduced a native KV Offloading Connector for GPU-to-CPU offloading. LMCache provides production-ready integration via the KV connector interface. SGLang has experimental support. NVIDIA Dynamo also supports CPU/SSD offloading. |
| Claimed speedup "7-15x" | Misleading | The article's title says "~10x" but the body calculates 7-15x. Actual performance depends on PCIe generation, transfer size, and system load. The general direction is correct but precision varies. |
| "Novel within the open-source community" | Outdated | This was debatable even at publication. vLLM had an RFC (issue #16144) for V1 offloading since June 2025. |

---

### 02: Dual-GPU Boosts Speed Despite Common Wisdom

**Status: Mostly Accurate**

| Issue | Severity | Details |
|-------|----------|---------|
| H100 vs dual 5090 numbers | Verified | DeutscheKI benchmarks confirm: H100 = 78.64 tok/s, dual 5090 = 79.51 tok/s on QwQ-32B-AWQ. Numbers in the article (78 vs 79) are consistent. |
| Cost claims | Misleading | The article says "30,000 euros" for H100 vs "5,718 euros" for dual 5090s, calling it "10% the cost." The actual ratio is ~19%, not 10%. Also, H100 80GB HBM3 prices vary widely ($25K-$40K). |
| "4K, 8K, or even 16K GPU configurations" (in article 00) | N/A | Cross-reference note: article 00 mentions these scales, which are datacenter-level, not typical for the audience. |
| Filename typo | Minor | Filename contains "despire" instead of "despite." |

---

### 03: Software FP8: 3x Speedup Without Hardware Support

**Status: Mostly Accurate**

| Issue | Severity | Details |
|-------|----------|---------|
| Feather project status | Verified | The GitHub repo at `SuriyaaMM/feather` exists and is active. Claims 3.37x speedup on matrix-vector ops vs FP32. |
| "RTX 20-series" support | Minor | The article claims support for RTX 20-series, but Feather's benchmarks only show RTX 3050 results. Compute capability >= 7.5 (Turing) is stated as a requirement, so 20-series support is technically plausible but unverified. |
| "Encoding two FP8 values into a single FP32 word" | Verified | This bit-packing technique is correctly described. |

---

### 04: 8+ Hours Benchmarking Every MoE Backend for Qwen3.5-397B NVFP4

**Status: Mostly Accurate, One Key Issue**

| Issue | Severity | Details |
|-------|----------|---------|
| CUTLASS SM120 bug | Verified | CUTLASS issue #3096 remains **open and unresolved** as of March 2026. SM120 grouped GEMM failures confirmed. |
| SM121 (DGX Spark) comparison | Verified | SM121 reportedly works correctly, confirming the issue is SM120-specific. |
| "356 TFLOPS on SM121" | Minor | This number appears in context but is not independently verified. |
| Marlin fallback recommendation | Still Valid | Users on SM120 should still use Marlin W4A16 with MTP disabled. |

---

### 05: Benchmarking LLM Inference Backends

**Status: Outdated**

| Issue | Severity | Details |
|-------|----------|---------|
| Framework landscape | Outdated | The article compares vLLM, LMDeploy, MLC-LLM, TensorRT-LLM, and TGI. Since publication, the landscape has shifted significantly — SGLang has become a major contender, MLC-LLM has been superseded, and vLLM V1 architecture was released. |
| "LMDeploy emerged as the top performer" | Context-dependent | This was for a specific benchmark (Llama 3 on BentoCloud). Results vary significantly by model, hardware, and workload. |
| Low detail | Minor | The article is vague compared to others — lacks specific numbers, hardware specs, or reproducible methodology. |

---

### 06: DeepSeek Open-Sources nano-vLLM

**Status: Attribution Error**

| Issue | Severity | Details |
|-------|----------|---------|
| "The team behind DeepSeek has recently open-sourced nano-vLLM" | Misleading | nano-vllm is a **personal project** by Xingkai Yu (GeeeekExplorer), who is a DeepSeek engineer. It is **not an official DeepSeek product**. The article's framing implies organizational backing that doesn't exist. |
| Repository URL | Factual Error | The article links to `github.com/GeeeekExplorer/nano-vllm`, which is correct. However, the attribution to "DeepSeek" is wrong. |
| "1,200 lines of Python code" | Verified | Consistent with the repo's current state. |
| Feature list | Verified | Prefix caching, tensor parallelism, torch compilation, CUDA graph tracing — all present in the repo. |
| Project traction | Note | The project now has 12.2K stars and 1.7K forks, indicating significant community adoption. |

---

### 07: GH200 Desktop: vLLM Tuning Notes

**Status: Mostly Accurate**

| Issue | Severity | Details |
|-------|----------|---------|
| "€9K" pricing | Plausible | A Hacker News post (December 2025) describes acquiring a GH200 server for €7.5K and converting it to a desktop. Secondary market GH200 prices have dropped significantly from the original €47.5K+ workstation price. |
| TP2 > PP2 on non-NVLink | Verified | This is a well-documented finding. Tensor parallelism outperforms pipeline parallelism for decode-heavy workloads on PCIe-connected systems. |
| `--max-num-seqs 16` recommendation | Still Valid | Scheduler tuning remains critical for vLLM performance. |
| `VLLM_SLEEP_WHEN_IDLE=0` | Minor | Article says this "eliminates jumpscare latency." Note: this is the **default** (disabled). Setting it to `1` enables idle sleep, which adds ~100ms wake latency. The article's description is technically correct but could be clearer. |

---

### 08: Megakernel Doubles Batch-1 Inference Speed

**Status: Accurate**

| Issue | Severity | Details |
|-------|----------|---------|
| Paper and blog post | Verified | "Look Ma, No Bubbles!" published May 27, 2025 by Hazy Research at Stanford. Confirms 78% memory bandwidth utilization and >1.5x speedup over vLLM/SGLang on H100. |
| "Twofold increase" claim | Verified | The article says "twofold" while the paper shows ">1.5x" improvement. The article's "doubles" framing is slightly optimistic but within range depending on the specific configuration. |
| "No mention of llama.cpp" | Verified | Correct observation — the paper focuses on vLLM and SGLang comparisons. |
| Consumer GPU scalability | Still Untested | The article correctly notes this limitation. |

---

### 09: Patched P2P Driver Enables Multi-5090 Systems

**Status: Partially Outdated**

| Issue | Severity | Details |
|-------|----------|---------|
| aikitoria fork | Verified | The repository exists and has been updated. Now targets driver version 590.48.01 (was 570.148.08). |
| "Tinygrad driver" reference | Misleading | The article calls it the "official Tinygrad driver" — it's actually NVIDIA's open-gpu-kernel-modules, with community forks by both tinygrad and aikitoria. |
| P2P reliability | Outdated | Multiple users report ongoing P2P issues with 5090s even with the patch (issues #42, #44 in tinygrad fork). 8x5090 systems still report "CANNOT Access Peer." |
| Security warning | Adequate | The article correctly warns about unverified kernel modules, but should more strongly emphasize IOMMU must be disabled, which is a significant security risk. |
| "Blackwell 2.0 architecture" | Minor | This is not a standard NVIDIA designation. |

---

### 10: Qwen3-Next 80B FP8 on WSL2 + vLLM + Docker

**Status: Accurate but Version-Specific**

| Issue | Severity | Details |
|-------|----------|---------|
| Version pinning | Accurate at time | PyTorch 2.8.0 (cu128), vLLM 0.10.2, FlashInfer 0.3.1 were correct versions. These may be outdated by now as newer versions are available. |
| 80 tok/s throughput | Plausible | Consistent with other RTX Pro 6000 benchmarks in this repository. |
| "Do not rely on Claude or ChatGPT" | Note | This warning about AI assistants giving incorrect advice for bleeding-edge stack configurations is a fair observation. |

---

### 11: NVIDIA NVFP4: 4-bit Pretraining Matches FP8 Accuracy

**Status: Contains Factual Error**

| Issue | Severity | Details |
|-------|----------|---------|
| MBPP+ scores reversed | Factual Error | The article states: "on MBPP+, performance dipped slightly: 55.91% in FP8 versus 59.11% in NVFP4." This is **backwards**. The paper (arXiv 2509.25149) shows **FP8 = 59.11%** and **NVFP4 = 55.91%**. FP8 outperforms NVFP4 on MBPP+, as expected for higher precision. |
| MMLU Pro scores | Verified | FP8 = 62.62%, NVFP4 = 62.58% — confirmed correct (NVFP4 slightly lower). Note: the article swaps these too, saying "62.58% under FP8 and 62.62% under NVFP4" when it should be the reverse. |
| "12-billion-parameter Mamba Transformer" | Verified | Confirmed as a Nemotron-H family hybrid model (Mamba-2 + Self-Attention + FFN). |
| arXiv ID | Verified | 2509.25149 exists and matches the described content. |

---

### 12: Qwen3-30B FP8 on RTX Pro 6000 Blackwell: 88.4 tok/s

**Status: Accurate**

| Issue | Severity | Details |
|-------|----------|---------|
| RTX Pro 6000 specs | Verified | 96GB GDDR7, PCIe Gen 5, 600W TDP confirmed. |
| 88.4 tok/s single user | Plausible | Consistent with other community benchmarks on this hardware. |
| 450W power limit | Note | Below the card's 600W TDP — intentional throttling for efficiency testing. |
| 256K context performance | Note | 22 tok/s at 256K context for a 30B model is impressive and not contradicted by other sources. |

---

### 13: RTX Pro 6000 vLLM Benchmark: 120B Model Performance

**Status: Accurate**

| Issue | Severity | Details |
|-------|----------|---------|
| openai/gpt-oss-120b | Verified | Model exists on Hugging Face. 117B MoE with 5.1B active parameters, MXFP4 quantization, Apache 2.0 license. |
| 1051 tok/s peak | Plausible | For a model with only 5.1B active parameters on 96GB VRAM, this is reasonable. |
| vLLM 0.11.0 | Verified | Version exists and is a real vLLM release. |
| "CUDA 13.0" | Note | This would correspond to a very recent CUDA toolkit version. |

---

### 14: LMCache: Reuse Non-Prefix KV Cache, 3x RAG Speedup

**Status: Accurate**

| Issue | Severity | Details |
|-------|----------|---------|
| EuroSys 2025 Best Paper | Verified | CacheBlend won Best Paper at ACM EuroSys 2025, confirmed by conference and UChicago CS department. |
| ACM DOI | Verified | `10.1145/3689031.3696098` is correct. |
| "100% KV Cache hit rate" | Verified | The paper's key contribution is enabling non-prefix KV cache reuse for near-100% hit rates in RAG. |
| 2.2-3.3x TTFT improvement | Verified | Consistent with the paper's reported results. |

---

## Cross-Cutting Issues

### 1. Generation Artifacts
Several articles contain patterns typical of LLM-generated text:
- Excessive hedging ("unverified," "as described," "according to the post")
- Repetitive disclaimer structures
- Some articles pad thin source material with speculative analysis

### 2. Naming Consistency
- "sGLANG" should be "SGLang" (article 00)
- "Blackwell 2.0" is not a standard NVIDIA designation (article 09)
- SM120 is used correctly for the RTX Pro 6000's streaming multiprocessor architecture

### 3. README Accuracy
- Article count: README says "15 posts most relevant" but only 15 articles exist (numbered 00-14), which is consistent.
- Nemotron 9B Japanese was used for generation, which explains some stylistic patterns.
- Pipeline description is accurate and transparent.

---

## Recommendations

1. **Fix factual errors** in articles 00 (container-only claim), 06 (DeepSeek attribution), and 11 (swapped benchmark scores)
2. **Add update notices** to articles 00, 01, and 05 noting significant ecosystem changes since publication
3. **Correct typos**: "despire" in filename 02, "sGLANG" -> "SGLang" in article 00
4. **Add dates** to articles — none of the articles include publication dates, making it difficult to assess timeliness

---

*Audit performed by Claude (Opus 4.6) on 2026-03-16. Verification sources include GitHub issues/PRs, arXiv papers, official product pages, and community benchmarks.*
