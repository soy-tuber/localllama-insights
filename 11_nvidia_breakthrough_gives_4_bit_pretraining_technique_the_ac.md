# NVIDIA Breakthrough: 4-Bit Pretraining Achieves FP8 Accuracy with NVFP4

**Score:** 808 upvotes, 99 comments (Reddit: r/LocalLLaMA)

In a significant advancement for efficient AI training, NVIDIA has introduced a new method called **NVFP4** that enables 4-bit precision during the pretraining phase of large language models while maintaining accuracy comparable to the widely used FP8 format. Shared in a viral Reddit post with 808 upvotes and 99 comments, the claim has sparked interest across the AI and machine learning communities, particularly among those working with resource-constrained environments like local LLM deployments.

According to the post, NVFP4 is designed to store numerical values using only 4 bits per parameter during training — a drastic reduction from conventional 8-bit or 16-bit formats. This compression not only accelerates training throughput but also reduces memory bandwidth requirements, making large-scale model training more feasible on consumer-grade hardware.

The post highlights experimental results using NVFP4 on a **12-billion-parameter Mamba Transformer** trained on **10 trillion tokens**. Researchers observed that NVFP4 achieved **validation loss within 1% of FP8** throughout most of the training process. Even during late-stage learning rate decay, the gap widened only to approximately **1.5%**, suggesting that the final model retains strong generalization capabilities.

Crucially, the impact on downstream task performance was minimal for most benchmarks. On **MMLU Pro**, the model scored **62.58%** under FP8 and **62.62%** under NVFP4, a near-identical result. However, on **MBPP+**, a coding-focused evaluation, performance dipped slightly: **55.91%** in FP8 versus **59.11%** in NVFP4. This discrepancy may reflect the higher sensitivity of code generation tasks to precision variations during training.

The findings are supported by an associated **arXiv paper** (ID: 2509.25149) and shared on **X** by the poster, @godofprompt, who linked the thread to broader implications for efficient AI training at scale.

While these results are promising, the post emphasizes that the information is **unverified community content** and should not be treated as peer-reviewed fact. The paper and code repository are linked but have not yet undergone formal review.

For developers and researchers exploring efficient LLM training on limited hardware — such as desktops with GPUs like RTX 4090 — NVFP4 represents a potential breakthrough. By enabling 4-bit pretraining without sacrificing most of FP8’s accuracy, it could democratize access to large-scale model development.

As the AI landscape continues to prioritize efficiency, techniques like NVFP4 may become foundational in the next generation of open-weight, locally deployable large language models.
