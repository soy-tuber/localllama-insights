> **Opus 4.6 Audit Insights (2026-03-16)**
> - €9K の価格は **中古/余剰市場でのみ妥当**。新品の GH200 ワークステーションは €47,500〜。2025年末に HN で €7.5K での中古購入報告あり。
> - TP2 > PP2（non-NVLink 環境）の知見は **広く確認されている**。デコード主体のワークロードで特に有効。
> - `--max-num-seqs 16` のスケジューラチューニングは **依然として重要**な最適化ポイント。
> - `VLLM_SLEEP_WHEN_IDLE=0` はデフォルト値（無効）。`=1` で有効化すると ~100ms のウェイク遅延が発生。

**Unverified Community Report: The €9K GH200 “Desktop” That Saved $1.27 on Claude Code (vLLM Tuning Notes)**
*Score: 648 upvotes | 170 comments*

---

### Introduction

This story comes directly from the r/LocalLLaMA community — unverified by official NVIDIA or vendor sources. As such, readers are encouraged to treat all technical claims as community observations, not endorsements. That said, the post has gathered significant attention, with 648 upvotes, suggesting this experience resonated deeply with those building large-scale local LLM setups.

---

### The Setup: A €9K GH200 “Desktop”

The author purchased two NVIDIA GH200 Grace–Hopper GPUs — each equipped with 96GB of HBM3 memory — for a total of **192GB VRAM**. Marketed as a “desktop,” the system was assembled locally and runs on a non-NVLink topology (`SYS`), meaning communication occurs over PCIe and NUMA rather than high-bandwidth NVLink interconnects.

Conventional wisdom suggested that without NVLink, the model would need to rely on pipeline parallelism (PP) to scale. But as the author put it: *“Surely guides on the internet wouldn’t betray me.”*

---

### What Actually Worked: The “Boring” But Critical Tuning

After a week of tuning **vLLM**, the author achieved a stable configuration that allowed **Claude Code** to run a **~140GB** model locally — avoiding any outbound API calls.

Key configuration choices:

- `--tensor-parallel-size 2` (TP2)  
- `--max-context-size 163840` (163,840 tokens)  
- `--max-num-seqs 16` — critical for scheduler responsiveness  
- Chunked prefill default (`8192`)  
- `VLLM_SLEEP_WHEN_IDLE=0` — eliminates “jumpscare” latency after idle periods

The community credits **mratsim** for providing a MiniMax-M2.1 model tuned for FP8+INT4 AWQ quantization at the 192GB VRAM sweet spot — a crucial enabler of this setup.

---

### The “I Can’t Believe This” Reality Check: Pipeline Parallel Fails

Despite the `SYS` topology (no NVLink), the author initially tried pipeline parallelism (PP2). However, this path hit a hard wall:

- PP2 failed to launch at 163,840 context length — KV cache allocation rejected it  
- Even after reducing to **114k** tokens, performance remained poor:
  - `short_c4`: **~49.9 tok/s** (vs. **~78 tok/s** with TP2)
  - `short_c8`: **~28.1 tok/s** (vs. **~66 tok/s** with TP2)
  - TTFT (time-to-first-token) showed **multi-second warmup** and erratic behavior

Conclusion: **Tensor parallelism (TP2) outperformed pipeline parallelism (PP2)** on this non-NVLink setup — a surprising reversal of expectations.

---

### The Hidden Boss: `--max-num-seqs`

The scheduler’s behavior was revealed to be the hidden performance bottleneck. Testing different values:

- `--max-num-seqs 4`: scheduler became overloaded — TTFT p99 spiked dramatically  
- `--max-num-seqs 16`: ideal balance — responsive, stable, high throughput  
- `--max-num-seqs 32`: acceptable, but 16 felt safer without performance loss

Insight: If **Claude Code feels randomly slow**, the issue may not be GPU-bound — it could be scheduler congestion due to aggressive request queuing.

---

### The Financial “Payout”

In a moment of dark humor, the system printed a cost summary after a run:

```
Total cost:            $1.27 (costs may be inaccurate due to usage of unknown models)
Total duration (API):  1m 58s
Total duration (wall): 4m 10s
Usage by model:
    MiniMax-M2.1-FP8:  391.5k input, 6.4k output, 0 cache read, 0 cache write ($1.27)
```

The author spent **€9,000** (~$9,500 USD) on this system and saved **$1.27** on a single API call. To break even on cost, they estimate needing **~7,000+ code reviews**.

---

### Final Thoughts

This story is a cautionary tale — and a curiosity — for those building local LLM infrastructure. While the financial payoff is minimal, the technical insights are valuable:

- **Topology matters**: NVLink absence doesn’t doom pipeline parallelism, but tensor parallelism can still win.
- **Scheduler tuning is critical**: `--max-num-seqs` isn’t just a knob — it’s a performance lever.
- **Quantization matters**: Fine-tuned FP8+INT4 AWQ models are essential for 192GB deployments.

The full technical breakdown is available at:  
[Read all the details here!](https://dnhkng.github.io/posts/vllm-optimization-gh200/)

*Disclaimer: This post contains unverified community observations. All technical decisions should be validated through benchmarking in your own environment.*
