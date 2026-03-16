> **Opus 4.6 Audit Insights (2026-03-16)**
> - CUTLASS issue #3096 は **2026年3月時点で未解決**。SM120 での grouped GEMM 失敗は継続中。
> - SM120 は SM100（データセンター向け）と異なり `tcgen05` 命令や TMEM を持たない。Triton は SM12x を SM80（Ampere）として扱うため Blackwell 固有最適化が無効。
> - Marlin W4A16 + MTP 無効の回避策は **依然として有効**。
> - FlashInfer 0.6.5 のパッチにより SM120 で部分的動作可能だが、Marlin より約 20% 低速。

## Unverified Community Report: Qwen3.5-397B NVFP4 Performance on RTX PRO 6000

**Disclaimer:** This article summarizes unverified community information shared by a user on Reddit. All claims, metrics, and technical observations are presented as-is from the original post and should be treated as anecdotal evidence until independently validated.

**Source:** Reddit post in r/LocalLLaMA  
**Upvotes:** 223  
**Comments:** 65  
**Original Poster:** u/TechnicalBenchmarker  
**Date:** Not specified  

---

### The Short Version

According to the post, the best sustained decode performance achieved for **Qwen3.5-397B-A17B-NVFP4** on **4x RTX PRO 6000 (SM120)** is **50.5 tok/s**, despite claims of higher throughput elsewhere. The user attributes this to a **known issue in NVIDIA’s CUTLASS library** on SM120 (RTX PRO 6000), where grouped GEMM kernels for FP4 tensor cores fail to initialize, preventing the model from leveraging NVFP4 acceleration. The author emphasizes that **SM121 (DGX Spark) works as expected**, suggesting the problem lies in unvalidated tile configurations for SM120 rather than hardware limitations.

---

### The Setup

The benchmark was conducted on the following hardware:

- **4x RTX PRO 6000 Blackwell Workstation Edition** (96GB GDDR7 per card, 384GB total)
- **SM 12.0** — desktop/workstation variant, not datacenter B200 (SM 10.0)
- **PCIe Gen5**, **no NVLink**
- **Threadripper 24C/48T**, **512GB DDR5**
- **Windows 11 + WSL2**
- **Model:** `nvidia/Qwen3.5-397B-A17B-NVFP4` (~140GB total, 397B parameters, 17B active per token)

The goal was to evaluate **every available MoE backend**, inference framework, and configuration under SM120 constraints.

---

### 16 Configurations Tested

The user evaluated **16 distinct configurations** across multiple frameworks and backends, including:

- **Marlin W4A16** (custom MoE quantized backend)
- **FlashInfer** with **CUTLASS**
- **vLLM native CUTLASS**
- **SGLang 0.5.8**
- **Expert Parallel (EP)**
- **TensorRT-LLM**
- **FlashInfer Sampler**

Results are summarized in the table above, with **Marlin TP=4, no MTP** achieving the highest sustained throughput at **50.5 tok/s**. Other configurations either underperformed or produced **NaN** or **garbage output**.

---

### The NVIDIA Bug Blocking NVFP4 Performance

The core issue identified is a **failure in CUTLASS’s grouped GEMM kernels** on SM120:

> All 80 TMA Warp Specialized (TMA WS) grouped GEMM tactics fail at initialization on SM120 with the error:  
> `Failed to initialize cutlass TMA WS grouped gemm.`  
> `Error: Error Internal (cutlass_kernel_file_gemm_grouped_sm120_M128_BS_group2.generated.cu:60)`

This prevents the use of **NVFP4-quantized model weights**, forcing the system to fall back to **Marlin**, which **dequantizes FP4 weights to FP16** and runs standard GEMM operations. As a result, the system **loses approximately half the theoretical FP4 throughput**.

Notably, the same model runs at **356 TFLOPS** on **SM121 (DGX Spark)**, confirming that the issue is **configuration-specific to SM120** and not a hardware or model flaw.

---

### Why MTP Makes Things Worse

The user found that **Multi-Token Prediction (MTP)** — expected to improve efficiency — caused a **-22% regression** under SM120 when using Marlin:

- Without MTP: 50.5 tok/s  
- With MTP: 39–40 tok/s  

This counterintuitive result suggests that **MTP introduces overhead on SM120** due to the underlying kernel instability, possibly from misaligned memory accesses or incomplete kernel initialization. The user concludes that **MTP should be disabled** on SM120 when using NVFP4 models.

---

### Conclusion

This unverified report highlights a **critical gap in NVIDIA’s software stack**: while the RTX PRO 6000 supports NVFP4 and the Qwen3.5-397B model is officially NVFP4-quantized, **CUTLASS does not yet support grouped GEMM on SM120**, rendering the hardware’s FP4 capabilities unusable for MoE inference. The user’s extensive testing across configurations confirms that **Marlin W4A16 with TP=4 and no MTP** is currently the most reliable option, delivering **50.5 tok/s** — well below the theoretical potential.

The author has filed a GitHub issue ([#3096](https://github.com/NVIDIA/cutlass/issues/3096)) with no response, underscoring the need for NVIDIA to validate and fix SM120-specific kernel configurations. Until then, users should avoid relying on CUTLASS for NVFP4 MoE workloads on SM120 and consider **disabling MTP** and **falling back to Marlin** for stable performance.
