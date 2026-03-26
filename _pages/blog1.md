---
permalink: /blog/turboquant/
title: "Some Thoughts on the TurboQuant Buzz on Social Media"
author: Hao Wang
author_profile: true
layout: single
---

<img src="/images/blog1_image.png" alt="Is this another Pied Piper moment?" style="max-width: 500px; width: 100%;">

<p class="blog-date">March 2026</p>

Today, I've seen quite a bit of discussion on social media about a recent work from Google Research, TurboQuant (originally posted on arXiv on April 28, 2025, and recently accepted to ICLR). It is an interesting paper and as someone who works on information theory, I'm excited to see classical tools (e.g., converse bounds and vector quantization) being brought into modern LLM research (congratulations to the authors)!

That said, in my opinion, the work is still far from being practical in real-world systems. Below are some of the main concerns.

**1. High Latency**

The proposed method introduces non-trivial overhead due to quantization and dequantization (e.g., multiplication with rotation matrices and nearest-neighbor search). These operations take time and can significantly slow down inference.

In particular, during decoding stage, every new token requires dequantization before attention computation. For workloads involving long outputs, which is common in reasoning models and agent-based tasks, this latency becomes a major bottleneck. In such settings, the memory savings may not justify the performance cost, and overall throughput could end up being **much worse** than that of the unquantized model.

**2. Incompatibility with Existing Inference Engines**

Another major limitation is the lack of compatibility with widely used inference frameworks such as vLLM and SGLang. Most existing low-bit KV cache quantization approaches (including this one) do not integrate cleanly into these systems.

In practice, this means that deploying such methods can lead to significantly slower inference and lower accuracy than running the original model on optimized inference engines. Moreover, integrating this approach into vLLM introduces several non-trivial engineering challenges, such as efficiently storing and packing/unpacking a 3-bit KV cache, managing auxiliary variables (e.g., codebooks and rotation matrices) for dequantization, and designing specialized kernels that fuse dequantization with attention computation.

While integrating it into vLLM/SGLang is not impossible, it would require A LOT OF engineering effort and changes to the architecture of these inference systems. This is a bitter lesson I learned through several discussions with vLLM contributors while attempting to integrate my own work ([https://arxiv.org/abs/2503.24358](https://arxiv.org/abs/2503.24358)) into vLLM...

**3. Limited Evaluation on Challenging Tasks**

The experimental evaluation in the paper focuses primarily on information retrieval and long-context benchmarks (e.g., LongBench v1). These are relatively "easy" tasks for KV cache quantization because the prefilling stage still uses unquantized KV caches.

More realistic scenarios (such as reasoning, coding, and agentic tasks) are not explored. These tasks involve long response tokens, where quantization errors can accumulate and propagate over time. It remains unclear how well the method performs under these more challenging scenarios.

**4. Unclear Benefits with Modern Attention Architectures**

The paper evaluates its method on models using standard (vanilla) attention mechanisms, such as llama and mistral models. However, most recent open-source models have already adopted more efficient attention mechanisms (e.g., deepseek MLA, hybrid Mamba/gated delta net, or sliding window attention). These models already reduce memory and compute costs. It is therefore unclear how much additional benefit low-bit KV cache quantization provides when layered on top of these new architectures.

**5. Community**

Low-bit KV cache compression is not a new direction. As early as 2023, works such as H2O and attention sink already explored similar directions and demonstrated promising results.

The use of randomized transformations in quantization is also well-established. For example, QuaRot (a paper I personally like a lot!) provides an end-to-end framework and shows how to fuse randomized rotations into model weights.

More broadly, randomized mechanisms for reducing outliers are standard in the model compression literature, see e.g., QuIP, SpinQuant, and NestQuant.

**Final Thoughts**

To be clear, I enjoyed reading this paper. It is a thoughtful piece of research and a nice example of how theory can inform modern machine learning systems.

However, I do think that **some of the discussion on social media has overstated its practical impact and overlooked the broader body of existing work in this area**. As researchers, it's important that we evaluate new contributions in proper context and avoid overclaiming. I believe doing so ultimately benefits the entire community.
