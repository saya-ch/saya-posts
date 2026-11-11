---
title: Claude 三天移植 AMD 内核，Qwen Code 1.0 并进 Harness
date: 2026-11-11T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期的主线是“把适配与编排的成本打下来”。Anthropic 在 IPO 路演里披露，Claude Mythos Preview 约 3 天就让一款 Claude 模型在 AMD MI355X 上跑通提示词处理，GPU 内核几乎全由模型自己移植，而过去这类适配通常要 6 至 12 个月。工程侧，阿里把 Qoder 的 Harness 并进 Qwen Code 1.0 开源内核，vLLM 0.34 候选版把 Gemma 与 StableLM 交还 Transformers 后端，Google 则把训练 Gemini 用的单控制器运行时 Cloud Pathways 在 GKE 上开放。研究侧三篇论文分别处理对照集发现、CPU 推理缓存与扩散模型双稀疏。还有一起 SGLang 漏洞导致 NEAR AI 数据泄露的事故，提醒推理栈本身就是攻击面。逐条数字与边界见正文。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Claude 三天把内核搬上 AMD MI355X，人工只做少量干预**
Anthropic 在 IPO 路演材料中披露，Claude Mythos Preview 约用 3 天就让一款 Claude 模型的提示词处理在 AMD MI355X 上达到可用性能，所需 GPU 内核几乎全部由模型自行移植或重写；作为对比，过去接入新芯片平台通常要几十名工程师花 6 至 12 个月。MI355X 有 288GB 高带宽显存，ROCm、HIPIFY 与 Triton 已降低迁移门槛，但单个环节跑通不等于整机规模化稳定，Anthropic 同时建了横跨 NVIDIA、AMD、TPU 与 Trainium 的调度系统，并计划从 2027 年上半年起部署最高 2GW 的 AMD GPU。
[原文](https://www.ithome.com/1/025/736.htm)

**2. NeurIPS 论文：把假设换成对照集，智能体多解 67% 问题**
11 月 10 日上传 arXiv 的 Contrastive Discovery 把单点假设检验改成“对照集”：每个科学问题配一组互斥且穷尽的竞争解释，实验按证据让信念在集合上重新分配多少来打分，信念集中到一个解释时问题才算解决。六个领域里，单点候选发生最大信念移动的样本有 89% 只是被拒绝；同样证据下，对照集把其中 73% 落到具体替代解释；用对照集引导搜索，相同实验预算下多解 67% 的问题，38 位人类专家的盲评在已解问题上与系统一致率为 78%。
[原文](https://arxiv.org/abs/2611.14514)

**3. CPU-ARC：把推理轨迹编译成 CPU 缓存，调用便宜 16 至 291 倍**
11 月 6 日的技术报告提出 agentic reasoning caching：把大模型已经推通过的解题轨迹构造成可在 CPU 上执行的推理缓存 ARC，新问题实例直接套用，不再让模型从零重推。作者称 ARC 在一批困难的数学与科学任务上与前沿模型表现相当，完全可解释、可继续训练、对输入扰动稳健，单次调用成本比源模型低 16 至 291 倍，但边界也很清楚：任务必须能同构到可复用解法，开放式问题仍要模型现场推理。
[原文](https://arxiv.org/abs/2611.09891)

**4. DUET 给扩散语言模型做双稀疏，32K 上下文端到端快 1.34 倍**
11 月 6 日的论文针对块扩散语言模型的两份重复开销：历史前缀注意力随生成变长越来越贵，每个去噪步又把整个当前块送进 Transformer；DUET 免训练地复用每个生成块内集中且跨步稳定的历史 KV 子集，并用浅层表示预测本步会更新哪些位置，只把这些 query 送进深层。LLaDA2.1-mini 上端到端与单步加速为 1.34 倍和 2.83 倍，SDAR-8B-Chat-b32 在 32K 上下文为 2.41 倍和 4.72 倍，精度保持竞争力。
[原文](https://arxiv.org/abs/2611.09239)

## 开源与工程

**5. Qwen Code 1.0 把 Qoder 的 Harness 并进开源内核**
阿里 11 月 11 日发布 Qwen Code 1.0 Preview，把在 Qoder 中打磨的 Harness 能力并入同一套开源内核，让接入、调度与执行分层，推理编排和工具执行独立管理，从“一个进程、多个会话”走向“多个节点、更多会话”。模型服务走统一入口集中管理配置、授权与调用记录，沙箱按相同权限边界复用，MCP 的配置、授权与连接也统一治理；仓库给出 SWE-bench Verified 76.40% 至 77.80% 的成绩，条件是同一 Qwen 3.7 Max、500 个任务、每版三次；1.0 走独立分支，稳定分支保留。
[原文](https://github.com/QwenLM/qwen-code)

**6. vLLM 0.34 候选版把 Gemma 与 StableLM 交给 Transformers 后端**
vLLM 11 月 8 日打出 v0.34.0rc0，核心改动是把 Gemma 与 StableLM 两个模型系列迁到 Transformers 后端（PR #66393），由上游库负责模型定义与权重加载，vLLM 侧专注调度与显存管理。这样做能减少分叉、让模型行为更贴近参考实现，但专用融合算子与量化路径可能短期缺位，升级前要重新核对权重加载兼容性、输出数值一致性和吞吐/显存表现；rc0 只适合兼容性测试，不建议直接上生产。
[原文](https://github.com/vllm-project/vllm/releases/tag/v0.34.0rc0)

**7. Meta 开源用了八年的设计系统 Astryx，一天涨 3108 星**
Meta 把内部最大的设计系统 Astryx 以 MIT 协议开源，Beta 版基于 React 19+ 与 StyleX，含 150 多个无障碍组件、七套主题、CLI 与现成模板，官方称它已在内部支撑 13000 多个应用、打磨了八年；它的卖点是让人和 agent 用同一套工具：组件可以 swizzle 出完整源码，主题只是一组 CSS 变量覆盖，样式方案不锁定 Tailwind、CSS Modules 或原生 CSS，文档与 CLI 也一起为 AI 助手设计。11 月 11 日它以 23021 星、单日增加 3108 星登上 GitHub Trending 首位。
[原文](https://github.com/meta/astryx)

**8. Cloud Pathways 在 GKE 正式可用，90 天跑了 1.3 亿 TPU 芯片小时**
Google Cloud 11 月 9 日宣布 Cloud Pathways 在 GKE 上正式可用，把 DeepMind 训练 Gemini 用的单控制器运行时开放给所有客户：一个 JAX 客户端经 IFRT 代理管理多个 TPU 切片，支持同机 Python sidecar、编译缓存和基于主机内存快照的弹性训练，硬件故障时可缩到 N-1 个切片而不崩客户端。过去 90 天已有客户跑了超过 1.3 亿 Cloud TPU 芯片小时；目前支持 v5e、v5p、Trillium 与 Ironwood，要求 JAX 0.10.1 以上，Slurm 集成仍在 Preview。
[原文](https://www.hpcwire.com/aiwire/2026/11/09/google-cloud-releases-pathways-runtime-for-large-scale-ai-training-and-inference/)

**9. SGLang 远程执行漏洞击穿 NEAR AI 推理服务器**
NEAR AI 11 月 9 日披露，攻击者利用 SGLang 推理引擎的远程代码执行漏洞，在一台推理服务器上执行代码，读到当天经手的请求，包括 confidential 模式下的请求，并横向进入已下线的 Private Chat 内部系统：约 100 名用户的邮箱、姓名及部分套餐与余额信息被下载，18 个旧版 Agent 的聊天记录、设置与日志被读取，两名 Cloud 客户的 API key 出现在日志中，随后被吊销。公司已在自己服务器上修复漏洞、轮换凭据并恢复服务，但没有公布 CVE 编号、受影响版本，也未说明上游是否已发补丁；其复盘结论很直接：TEE 保护不了跑在里面的软件漏洞。
[原文](https://near.ai/blog/security-incident-report-sglang-vulnerability)

*本期由站点自动生成 · 信息以原文为准。*
