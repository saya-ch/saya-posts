---
title: Mistral 放出 1T 参数开源旗舰，MIT 团队让大模型直接改写网络权重
date: 2026-11-05
type: news
tags: [AI, 开源]
status: published
summary: 本期主线是两条「权重」消息。Mistral 10 月 29 日开放旗舰 Large 4 全部权重，1 万亿参数 MoE、1M 上下文，直接可下载。MIT 团队则让 GPT-6 Astra 不靠梯度下降，只改 Qwen 小模型 0.016% 的权重，MATH-500 就从 7.9% 提到 48.1%。工程侧，Google 把 Gemini 3.5 Flash 与 Omni 推向企业，Gravitee 4.13 一套网关统一模型协议。从能力开源到生产落地都在提速，点开正文逐条核对原文。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. MIT 团队展示 Weight-Space Programming：GPT-6 Astra 直接读写神经网络权重**
四位 MIT CSAIL 研究者 11 月 2 日报告：GPT-6 Astra 不用梯度下降，仅改 Qwen3.5-0.8B 权重的 0.016% 就把 MATH-500 从 7.9% 提到 48.1%，指令遵循提升约 3 倍，修复卡死的小车策略只动 3.5% 权重即达 100% 成功率。
[原文](https://www.alphaxiv.org/abs/2611.weight-space-programming)

**2. Mistral 开放旗舰 Large 4 权重：1 万亿参数 MoE，主打编码智能体与网络防御**
10 月 29 日权重上线 Hugging Face，原生多模态、1M 上下文，提供 FP8 1.06TB 与 NVFP4 610GB 两套检查点，官方称其为开源模型中 Coding Agents 与 Cyber Defense 的最强者，距预览仅 23 天。
[原文](https://www.linkedin.com/posts/pierrestock_weve-just-released-the-weights-of-mistral-activity-7523050815712808961-MXMd)

**3. Google Cloud 11 月盘点：Gemini 3.5 Flash 打头，Spark 个人 Agent 与 Antigravity 接入 Agent Platform**
11 月 3 日官方月度回顾确认：Gemini 3.5 家族首发成员为 Flash；Gemini Omni 主打任意输入生成任意输出，先从视频做起；Gemini Spark 面向 Gemini Enterprise 与 Workspace 用户 7×24 自主执行任务。
[原文](https://cloud.google.com/blog/products/ai-machine-learning/what-google-cloud-announced-in-ai-this-month)

## 开源与工程

**4. NVIDIA TensorRT-Edge-LLM 0.11.1：Nemotron 3.5 Bolt E4B 上车，Windows 与 JetPack 7.2.2 支持到位**
10 月更新的 0.11.1 新增 Bolt E4B 及其 DFlash draft、TensorRT RTX 的 Windows x64 支持；此前 9 月该引擎在 Jetson AGX Thor 上以 6.4 倍速度完成 MLPerf Edge Agentic 基准。
[原文](https://github.com/NVIDIA/TensorRT-Edge-LLM)

**5. MoE 分层推理引擎 colibri 冲上 GitHub 热榜：C 语言实现，4.7 万星**
热专家常驻 GPU（CUDA/Vulkan）、温专家放内存、冷专家从 NVMe 流式读取，让前沿 MoE 大模型跑在既有硬件上，11 月 3 日仍在持续更新。
[原文](https://github.com/JustVugg/colibri)

**6. Gravitee 4.13 发布：一套 AI 网关翻译所有模型协议，混跑不同版本 MCP**
11 月 4 日上线的能力覆盖 OpenAI、Anthropic 与自托管模型的协议互转、MCP 多版本混合路由、浏览器直连调用与 A2A 流量追踪。
[原文](https://www.gravitee.io/blog/one-ai-gateway-every-model-protocol-spec-version)

**7. Honeycomb 推出 AI SRE Agent：盯部署、抢在客户之前抓回归、偿还可靠性技术债**
11 月 3 日官宣的个人与团队 Agent 从高基数 trace 出发，从每次事故调查中学习，并且只在团队设定的权限边界内执行动作。
[原文](https://www.honeycomb.io/blog/honeycomb-introduces-ai-sre-agents)

**8. claude-mem 登上热榜：把 Agent 会话压缩成持久记忆，跨多款编码工具复用**
11.2 万星项目 11 月 3 日更新：自动捕获会话、用 AI 压缩并在后续会话注入相关上下文，支持 Claude Code、Codex、Gemini、Copilot、OpenCode 等工具。
[原文](https://github.com/thedotmack/claude-mem)

*本期由站点自动生成 · 信息以原文为准。*
