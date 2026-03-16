# Qwen3-30B-A3B in FP8 on RTX PRO 6000 Blackwell: Community Benchmarks Revealed

*This article presents unverified community information from a Reddit post in r/LocalLLaMA. All claims, metrics, and interpretations are shared directly from the post without endorsement or validation by NVIDIA or the author.*

## Introduction

A recent post on Reddit’s r/LocalLLaMA has sparked interest among AI enthusiasts and developers exploring local large language model (LLM) deployment. The report details performance benchmarks for **Qwen3-30B-A3B** using **FP8 quantization** on an **NVIDIA RTX PRO 6000 Blackwell** GPU, with inference powered by **vLLM**. The post includes real-world throughput and latency measurements under varying user loads and context lengths, offering insights into the practical limits of fine-tuned models on consumer-grade hardware.

With **96 upvotes** and **52 comments**, the post has gained notable traction, positioning itself as a reference point for those evaluating FP8 inference performance on NVIDIA’s latest Blackwell architecture.

## Hardware and Software Setup

- **GPU**: NVIDIA RTX PRO 6000 Blackwell  
- **Power Limit**: Set to **450W**  
- **Framework**: vLLM  
- **Model**: Qwen3-30B-A3B quantized to **FP8**  
- **Context Lengths Tested**: 1K, 32K, 64K, 256K tokens  
- **User Loads**: Single user and 10 concurrent users

## Performance Breakdown

### Short Context (1K Tokens)

- **Single user**: **88.4 tok/s**
- **10 concurrent users**: **652 tok/s** (total), with **latency increasing from 5.65s to 7.65s**

### Sweet Spot (32K–64K Tokens)

- **64K tokens @ 10 users**: **311 tok/s** total (**31 tok/s per user**)
- **32K tokens @ 10 users**: **413 tok/s** total (**41 tok/s per user**)

This range shows the best trade-off between context length and throughput, making it ideal for interactive or multi-user applications.

### Long Context (256K Tokens)

- **Single user**: **22.0 tok/s**
- **10 concurrent users**: **115.5 tok/s** total, with **latency rising from 22.7s to 43.2s**

Despite significantly lower throughput and higher latency, the model **still handles 10 concurrent requests**, indicating strong stability under load — a notable achievement for a 30B-parameter model at this context length.

## FP8 Quantization Impact

The post emphasizes that **FP8 quantization significantly enhances efficiency**, enabling practical deployment of a 30B-parameter model like Qwen3-30B-A3B on a single RTX PRO 6000 Blackwell GPU. Even under a 450W power cap, the model delivers impressive aggregate throughput of **115.5 tok/s at 256K context with 10 users**, a figure the community describes as “wild.”

This performance suggests that FP8 is not only viable but competitive for inference tasks involving long contexts, especially in scenarios where latency is less critical than throughput.

## Visuals and Community Engagement

The original post includes a detailed performance chart (linked in the Reddit submission) that visualizes throughput vs. context length and user concurrency. The image has been referenced in multiple comments, with users praising the clarity of the data and calling it one of the most compelling benchmarks they’ve seen for local LLM deployment.

## Conclusion

While unverified by official channels, the r/LocalLLaMA post offers a compelling snapshot of what’s possible with modern quantization and efficient inference frameworks. For developers considering running large models locally, these benchmarks highlight the potential of **FP8 + vLLM + RTX PRO 6000 Blackwell** to deliver usable performance — even at the extreme edge of context length.

As NVIDIA’s Blackwell architecture continues to mature and tooling improves, community-driven testing like this will play a crucial role in shaping real-world expectations for local AI inference.

> *Note: Always validate benchmarks with your own hardware and software configuration. Performance can vary based on drivers, power management, and system load.*
