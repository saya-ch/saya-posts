---
title: OpenAI 流水线挖出 948 个 Linux 提权漏洞，753B 科学模型以 MIT 开放
date: 2026-11-06T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 本期最重的一条是 OpenAI 的自主漏洞流水线：一次运行复现 1,724 个 Linux 内核问题、产出 948 个本地提权利用，API 成本 121.3 万美元。模型侧，753B 的 Hydrogen-1.0 以 MIT 开放，在科学基准上压过 GPT-6 Astra；Ripple-65B 用全稠密架构和 3,200 万美元算力在 Terminal-Bench 2.1 拿到 80.5。工程侧，Paperclip 以 11 万星把多 Agent 管成组织，Worktrunk 与 Headroom 则从工作树隔离和上下文压缩解决并行开发的脏活。这些数字指向同一件事：能力之外，成本、隔离与可管理性正在成为新的比较维度。想核对每条细节与原始链接，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. OpenAI 自主漏洞流水线：复现 1,724 个内核问题，产出 948 个本地提权利用**
OpenAI 11 月 4 日公布 Patch the Planet 的一次完整运行：流水线先筛出 2,308 个 Linux 内核候选，动态复现 1,724 个，最终为 948 个本地提权漏洞生成可用利用与补丁，API 成本 121.3 万美元；7 月的 GPT-5.6 Sol 在同一流程下只找到 166 个，成本 51.4 万美元。团队还在 kernelCTF 环境拿到 25 个信息泄露 PoC 与 21 个提权利用，112 小时花费约 3.29 万美元，并称更强模型能在同一份代码里发现旧模型漏掉的问题。所有报告经人工复核后提交维护者，部分已进入上游修复，这说明“AI 找漏洞”已经变成可计价的工程流水线，而不是演示。
[原文](https://openai.com/index/a-thousand-fewer-paths-to-root)

**2. Hydrogen-1.0：753B 科学模型以 MIT 开放，科学基准 38.3% 压过 GPT-6 Astra**
杭州深度原理 11 月 2 日发布 Hydrogen-1.0，以 Z.ai 的 GLM-5.3 为底座继续后训练，总参数 753B、推理时激活约 40B，支持 100 万 token 上下文与 12.8 万 token 输出。官方在 FrontierScience-Research 上报 38.3%，高于 GPT-6 Astra 的 33.3%，GPQA-Diamond 为 93.0%；权重以 MIT 许可放在 Hugging Face，提供 BF16、NVFP4 与 MXFP4 三个版本，参考部署用 SGLang 跨 8 卡张量并行。它在 NMR 结构解析等项目上仍然落后，说明垂直后训练能换来领域高分，却换不来全面领先；对实验室而言，可下载的 753B 权重依然意味着企业级 GPU 集群。
[原文](https://www.marktechpost.com/2026/11/02/deep-principle-hydrogen-1-0-open-weight-scientific-model)

**3. Flower Labs Endeavor 1.1：综合智能 97.18 登顶，欧盟境内推理可自托管**
Flower Labs 11 月 3 日发布 Endeavor 1.1，首次支持图像输入，并在全球云之外增加“仅欧盟 GPU”推理档与私有云、本地部署选项。官方给出的综合智能分为 97.18，高于 GPT-5.6 Sol 的 96.25、Claude Fable 5 的 95.30 与 Kimi K3 的 94.83；分项是 GPQA 94.4、AIME 2026 99.9、HumanEval 99.9、IFEval 94.5、MMMU-Pro 81.12，较 1.0 的 GPQA 92.0、HumanEval 98.2 有明显提升。用户可查看思维轨迹并保留输出用于训练自己的模型，公司还计划推出英国专属推理与中东、亚洲、非洲区域集群。对数据必须留在境内的团队，这提供了一个不依赖美国 API 的前沿模型选项。
[原文](https://flower.ai/blog/2026-11-03-introducing-endeavor-1.1)

**4. Joules-to-GDP：Gemma 4 31B 用 23% 的电拿到 GLM 5.2 56% 的价值**
斯坦福、伯克利与 UT Austin 11 月 4 日发布 Joules-to-GDP，把 AI 产出折算成美元净值再除以耗电，测试 5 个开放模型、220 个来自 GDPval 的专业任务，覆盖 44 种职业。结论有三：GLM 5.2 单任务净值最高，但 Gemma 4 31B 只用 23% 的能量就拿到 56% 的净值，每焦耳净值约为前者 2.5 倍；GPT-OSS 120B 与 GLM 5.2 价格、速度接近，耗电却少 68%；同一 Qwen3.6-27B 在 B200 上每兆焦净值 535.5 美元，在 MI350X 上只有 124.1 美元。把 Qwen3.6-27B 从 1 张 H100 扩到 8 张，任务时间只快 22%，能耗却涨 3.2 倍——扩容并不等于效率，选型时该把“每焦耳能换多少产出”单独列一栏。
[原文](https://snorkel.ai/blog/joules-to-gdp-measuring-the-economic-value-of-ai-per-unit-of-energy)

**5. Ripple-65B Preview：65B 稠密模型用 3,200 万美元算力，Terminal-Bench 2.1 拿 80.5**
前谷歌研究员创办的 Ricursive Intelligence 11 月 4 日放出 Ripple-65B Preview：65B 全稠密、80 层（64 层滑动窗口加 16 层全局注意力）、100,352 词表，在 12 万亿 token 上从零预训练，Apache 2.0 许可。它主打智能体编码，Terminal-Bench 2.1 报 80.5，略高于 Reflection Beam 的 80.1 与 Meta Muse Spark Open 的 76.4；SWE-bench Pro v2.0 Hard 为 79.1，高于 Beam 的 76.5。代价是 DeepSWE 1.1 只有 44.3，落后 Muse 的 60.0，知识类基准也不占优；官方称训练算力约 3,200 万美元，是上一代开放权重 SOTA 的十分之一。权重含 BF16、NVFP4 与 base 版，但 OpenRouter 目前只标注 16,384 token 上下文，与模型卡的 100 万 token 不是一回事。
[原文](https://huggingface.co/ricursiveAI/Ripple-65B-Preview)

## 开源与工程

**6. Paperclip 冲到 11.1 万星：把一群编码 Agent 管成一家公司**
本周 GitHub 趋势榜首的 paperclipai/paperclip 已积累约 11.14 万星、1.84 万 fork，MIT 许可，TypeScript 编写，要求 Node.js 24.11 以上。它把 Claude Code、Codex、Cursor、OpenCode 等运行时统一成“员工”，用组织架构、汇报线、工单、线程会话和按 Agent 的月度预算来管理，预算超限会硬停；Process 适配器把本地 CLI 当子进程，HTTP 适配器接远端 Agent。项目 3 月上线，5 个月内到 7.7 万星，本周突破 11 万，说明多 Agent 的痛点已经从“能不能跑”变成“谁来管、花多少”。
[原文](https://github.com/paperclipai/paperclip)

**7. Worktrunk：给每个并行 Agent 一份独立工作树，Rust CLI 单日涨 438 星**
max-sixty/worktrunk 是一个用 Rust 写的 Git worktree 管理 CLI，目标很具体：同时开多个编码 Agent 时，让每个 Agent 在独立目录、独立分支上工作，避免互相覆盖未提交改动。它把 worktree 的创建、切换、清理以及与 tmux、Agent 会话的绑定收进一条命令，支持 Claude Code、Codex 等常见运行时；项目约 1.1 万星、375 fork，11 月 4 日单日新增 438 星、冲到 GitHub Trending 第 2。对已经习惯“一个功能开一个 Agent”的团队，这比再买一张显卡更直接地解决并行化。
[原文](https://github.com/max-sixty/worktrunk)

**8. Headroom：把 5.6 万 token 的 Agent 提示压到 2.4 万再发给模型**
headroomlabs-ai/headroom 已有约 7.89 万星，做的是 Agent 的“上下文压缩层”：在工具输出、日志、RAG 片段、文件和对话历史进入模型前先压缩。官方示例把 55,957 token 的提示压到 24,340 token 再发送，并强调关键行（如第 67 条的 FATAL）逐字节保留；编码 Agent 约有 20% 的 token 节省，JSON 类内容的压缩率为 60% 至 95%。它支持库、代理与 MCP 三种接入，压缩在本地完成、不上传内容。对按 token 计费的团队，这比换一个更便宜的模型更可控，也更容易逐项验证答案有没有被压坏。
[原文](https://github.com/headroomlabs-ai/headroom)

**9. Olla v0.1.0：单二进制把 Ollama、vLLM、SGLang 合成一个推理池**
Olla 发布第 25 个版本 v0.1.0：单个 Go 二进制或容器，不需要数据库，把 Ollama、vLLM、SGLang、llama.cpp、LM Studio、MLX 等运行时聚合成一个端点，对外提供 OpenAI 兼容与 Anthropic 兼容接口。它支持故障转移、粘性会话以保住 KV cache 局部性、端点健康检查、请求统计、Prometheus 指标和内置面板；对多机多卡、模型分散在不同机器上的自托管团队，省去的是逐个改客户端配置和手工切流的时间。生产环境仍要自己补鉴权、限流与网络隔离，别把一个内网工具直接暴露出去。
[原文](https://www.reddit.com/r/LocalLLaMA/comments/1xmq913/olla_v010_lightweight_opensource_llm_proxy_for)

*本期由站点自动生成 · 信息以原文为准。*
