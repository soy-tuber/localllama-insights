> **Opus 4.6 Audit Insights (2026-03-16)**
> - CacheBlend の **ACM EuroSys 2025 Best Paper 受賞を確認**。UChicago CS 学部と EuroSys 公式アカウントが裏付け。
> - ACM DOI（`10.1145/3689031.3696098`）は**正確**。
> - TTFT 2.2〜3.3x 改善の主張は論文の報告と**一致**。
> - LMCache は現在 vLLM の公式 KV connector インターフェースに統合されており、**エコシステムでの地位を確立**。

# Reuse Non-Prefix KV Cache and Speed Up RAG by 3X with LMCache

**According to the post on Reddit (r/LocalLLaMA) with 127 upvotes**, a team behind the open-source project LMCache has introduced **CacheBlend**, a technique that enables 100% reuse of KV caches regardless of their position in the input sequence. Recognized with a **Best Paper Award at ACM EuroSys 2025**, this innovation is designed to solve a persistent inefficiency in Retrieval-Augmented Generation (RAG) and other dynamic context applications.

## The Problem: Your KV Cache Is Wasting Potential

In RAG workflows, documents are retrieved and injected into the model’s context during inference. However, this retrieved content is rarely placed at the very beginning of the prompt — it often appears mid-sequence or even at the end. Traditional KV caching mechanisms only allow reuse of the *prefix* — the initial part of the input that remains fixed across multiple generations. As a result, when new or dynamically retrieved tokens appear outside this prefix, the cache miss rate spikes.

**As the post states**, this forces the model to recompute embeddings and attention for the same tokens repeatedly, leading to wasted compute and slower response times. For high-throughput or low-latency applications, this inefficiency can be a major bottleneck.

## The Solution: CacheBlend — 100% Hit Rate, No Compromises

**CacheBlend**, the core mechanism described in the post, revolutionizes how KV caches are reused. Unlike prefix-only caching, CacheBlend allows **non-prefix KV cache reuse**, meaning any previously computed key-value pairs can be reused no matter where they appear in the input sequence.

The result? A **100% KV Cache hit rate** in RAG and similar applications. This unlocks dramatic performance improvements:

- **Faster Time-To-First-Token (TTFT):** Users receive initial responses significantly quicker.
- **Higher Throughput:** The same hardware can serve many more users under load.
- **Near-lossless Output Quality:** Despite aggressive cache reuse, the quality of generated text remains almost unchanged.

The technical foundation of CacheBlend addresses two critical challenges when reusing out-of-order KV caches:

1. **Positional Encoding Update:** The model must always know the correct position of each token. CacheBlend efficiently updates positional encodings to reflect the true sequence order, even when cached and new tokens are interleaved.
2. **Selective Attention Recalculation:** Instead of recomputing full attention across the entire sequence, CacheBlend intelligently recalculates only the minimal cross-attention required between new and cached segments — preserving accuracy while cutting computation.

These optimizations allow the system to treat the entire input as cacheable, eliminating the “prefix-only” limitation of conventional methods.

## Where Can I Try It?

The team provides an interactive demo at:  
[https://github.com/LMCache/LMCache-Examples/tree/main/demo-rag-blending](https://github.com/LMCache/LMCache-Examples/tree/main/demo-rag-blending)

This allows developers to experiment with CacheBlend in real-time RAG scenarios and observe the performance gains firsthand.

## Final Thoughts

For engineers building scalable LLM applications today, CacheBlend represents a significant step forward in efficient context management. By treating KV caches as fully reusable across arbitrary token positions, it removes a key performance ceiling in dynamic prompting paradigms like RAG.

**According to the post**, this breakthrough has already earned academic recognition and is available as an open-source tool — making it accessible to the broader AI community. With 127 upvotes and active discussion in the comments, it’s clearly resonating with developers seeking to optimize real-world LLM deployments.

For further details, refer to the official paper:  
[https://dl.acm.org/doi/10.1145/3689031.3696098](https://dl.acm.org/doi/10.1145/3689031.3696098)

> *This blog post presents claims and data from a Reddit submission in r/LocalLLaMA, which has received 127 upvotes. All technical terms, numbers, and attribution are quoted directly from the original post.*
