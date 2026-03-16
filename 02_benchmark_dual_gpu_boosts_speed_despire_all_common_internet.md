> **Opus 4.6 Audit Insights (2026-03-16)**
> - ベンチマーク数値は DeutscheKI リポジトリで **検証済み**: H100 = 78.64 tok/s, Dual 5090 = 79.51 tok/s。記事の数値（78 vs 79）と一致。
> - コスト比較「H100 の 10%」は **不正確** — 実際の比率は約 19%（5,718 EUR vs ~30,000 EUR）。H100 価格は $25K〜$40K と幅がある。
> - H100 は TTFT（33.1ms vs 56.9ms）で優位。スループットのみの比較である点に注意。

**Dual-GPU Surprise: Why Two GPUs Outperform One — And the RTX 6000 Ada’s Pricey Disappointment** (r/LocalLLaMA, 161 upvotes)

In a recent benchmarking spree on r/LocalLLaMA, an enthusiastic tester shattered widely accepted assumptions about LLM inference performance on multi-GPU systems. With a focus on the QwQ-32B-AWQ model running via vLLM, the results revealed that dual-GPU setups not only work — they *dominate*, often outperforming flagship single cards like the NVIDIA H100 and RTX 4090. This challenges the common internet wisdom that parallel setups don’t improve *sequential* speed for inference workloads. Let’s dive into the numbers.

According to the post, the Time To First Token remained consistently under 0.1 seconds across all configurations, so the real race was for Output Tokens Per Second (OT/s), especially critical for reasoning models like QwQ-32B, where 4,000+ tokens of internal monologue dominate latency. Testing was performed under a single-user scenario to isolate performance gains.

To the surprise of many, two RTX 4070 TI SUPER cards beat a single RTX 4090, delivering 46 OT/s versus 43 OT/s. Even more impressively, two RTX 4080 cards achieved 52 OT/s, reaching 80% of a single 5090’s performance. The older RTX 3090 TI still holds its ground at 40 OT/s — a solid 61% of what a new 5090 offers.

But the real headline is the dual-RTX-5090 setup, which outperforms the NVIDIA H100 80GB HBM3 by a narrow margin, delivering **78 OT/s** for the H100 versus **79 OT/s** for two RTX 5090s. This means **30,000 euros of inference performance** at just **5,718 euros**, or roughly **10% the cost** of the H100. The advantage comes from the 5090’s ability to leverage additional RAM for faster attention kernels — a feature the H100 doesn’t match in this use case.

Meanwhile, the RTX 6000 Ada, priced at a premium, disappoints with only 42 OT/s — barely better than the 3090 TI and significantly behind dual-4070 setups. This makes it one of the least efficient high-end options for LLM inference today.

These results, shared on GitHub ([link](https://github.com/DeutscheKI/llm-performance-tests#qwq-32b-awq)), challenge long-held beliefs and highlight the evolving landscape of consumer-grade LLM acceleration. While NVIDIA’s data center cards still hold value for specific workloads, the latest RTX 50-series GPUs are proving to be surprisingly powerful — especially when paired.
