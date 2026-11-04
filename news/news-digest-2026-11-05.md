---
title: Anthropic 单卡生成推理证明，4B 循环模型反超 9B
date: 2026-11-05T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 本期最硬的一条来自 Anthropic：FlashProve 在单张 H100 上为 400B 的 Llama 4 Maverick 生成零知识推理证明，速度约 110 token/s，约 19MB 的证明可在 CPU 上十几秒验完，代码与 Lean 证明全部开源。模型侧，Dory1.0-4B 用循环深度在 12 项基准平均分上压过 Qwen3.5-9B，ARLoop 则用 0.5T token 训出可追平 2B 模型的 1.2B 全开放循环模型。工程侧，Memdoor 让 5 个 Qwen3.8-27B 组队在 LiveCodeBench Hard 上压过 Fable 5 单次，vLLM 与 SGLang 接连发版把 MoE 解码吞吐再推高一截。这些进展的共同点，是能力不再只靠堆参数，而靠可验证的计算、可复用的循环深度与可编排的工作流。想核对具体数字与原始链接，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Anthropic 开源 FlashProve：给 400B MoE 推理附上 19MB 零知识证明**
Anthropic 11 月 2 日公开 FlashProve 原型：为 Llama 4 Maverick（400B 总参数 MoE、每 token 约激活 17B）生成零知识推理证明，单张 H100 约 110 token/s，1,024 token 请求的证明约 18.78MB；验证端不需要权重，24 线程 CPU 约 17 秒验完。官方给出的开销是 vLLM 原生推理的 72-94 倍，离线承诺约 39 分钟、可覆盖约 1,600 万次请求；它针对的是路由服务偷换模型、token 流被篡改时用户无从验证的问题，代码、Lean 证明与协议文档均以 Apache 2.0 开放。
[原文](https://www.anthropic.com/research/provable-inference) · [代码](https://github.com/anthropics/FlashProve)

**2. Dory1.0-4B 开源：4B 循环模型两次循环即压过 Qwen3.5-9B**
11 月 2 日上传的论文给出 Dory1.0-4B：4B 潜循环模型，单 checkpoint 可调推理深度；在 6T token 上预训练（前 4.5T 用 NVFP4），上下文扩到 256K，SFT+RLVR 后用难度感知循环课程把难题路由到更深循环。两次以上循环时，它在 12 项基准的平均分超过 Qwen3.5-9B、Gemma 4 12B 与 Gemma 4 26B-A4B；四次循环在 agentic 基准上比单次少生成 1.3-2.5 倍 token。这条路线把“参数量”换成“循环次数”，让同一份权重按任务难度分配算力；BF16、MXFP8、NVFP4 与 GGUF 权重全部开放，KV cache 只保留最后一循环。
[论文](https://arxiv.org/abs/2611.03998) · [vLLM 已支持](https://github.com/vllm-project/vllm/releases/tag/v0.33.0rc3)

**3. ARLoop 把循环 Transformer 推到 1.2B：0.5T token、三次循环追平 Qwen3.5-2B**
11 月 3 日的 ARLoop 论文把循环 Transformer 的残差改成对模型深度做注意力：每层输入从历史表示里选择性聚合，滑动窗口深度注意力在更大循环深度上不输全历史。作者在 140M/370M、R=1-5 上稳定超过旧残差设计，再扩到 0.6B/1.2B 真实预训练；1.2B 版本只用 0.5T token、三次循环，就在全部评测任务上超过 Huginn-3.5B，并与 Qwen3.5-2B 相当。对想把循环深度当作新缩放轴的团队，这提供了一组可复现的对照。
[原文](https://arxiv.org/abs/2611.05234)

**4. Figma 公开设计编辑模型配方：70B 模型 25 万样本打底，SFT 后转 RL**
Figma 10 月 29 日公开设计编辑模型的训练配方：早期用 Llama 3.3 70B 在超过 25 万个合成编辑样本上做 SFT，就能在画布编辑任务上接近当时顶级模型；现在基座换成 Qwen、Muse Spark Open、Inkling 等，并在 SFT 后加入 RL。训练管线用确定性校验器和视觉模型评审筛样本，评估同时看指令遵循、改动可用性、速度与成本。Figma 的结论是，设计编辑这类难以自动验证的任务，可以先用合成数据教会格式，再用 RL 对齐设计师偏好，垂直后训练足以和通用大模型正面竞争。
[原文](https://www.figma.com/blog/training-a-figma-fluent-editing-model/)

**5. Tokenizer Myths：BPB 不是 tokenizer 无关指标，0.989 的相关性修偏后只剩 0.057**
11 月 3 日的论文拆了三个常见结论：在未训练模型上，BPB 已经会偏好某些 tokenizer 与词表；BPB 更低并不等于下游更好，真实模型上二者只是弱相关；被称为 tokenizer 无关的 unigram-normalized loss 与 BPB 的 0.989 相关性来自混淆因子，修正后只剩 0.057，同一批数据在不同模型规模上会选出不同词表。作者由此提出“tokenizer 三难”，提醒选指标本身就是在选研究问题；对做中文词表或多语言分词的人来说，跨模型比较 BPB 的旧结论需要重算。
[原文](https://arxiv.org/abs/2611.04215)

## 开源与工程

**6. Memdoor 用 5 个 Qwen3.8-27B 组队：LiveCodeBench Hard 前 10 题均值 9.8，压过 Fable 5 单次 8.0**
10 月 31 日登上 Hacker News 的 Memdoor 把编码 Agent 做成 Go 单二进制工作流，Apache 2.0。它复刻 GVS5H 的提示词与循环，让 5 个 Qwen3.8-27B 各跑 5 次：LiveCodeBench Hard 前 10 题均值 9.8/10，原作者 harness 为 9.6，Fable 5 单次为 8.0；作者在完整 100 题上报出的对照是 92.4 与 90.4。50 次运行的请求、回复、程序与评分输出全部公开，可接 Ollama、llama.cpp、vLLM 或云端 key。它想证明的不是小模型单项更强，而是工作流可以把多次采样组织成稳定的上限。
[原文](https://github.com/guregodevo/memdoor)

**7. vLLM 0.33.0 一次并入十余个新模型：927 个提交，DSpark 把吞吐从 1,373 拉到 3,390 token/s**
11 月 2 日放出的 vLLM v0.33.0（标签 v0.33.0rc3）含 927 个提交、389 位贡献者，其中 122 位是新人；新增 Reflection Beam-1.0（501B MoE、23B 激活）、Step-5 Preview、Dory1.0-4B 等十余个模型。DFlash/DSpark 不再预留 draft 槽位后，DeepSeek-V4.1-Flash + DSpark 在 GB300 上从 1,373 升到 3,390 output token/s；FLy 有损校验策略带来 1.04-1.31 倍解码吞吐，CUDA 12.9 的 wheel 正式停发；这一版把 wheel 与镜像统一到 CUDA 13.0，旧环境的升级路径需要提前确认。
[原文](https://github.com/vllm-project/vllm/releases/tag/v0.33.0rc3)

**8. SGLang 0.5.23 合并 1,007 个 PR：单用户解码最高提升 75%，MoE 首 token 提前 30%-58%**
11 月 2 日发布的 SGLang v0.5.23 合并 1,007 个 PR、来自 257 位贡献者，新增 Muse Spark Open、Beam-1.0、MiniMax-M3.1-Flash 等 10 个模型。NVIDIA megakernel 与 AMD monokernel 让 DeepSeek-V4.1-Flash 的单用户解码在 GB300 提升 44%、MI350X 提升 75%；MoE 模型在 8K 提示上首 token 提前 30%-58%；Paged Experts 让超过显存的 MoE 在单张消费卡上运行，比 CPU offload 快 30 倍，并新增 TPU7x 支持。对自建推理服务的人来说，这些数字比模型榜单更直接：同样的卡，能多接多少并发。
[原文](https://github.com/sgl-project/sglang/releases/tag/v0.5.23)

**9. Heretic 2.0 拆掉配置兼容：3.9 万 star 的“免训练改模型”工具转向插件化**
11 月 1 日的 Heretic 2.0.0 是一次破坏性更新：1.x 配置不再兼容，需要按迁移指南重写；研究功能被移除，作者建议改用 Lophius；新增通用插件系统、headless 运行与端到端测试、benchmark scorer，以及可复现插件属性。这个“免训练改模型”项目现有约 3.9 万 star，继续用定向消融处理拒绝、啰嗦与谄媚等行为。这类工具把“改行为”从重新训练变成一次优化搜索，也让模型许可与安全责任更多落在使用者身上。
[原文](https://github.com/p-e-w/heretic/releases/tag/v2.0.0)

*本期由站点自动生成 · 信息以原文为准。*
