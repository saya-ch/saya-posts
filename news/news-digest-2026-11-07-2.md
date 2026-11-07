---
title: 280B 的 ERNIE 放开权重，vLLM 0.33 解码快一倍
date: 2026-11-07T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 开放权重的竞争正在从“谁更大”转向“谁跑得更便宜”：百度 ERNIE 5.5 Flash Base 以 280B 总参数、14B 激活和 MMLU-Pro 78.4 分放出权重，Thinking Machines 的 Inkling 2 Small 则用 594B/17B 把 SWE-Bench Pro v2 Hard 推到 86.3%，但权重还要等一周。服务侧，vLLM 0.33 用 927 个提交把 GB300 解码压快 1.5–2 倍，Buoyant 的 Kinference 把中位 TTFT 从 6.88 秒降到 0.31 秒。AMD 阵营有 Zyphra 的 FP8 内核把 AITER 甩开 1.76 倍，Sarvam Prism 用一次前向把决策延迟做到 44ms，SumOne 则用 2B 小模型砍掉 70% 的 Agent 上下文成本。八条新闻里，三条模型、三条服务栈、两条上下文成本研究，指向的是同一个成本问题。真正拉开差距的，是权重之后那一层工程，逐条数字见正文。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 百度放开 ERNIE 5.5 Flash Base 权重：280B 总参数、14B 激活，MMLU-Pro 78.4**
百度 11 月 5 日放出 ERNIE-5.5-Flash-Base 的 FP8 与 BF16 权重，模型从零预训练 33.5T token，训练上下文 128K，微调与推理可开到 1M；43 层里 37 层用 HCA 把每 128 个 token 压成一个缓存条目，只有 6 层 MLA 保留 token 级访问。它在同管线对比中拿到 MMLU-Pro 78.4、HumanEval+ 82.3、LiveCodeBench 40.1，分别压过 DeepSeek-V4-Flash-Base 的 69.2、69.5 与 31.9；这套混合注意力把 1M 上下文的 KV 缓存压到较小规模；代价是自定义许可 v1.0 只对年营收 5,000 万美元以下免费，并排除欧盟、英国与韩国。
[原文](https://ernie.baidu.com/release/ernie-5-5-flash-base)

**2. Thinking Machines 的 Inkling 2 Small：594B/17B，SWE-Bench Pro v2 Hard 86.3%**
Thinking Machines 11 月 5 日发布第二代 Inkling 2 Small，594B 总参数、17B 激活，支持图文输入与 1M 上下文，Artificial Analysis 智能指数从上一代 13 跳到 31；官方称一次统一 RL 在 2,048 张 GPU 上跑了 9 天、1,075 步、超过 1,700 万次 rollout。它在 SWE-Bench Pro v2 Hard 拿 86.3%、MMMU Pro 81.6%、JobBench 70.5%，Terminal-Bench 4.0 却只有 13.1%，说明长程编码与终端操作仍是两件事；发布时权重仍闭源，只上 Tinker，Apache 2.0 要等下周外部红队结束；这两项落差提醒，基准分数必须绑定具体 harness。
[原文](https://thinkingmachines.ai/news/inkling-2-small)

**3. Sarvam Prism：105B 底座一次前向出决策，单卡 B200 中位 44ms**
Sarvam AI 11 月 3 日开源 Prism，把 105B 总参数、10B 激活的 MoE 底座做 LoRA 微调，不再逐 token 生成，而是对给定选项做一次前向、由打分头输出每个选项的概率；三项决策指数平均 71.8，单张 B200 上中位延迟 44ms，比 16 位双卡时的 664ms 快约 15 倍。它用 FP8 权重、单卡 CUDA Graphs 与合并适配器把延迟压下来，128K 上下文、约 120GB 显存即可跑；但自定义 FP8 布局导致它无法用标准 Hugging Face 工具或通用推理服务器加载，只能走 Sarvam 自己的代码，这既是性能来源也是锁定风险；它面向客服意图路由、模型路由与 Agent 结果复核等固定选项场景。
[原文](https://www.sarvam.ai/blogs/sarvam-prism)

**4. NVIDIA 研究：后段 token 贵 7 倍，短上下文压缩几乎不掉分**
NVIDIA 团队 11 月 3 日上传的论文测量了长上下文真实成本：在多个开放权重模型上，后段 token 的生成开销可比前段高出最多 7 倍，显存占用也随上下文线性上升。作者用频繁压缩把长程任务与长文档任务维持在短上下文，发现前沿模型准确率几乎不降；在长程软件工程任务里，即使每个上下文只允许执行一个动作，性能也没有崩溃，据此提出“语义状态空间模型”（SSSM）的方向。论文的结论相当直接：与其继续堆上下文窗口，不如把“记住什么”做成可训练、可压缩的状态管理；这也解释了为什么“无限上下文”在成本上并不成立。
[原文](https://arxiv.org/abs/2611.05306)

## 开源与工程

**5. vLLM 0.33.0：927 个提交，GB300 解码快 1.5–2 倍**
vLLM 11 月 1 日发布 0.33.0，含 927 个提交、389 位贡献者，升级到 PyTorch 2.14，发布构建只保留 CUDA 13.0；在 GB300 上官方测得解码延迟降低 1.5–2 倍、首 token 时间降低 39%–48%，ROCm 10.0 也成为默认。块扩散草稿器 DFlash/DSpark 不再为草稿预留 token 槽位，新增可选的 FLy 有损验证器与 /v1/decisions 端点，同时移除 Medusa 与 MLP speculator；这批改动说明推理框架的竞争已从“支持多少模型”转向“把每一段 KV 与每一纳秒调度都算清楚”；对自建集群，升级前要先确认自定义内核与 CUDA 13 的兼容性。
[原文](https://github.com/vllm-project/vllm/releases/tag/v0.33.0)

**6. Kinference：把 vLLM 包成 K8s 控制面，中位 TTFT 从 6.88 秒降到 0.31 秒**
做 Linkerd 的 Buoyant 11 月 4 日开源 Kinference，用 Rust 写成的 Apache 2.0 控制面，把 vLLM 当数据面，按 KV 缓存前缀与队列长度预测 TTFT 来选 Pod，并用 InferencePolicy 声明模型替换、PII 本地化与云上突发。官方在每池 3 张 L40S 的测试里称，共享前缀负载的中位 TTFT 从 6.88 秒降到 0.31 秒（-95%），端到端中位延迟降 45%，峰值 goodput 提升 7.4 倍、多轮对话提升 3.5 倍。它目前仍是 v0.1.0-alpha.7，限流、语义分类与 Anthropic API 兼容尚未落地，适合已有 vLLM 集群、想补一层多租户路由的团队；与普通 Service 轮询相比，它把“请求该去哪”从网络层挪到了应用层。
[原文](https://vllm.ai/blog/2026-11-04-introducing-kinference)

**7. Zyphra ZYFA：AMD MI355X 上的 FP8 注意力，比 AITER 快 1.16–1.76 倍**
Zyphra 11 月 5 日开源 ZYFA，一套面向 AMD Instinct MI355X（CDNA4）的 FP8/BF16 因果 prefill 注意力内核，Apache 2.0，要求 ROCm 7.2.3 与 PyTorch 2.12。它在 Mistral-7B、Qwen3-8B 与 DeepSeek-V2-Lite 的真实激活上做到 AITER 的 1.16–1.76 倍吞吐；32K token 的 FP8 测试达到 2,672 TFLOP/s，约为峰值的 53%，而 AITER 为 39%，head dim 128 下误差还低 62%–78%。代价是它只支持 MI355X，且 FP8 时钟会随数据分布下降，说明 AMD 软件栈的优化空间还很大；Zyphra 同时公开了论文与 HIP/汇编实现，便于其他团队复现。
[原文](https://zyphra.com/our-work/zyfa)

**8. SumOne：2B 压缩器把 Agent 上下文成本砍 70%，只掉 0.04 分**
本周挂在 arXiv 的 SumOne 是一个 2B 参数的即插即用上下文压缩器，用带掩码重建的强化学习训练，独立于具体任务、Agent 模型与 harness；它把不断增长的对话历史换成有界摘要，在 DeepSearch QA 与 SWE-Bench-Verified 上把端到端成本降低 70%，性能只下降 0.04。作者称同一套权重可迁移到 3 个领域、5 个下游模型（含 GPT-6）与 4 种编码 Agent harness，无需额外微调。对正在把长任务 Agent 推上线的人来说，这比再换一个更大模型更直接：先决定哪些历史值得留，再谈窗口大小；它的训练目标不是复述原文，而是保留后续决策需要的状态。
[原文](https://arxiv.org/abs/2611.06445)

*本期由站点自动生成 · 信息以原文为准。*
