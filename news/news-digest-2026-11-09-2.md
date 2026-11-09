---
title: 30B 多模态模型下放到本地，前缀缓存却在高峰塌陷
date: 2026-11-09T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头对准推理栈的两端。Meta 在 11 月 5 日放出 30B 的 Muse Glimmer，Apache 2.0、2B 视觉编码器加 28B 文本解码器，MCP Atlas 拿到 75.5，BF16 推理最低一张 80GB H100。另一端，两篇 11 月 6 日的论文拆开缓存：Nebula 把中位首字延迟压了 61.4%，另一篇证明 200 会话的突发能让引擎命中率归零，而会话上限在 30/30 次实验里挡住了塌陷。工程侧，llama.cpp 0.9.0 补上 FP8 E4M3，Ollama 解除 recurrent 模型并发限制，Kinference 把 KV 感知路由做成 Kubernetes 控制面。模型变小之后，决定体验的正是缓存、路由与并发这些脏活，具体数字和原文链接都在正文。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Muse Glimmer 30B：Apache 2.0 的本地多模态模型，MCP Atlas 75.5**
Meta 在 11 月 5 日放出 Muse Glimmer：30B 稠密结构由 2B 视觉编码器与 28B 文本解码器组成，Apache 2.0 许可，主打本地智能体，三层滑动窗口加一层全注意力交替 13 次，GQA 让每个 KV 头被 16 个查询头共享、把 KV 缓存压到 1/16，自带的 DFlash 草稿模型以 16 个 token 为一块、在 llama.cpp 里把投机步数上限卡在 15。官方榜单上 MCP Atlas 75.5、SWE-Bench Verified 76.0、OSWorld-Verified 65.9，BF16 推理与评测的最低配置是一张 80GB H100，量化后还能再往下探，transformers、llama.cpp 与 vLLM 均提供 day-0 支持，权重、GGUF 量化版与投机解码草稿模型同步上线，省去自己转换的步骤。
[原文](https://huggingface.co/blog/muse-glimmer)

**2. Nebula：把位置无关缓存做成分布式一等公民，首字延迟降 61.4%**
11 月 6 日提交的 Nebula 是第一个把位置无关缓存（PIC）段落当作一等公民的分布式推理框架：它用乐观记账加事件驱动更新维护集群视图，靠 ghost-hit 反馈自动在传统前缀缓存与 PIC 之间分配共享容量，再把排队、计算与 KV 传输折进统一的 prefill 完成时间模型。论文在 4 个模型、4 类 RAG 与智能体负载上测得中位首字延迟平均降 61.4%，并找回前缀缓存漏掉的 76.1% token；对 RAG 与多轮智能体这类反复复用长前缀的负载，收益主要来自少算而不是少传。
[原文](https://arxiv.org/abs/2611.09474)

**3. 前缀缓存会“塌陷”：200 会话突发让命中率归零，会话上限 30/30 次挡住**
11 月 6 日的《Inside the Trap》给出另一种失败：前缀缓存会在突发下塌陷。在 SGLang 0.5.22 与 vLLM 0.32.0、A100 与 H200、Qwen2.5-7B 与 Llama-3.1-8B 的注册对照实验中，200 个会话的突发让引擎在全部 30 次运行里命中率归零，而加会话上限的 30 次全部挡住，已经形成的 9 次塌陷也被同一上限救回 9 次；命中时一轮 prefill 约 551 token，全 miss 时约 11,100 token，差 16 倍，积压增长速度是超额负载的 0.6 到 0.8，这说明恢复阈值不等于塌陷态测到的吞吐，限流要按全 miss 容量来算。
[原文](https://arxiv.org/abs/2611.08902)

## 开源与工程

**4. llama.cpp 0.9.0 补齐 FP8 E4M3，统一 KV 多序列推理回血**
llama.cpp 在 11 月 4 日把版本号推到 0.9.0：这一版加入 OCP FP8 E4M3 张量类型（饱和模式）与转换工具，新增 flash_attn_ext_rows 算子，让统一 KV 在多序列推理下恢复非统一 KV 的 prefill 性能，同时补上 Clover、Limite、Metis-1.6、Mistral Large 4 四个架构，Chatterbox TTS、Nemotron 3.5 Transcribe 与 Pocket TTS 预置音色进入音频链路。底层 ggml 升到 0.29.0，带来 NCCL 多节点张量并行，以及 CPU、CUDA、Metal、Vulkan 等后端的性能修整，对本地多路推理来说，这两项改动比新增架构更直接影响吞吐。
[原文](https://github.com/ggml-org/llama.cpp/pull/31646)

**5. Ollama 0.40.15 解除 recurrent 并发限制，接入小米 MiMo V2.6**
Ollama 0.40.15 在 11 月 3 日转正，最实用的一条是 recurrent 与视觉 GGUF 模型终于能同时服务多个请求，此前这类模型只能串行排队。模型侧接入小米 MiMo V2.6（9B，面向代码与 Agent 工作流）和 AutoTrust 决策模型系列（9B、35B、27B-VL，输出是/否概率与评分，后两者可读图）；工具侧 ollama launch 支持 Kimi Code CLI 并迁移旧 kimi-cli 配置，同时自动补全 Pi 的模型列表、修正 Claude Desktop 的模型显示，对用消费级显卡跑本地模型的人，并发从串行变成多路是体验分水岭。
[原文](https://github.com/ollama/ollama/releases/tag/v0.40.15)

**6. GitHub Agentic Workflows 1.2.1：跨 Agent 续跑，安全审查失败即关闭**
GitHub Agentic Workflows 在 11 月 6 日发布 1.2.1，重点从“能跑”转向“跑完”：新增的 artifact-backed continuation 让工作流可以跨 Agent 续跑，规范感知的 PR 审查器会在改实现的同时更新 PR 分支上的规格文档，安全审查未解决时改为 fail closed，并对动态 checkout ref 发出警告。上一版 1.2.0 则把 pull_request_target 的 fork 使用改成显式 opt-in，属于会打断现有工作流的破坏性变更，升级前要先排查面向 fork 的流程，这类改动说明智能体工作流正从演示走向需要审计、恢复与权限边界的基础设施。
[原文](https://github.com/github/gh-aw/releases/tag/v1.2.1)

**7. NVIDIA 为四款韩系开放模型出 Dynamo 配方，B200 上最高 2 倍**
NVIDIA 在 11 月 8 日为四款韩系开放模型发布 Dynamo 部署配方：SK 电讯的 A.X K2、LG AI 研究院的 K-EXAONE 2.0、Upstage 的 Solar Open 2 与 Motif Technologies 的 Motif-3。配方针对 B200 调好请求路由与通信，官方称开箱即可拿到最高 2 倍性能，省去把模型切到多卡、再手工调并行的过程；Dynamo 本身是 Apache 2.0 的分布式推理框架，这次更新等于把“模型权重开放”往“能上生产”又推了一步；对想把区域开放模型留在自有云的公司，这类配方往往比模型权重更稀缺。
[原文](https://blogs.nvidia.com/blog/nvidia-dynamo-korea-open-models/)

**8. Kinference 0.1 alpha：用 5 个 CRD 把 KV 感知路由塞进 Kubernetes**
Buoyant（Linkerd 背后团队）在 11 月 4 日放出 Kinference，一个面向 Kubernetes 与 vLLM 的开源推理控制面：Rust 编写、Apache 2.0、用 5 个 CRD 描述网关后端、推理池、策略、提供方与 worker 集，监听 vLLM 的 KV 事件与引擎指标，按缓存重叠和排队长度预测 TTFT 再决定请求去哪个 worker。自报基准用 3 张 L40S、vLLM 0.31.0 与 Qwen3.8-27B-FP8：共享前缀负载的中位 TTFT 从 6.88 秒降到 0.31 秒（约 -95%），峰值 goodput 从 14.0 升到 103.0 请求/分钟，但没有共享前缀时几乎没有收益，项目也仍是 v0.1.0-alpha.7，它把网关、调度与租户计量从自研组件变成声明式配置，代价是仍要跟着 vLLM 与 Gateway API 的版本走。
[原文](https://github.com/Kinference-AI/kinference)

*本期由站点自动生成 · 信息以原文为准。*
