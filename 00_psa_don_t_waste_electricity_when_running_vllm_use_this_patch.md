> **Opus 4.6 Audit Insights (2026-03-16)**
> - PR #16226 は **2025年6月5日にマージ済み**。`VLLM_SLEEP_WHEN_IDLE=1` または `--sleep-on-idle` で利用可能。
> - 「コンテナ内でのみ動作」は **誤り** — Python レベルの修正（`shm_broadcast.py`）であり環境を問わず動作する。
> - "sGLANG" は **SGLang** の誤記。対応 PR #6026 も 2025年6月12日にマージ済み。
> - ただし Ray 分散環境やワーカーレベルのビジーループには未対応（issues #21231, #19036, #25122）。

# PSA: Don't Waste Electricity When Running vLLM — A Community-Optimized Patch

> **Disclaimer**: The following information is based on unverified community reports from r/LocalLLaMA. It reflects user experiences and unverified technical guidance provided by Reddit users. Always exercise caution and verify before applying patches to production systems.

---

## The Problem: Unnecessary CPU Overhead in vLLM

According to a Reddit post in r/LocalLLaMA, which has received **303 upvotes** and sparked 29 comments, users have reported an unexpected power consumption issue when running **vLLM**, a high-performance LLM inference engine.

The problem, as described, is that vLLM aggressively utilizes CPU resources — specifically, it uses **100% CPU across as many cores as there are connected GPUs**, even during idle periods. For example, a user with **8 GPUs** connected to a single machine observed that the system’s idle power draw nearly **doubled** due to this behavior, resulting in significant energy waste.

This inefficiency stems from vLLM’s default scheduling strategy, which appears to over-provision CPU threads to match the number of GPUs, even when no active inference is occurring.

---

## The Solution: A Community Patch

To address this, a Reddit user submitted a targeted fix via a pull request:  
**[PR #16226 in vLLM](https://github.com/vllm-project/vllm/pull/16226)**

The patch aims to reduce unnecessary CPU thread allocation during idle states, thereby lowering power consumption without compromising performance during active workloads.

While the PR is reportedly taking a long time to be merged — a common challenge for community-contributed fixes — users can apply the proposed changes **today** using instructions provided in the issue comments:  
**[Issue Comment #2839769179](https://github.com/vllm-project/vllm/pull/16226#issuecomment-2839769179)**

> ⚠️ **Important**: This patch only works when deploying vLLM inside a **container**. Users running vLLM in non-containerized environments (e.g., bare-metal or Docker-free setups) will not be able to apply this fix directly.

---

## How to Apply the Patch

The linked comment outlines a step-by-step process to manually apply the patch:

1. Clone the vLLM repository:  
   ```bash
   git clone https://github.com/vllm-project/vllm.git
   cd vllm
   ```

2. Apply the patch from the PR:  
   ```bash
   git apply https://github.com/vllm-project/vllm/pull/16226.patch
   ```

3. Rebuild the project:  
   ```bash
   pip install -e .
   ```

4. Rebuild with CUDA support (if needed):  
   ```bash
   pip install --editable . --global-option="--cpp_ext" --global-option="--cuda_ext"
   ```

5. Deploy vLLM inside a container (Docker or Podman) as usual.

According to the post, after applying the patch, the user reported a **significant reduction in idle power usage**, bringing it closer to optimal levels — especially beneficial in multi-GPU setups.

---

## A Similar Fix for sGLANG

In addition to vLLM, the same Reddit user pointed out a **similar patch for sGLANG**, another LLM inference framework:

- **[PR #6026 in sGLANG](https://github.com/sgl-project/sglang/pull/6026)**

This patch addresses comparable inefficiencies in sGLANG’s CPU-GPU thread management, suggesting that this is a broader issue across LLM serving platforms.

---

## Why This Matters

For researchers, developers, and hobbyists running large language models locally — especially across 4K, 8K, or even 16K GPU configurations — idle power consumption can become a major cost center. Even small inefficiencies compound quickly in multi-GPU setups.

By reducing unnecessary CPU thrashing, these community-driven fixes help optimize both **energy efficiency** and **thermal management**, extending hardware lifespan and reducing electricity bills.

---

## Final Notes

While the patches are not yet officially merged, they represent active community efforts to improve real-world performance. As always, apply such changes at your own risk, and consider reaching out to the vLLM and sGLANG maintainers to advocate for inclusion in upstream releases.

If you’re running LLMs on multiple GPUs and noticing high idle power usage, this could be a simple yet impactful optimization.
