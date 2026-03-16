> **Opus 4.6 Audit Insights (2026-03-16)**
> - aikitoria フォークは **活発に更新中**（現在 driver 590.48.01 → 595.45.04 対応、151 スター）。
> - NVIDIA は **公式には P2P をコンシューマカードでサポートしていない**。NCCL や公式フォーラムでも未対応を確認。
> - 8x5090 環境では依然として「CANNOT Access Peer」が報告されており（tinygrad fork issues #42, #44）、**大規模構成での信頼性に課題**。
> - IOMMU 無効化が必須であり、**セキュリティリスクが大きい**。記事ではこの点の強調が不十分。
> - 「Blackwell 2.0」は NVIDIA の公式呼称ではない。

**Unverified Community Update: Patched P2P NVIDIA Driver Enables High-Bandwidth Interconnect Between Multiple 5090s and Blackwell Graphics Cards**

*Source: Reddit post from r/LocalLLaMA, 86 upvotes, 24 comments*

---

### Introduction

A recent unverified community report suggests that a patched version of the NVIDIA driver now supports peer-to-peer (P2P) memory access across multiple high-end GPUs, including the RTX 5090, and may extend compatibility to the upcoming Blackwell 2.0 architecture. This development is particularly significant for multi-GPU setups used in AI inference and training, where efficient GPU-to-GPU communication can dramatically improve performance.

The update, shared by user *pancho* in r/LocalLLaMA, indicates that a forked kernel module — [open-gpu-kernel-modules](https://github.com/aikitoria/open-gpu-kernel-modules) — now resolves long-standing P2P issues observed in the official Tinygrad driver (branch `570.148.08-p2p`). According to the post, this patched driver enables reliable bidirectional P2P between multiple RTX 5090s, with measurable gains in bandwidth and latency.

> **Important Note**: This information is sourced directly from a community Reddit post and has not been independently verified by NVIDIA or third-party security teams. Users should exercise caution before deploying unverified kernel modules, especially in production or cloud environments.

---

### Technical Details and Performance Results

Using the patched driver, *pancho* ran the `cuda-samples/p2pBandwidthLatencyTest` utility on a system configured with two RTX 5090s operating at X8/X8 5.0 over a Threadripper CPU. The results, shown below, reveal substantial improvements in GPU interconnect performance.

#### P2P Connectivity Matrix
```
    D\D   0   1
  0     1   1
  1     1   1
```
The matrix confirms full bidirectional P2P connectivity between both devices.

#### Bandwidth and Latency Comparisons
| Mode                     | Bandwidth (GB/s) | Latency (us) |
|--------------------------|------------------|--------------|
| **Unidirectional (Disabled)** | 1741.98 → 1751.59 | 2.08 → 2.08   |
|                          | 24.35 → 28.67    | 14.38 → 0.48  |
| **Bidirectional (Enabled)**  | 1737.98 → 1765.44 | 2.10 → 2.07   |
|                          | 30.20 → 55.94    | 14.65 → 0.48  |

- **Unidirectional P2P writes** improved from **24.35 GB/s to 28.67 GB/s**, a **~18% increase**.
- **Bidirectional P2P** achieved a dramatic leap from **30.20 GB/s to 55.94 GB/s**, nearly doubling bandwidth.
- **Latency dropped from ~14 microseconds to 0.48 microseconds**, representing a **96% reduction**.

*pancho* notes that with a Threadripper processor providing X8 connectivity to two RTX 5090s, the effective bidirectional bandwidth approaches **112 GB/s** — a substantial boost for multi-GPU AI workloads.

---

### Multi-GPU System Validation

Beyond the 5090 duo, the patched driver appears to support cross-architecture P2P communication. On a system with seven GPUs — two RTX 5090s, two RTX 4090s, two RTX 3090s, and one A6000 — all consumer-class devices — P2P functionality worked between the 4090s, 3090s, and A6000.

This suggests the patch enables interoperability across different GPU generations, potentially simplifying multi-GPU configurations in research and development environments.

---

### Implications for AI and Machine Learning

The enhanced P2P bandwidth and near-instantaneous latency could benefit large language model (LLM) inference and training pipelines that leverage model parallelism. Frameworks such as **LocalLLaMA** and **tinygrad** may see performance gains when deployed across multiple RTX 5090s or future Blackwell GPUs.

Moreover, the ability to connect consumer GPUs like the RTX 4090 and 3090 via P2P opens new possibilities for cost-effective, high-throughput AI clusters built from readily available hardware.

---

### Caution and Next Steps

While the results are promising, users should:
- Verify system stability and driver compatibility before deploying unverified kernel modules.
- Monitor thermal and power limits, as increased GPU interconnect usage may raise system demands.
- Await official NVIDIA confirmation or future driver releases that may incorporate similar functionality.

---

### Conclusion

An unverified but technically detailed community report indicates that a patched NVIDIA P2P driver now enables high-bandwidth, low-latency communication between multiple RTX 5090s and potentially across newer Blackwell architectures. With bidirectional bandwidth exceeding **55 GB/s** and latency dropping below **0.5 microseconds**, this development could reshape multi-GPU AI deployments.

As more users test and validate these findings, the open-source community may play a key role in shaping future GPU virtualization and interconnect standards.

---

*Source: Reddit post by u/pancho, r/LocalLLaMA, 86 upvotes*
