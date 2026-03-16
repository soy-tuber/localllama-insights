# Benchmarking LLM Inference Backends: vLLM, LMDeploy, MLC-LLM, TensorRT-LLM, and TGI

## Introduction

In the rapidly evolving landscape of large language model (LLM) deployment, selecting the right inference backend can significantly impact performance, scalability, and cost efficiency. To support developers in making data-driven choices, the BentoML engineering team recently conducted a detailed benchmark study comparing five leading LLM serving frameworks: **vLLM**, **LMDeploy**, **MLC-LLM**, **TensorRT-LLM**, and **Hugging Face Text Generation Inference (TGI)**. The evaluation focused on Llama 3 models and was carried out on **BentoCloud**, a platform designed for low-latency, high-throughput AI inference.

This article presents the key findings as reported in the community post from [r/LocalLLaMA](https://www.reddit.com/r/LocalLLaMA/), which received **50 upvotes** and sparked active discussion among developers and AI engineers. While the results are valuable for reference, readers should treat them as unverified community observations until independently validated.

## Methodology

The benchmark tested each framework’s ability to serve Llama 3 models under varying user loads, emphasizing decoding throughput — measured in **tokens per second (tok/s)**. Performance was measured across different configurations, including single-user and multi-user scenarios, with attention to resource utilization and latency.

Detailed technical specifications, such as hardware setup and model quantization methods (e.g., FP8), were preserved as reported in the original study. The evaluation aimed to provide a fair comparison by standardizing the environment and workload as much as possible.

## Key Results

According to the post, **LMDeploy** emerged as the top performer in decoding efficiency, achieving up to **4000 tokens per second** when serving 100 concurrent users. This result positions LMDeploy as a strong candidate for large-scale, interactive applications requiring fast token generation.

Other frameworks also demonstrated competitive performance, with trade-offs in latency, memory usage, and ease of deployment. For instance, **vLLM** was noted for its efficient memory management and strong performance in batch inference, while **TensorRT-LLM** leveraged NVIDIA’s optimized inference engines to deliver low-latency responses in specific hardware configurations.

**MLC-LLM** and **TGI** showed reliable performance, particularly in cloud-native and containerized environments, respectively. Each backend brought unique advantages depending on the use case — whether prioritizing speed, simplicity, or integration with existing ecosystems.

## Visual Summary

A comparative chart (linked in the original Reddit post) visually summarizes the results, highlighting LMDeploy’s lead in multi-user decoding scenarios. The image, sourced from a BentoCloud technical preview, includes metrics such as tokens per second per user, startup time, and memory footprint across the five frameworks.

## Conclusion

The benchmarking study underscores the importance of selecting an inference backend aligned with specific deployment goals. While LMDeploy currently leads in multi-user token generation speed, other frameworks offer compelling alternatives based on operational requirements.

Developers evaluating LLM serving solutions should consider factors beyond raw throughput, including model compatibility, ecosystem support, and deployment complexity. The full methodology and results are available in the detailed blog post by BentoML: [Benchmarking LLM Inference Backends](https://bentoml.com/blog/benchmarking-llm-inference-backends).

*This summary is based on unverified community information from a Reddit post with 50 upvotes. Always validate critical performance metrics through independent testing.*
