---
title: GPT-6 独立设计蛋白质，蚂蚁 124B 多模态模型以 MIT 许可开源
date: 2026-11-26T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期最重的一条来自 OpenAI：11 月 24 日的预印本显示，GPT-6 不依赖专用生物模型，只用推理和通用编程工具就能设计蛋白质三维结构与氨基酸序列，蛋白排列指标从 GPT-5.6 的约 0.05 升到 0.28，稳定性研究里 265 组样本的得分也从 0.41 升到 0.52。开源侧，蚂蚁把 124B 总参、5.5B 激活的 Ling-3.2-air 以 MIT 许可放出，SWE-bench Pro 从 26.40 升到 58.28；英伟达则开放 30B 图像模型权重，GenEval 拿到 92.86。三条消息分别指向通用模型切入科研、开放权重继续逼近闭源，以及图像生成进入可自托管阶段；工程侧同期的变化也说明，成本与运行时正在成为比分数更实际的变量。想核对每条的数字与原始出处，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. OpenAI 用 GPT-6 直接设计蛋白质，排列指标从 0.05 升到 0.28**
OpenAI 11 月 24 日发布预印本，称 GPT-6 系列不依赖 RFdiffusion 一类专用生物模型，仅凭推理与通用编程工具就能给出蛋白质三维骨架和氨基酸序列，并在多轮实验室测试中验证结合效果；蛋白排列指标从 GPT-5.6 的约 0.05 升到 GPT-6 的 0.28，稳定性研究的 265 组样本得分从 0.41 升到 0.52，GPT-6.2 为 0.57。团队同时承认，实验成功率仍落后于专用模型。这条路线把蛋白质设计从专用网络的专门任务，推向通用模型加工具链的通用能力；若复现成立，生物团队的门槛会从训练专用模型转为组织实验验证。
[原文](https://openai.com/index/gpt-6-models-demonstrate-intrinsic-protein-design-capabilities/)

**2. Krafton 开源 30B 的 Raon-1 Mini，每次只激活 4B**
Krafton 11 月 25 日发布自研 30B 级大语言模型 Raon-1 Mini，采用混合专家加自研线性注意力的架构，约 300 亿总参数中每个 token 只激活 40 亿，权重以 Apache 2.0 上传 Hugging Face；官方公布 MultiChallenge 64.9 分、GDPVal 72.3 分、BrowseComp Plus 55.9 分，主打低算力下的多轮指令跟随与代理任务。对游戏与内容公司来说，这意味着可以在自有数据上微调一个可本地部署的 30B 模型，而不必把推理全部交给外部 API。
[原文](https://www.krafton.com/news/press/%ed%81%ac%eb%9e%98%ed%94%84%ed%86%a4-%eb%8f%85%ec%9e%90-%ea%b0%9c%eb%b0%9c-%ea%b3%a0%ed%9a%a8%ec%9c%a8-llm-%eb%9d%bc%ec%98%a8-1-%eb%af%b8%eb%8b%88-%ec%a0%95%ec%8b%9d-%ec%b6%9c/)

**3. 商汤开源 8B 的 SenseNova U1.6，把生成、检查、改错压进一次出图**
商汤 11 月 24 日发布并开源 SenseNova U1.6，首次为系列 8B 开源模型加入原生图像自修正，把生成、检查与改错合并为单次出图；在 UltraTextBench 密集信息图测试中，开启自修正后的得分从 86.04 升到 92.28，提升 6.24 分、约 7%。权重已上 GitHub、ModelScope 与 ComfyUI，API 版本 U1.6 Lite 同步接入小浣熊。自修正的关键不在多画一遍，而在模型能判断哪块文字或结构出错，再只改错处；这决定了它能否用于海报、信息图和电商图等文字密集场景。
[原文](https://news.iresearch.cn/content/202611/574757.shtml)

**4. 英伟达开放 30B 图像模型权重，GenEval 92.86 压过 GPT Image 2**
英伟达 11 月 25 日在 Hugging Face 开放 Nemotron-Labs-Image-30B-A3B，采用混合专家架构，总参约 300 亿、推理时激活约 30 亿，配 6.42 亿参数视觉分词器，以自回归方式生成视觉 token；官方披露完整 VisualCreation 框架在 GenEval 拿到 92.86，高于 GPT Image 2 的 89.00，CVTG-2K 为 92.53、WISE Verified 为 91.50、BizGenEval 为 86.0，但单模型与完整框架的差距仍要回模型卡核对。值得注意的是，英伟达这次没有办发布会，权重先出现在 Hugging Face，再由媒体发现；这类静默发布正成为开放权重模型的常见节奏。
[原文](https://www.ithome.com/1/032/444.htm)

**5. 蚂蚁开源 Ling-3.2-air：124B 总参只激活 5.5B，MIT 许可**
蚂蚁集团 11 月 23 日在 Hugging Face 上线 Ling-3.2-air，124B 总参数、每 token 激活 5.5B 的原生多模态混合专家模型，以 MIT 许可给出 129GB FP8 权重、共 13 个分片，支持 256K 上下文与 SGLang 部署；模型卡称 SWE-bench Pro 从 Ling-3.0-flash-VL 的 26.40 升到 58.28，AutomationBench-AA 从 15.7% 升到 61%。5.5B 的激活量把每 token 的计算压到小模型级别，但 129GB 的 FP8 权重仍需要多卡或大显存机器，官方给出的 256K 配方就用了四张大显存 GPU。
[原文](https://pandaily.com/ant-group-ling-3-2-air-multimodal-open-weights)

## 开源与工程

**6. Anthropic 把 MCP 的 Tasks 与 Elicitation 转正，Skills 进公测**
Anthropic 11 月 24 日宣布 MCP 的 Tasks 与 Elicitation 正式可用，Claude 与 Claude Code 的 Pro、Max、Ultra 用户先拿到，Team 与 Enterprise 随后开放；Elicitation 会在工具调用中途暂停，用表单或外链收集用户输入再继续。Skills over MCP 同时进入公测，服务器可把可复用工作流说明随工具一起分发，Pro、Max、Ultra 默认开启，Team 需管理员启用，Enterprise 尚未支持。对工具开发者而言，Tasks 把长任务的状态管理交给协议层，Elicitation 则让需要用户确认的步骤不必退出会话；两项合起来，MCP 才更像一个能跑完整流程的接口。
[原文](https://claude.com/resources/articles/skills-over-mcp-tasks-and-more)

**7. 同一模型换 13 个 harness，单任务成本最多差 5.4 倍**
Artificial Analysis 11 月 24 日在 Coding Agent Index 2.1 上线 Harness Comparisons，把模型固定、只换编码代理外壳，用两个模型跑 13 个 harness；结果显示分数变化远小于成本，同一模型单任务成本最多差 5.4 倍、分数只差约 1.5 倍，Codex 配 GPT-6 Luna 排第一，配 GLM-5.3-Flash 却垫底，说明模型排行榜无法直接回答该选哪套代理运行时。这也解释了为什么同一模型在不同产品里体验差异明显：提示词、工具集、重试策略和上下文压缩都在成本里，而不只体现在榜单分数上。
[原文](https://artificialanalysis.ai/agents/coding-agents)

**8. Hermes Agent 0.23.4 收进 229 个 PR，补齐商店与云版本**
NousResearch 11 月 23 日发布 Hermes Agent v0.23.4，把 v0.23.3 以来的约 229 个 PR 汇成稳定版，同时覆盖 GitHub Release 的 CLI 与安装器、Microsoft Store 应用、Docker 镜像和 Hermes Cloud；完整的分类发布说明要等到 v0.24.0，届时会补上从 v0.23.0 起的功能区与贡献者名单。补丁版把分发渠道一次铺齐，说明代理工具正在从开发者脚本变成需要商店、容器和托管服务的常规软件。
[原文](https://github.com/nousresearch/hermes-agent/releases/tag/v0.23.4)

**9. Transformers 5.24 让 SigLIP2 一次处理整批图像**
Hugging Face 11 月 20 日发布 Transformers v5.24.0，SigLIP2 与 LFM2-VL 改为用张量运算一次处理所有图像，减少逐图循环带来的调度开销；同一版还修了 OmniASR 的 CTC blank token、Qwen3 Omni 音频参数、Qwen3-ASR 的 truncation 默认值，以及 PeAudioVideo 在 timm 1.0.31 下的加载问题。这类底层库改动不会出现在模型榜单上，却会直接影响多图推理的显存峰值与首 token 延迟，做视觉应用的团队值得跟进。
[原文](https://github.com/huggingface/transformers/releases/tag/v5.24.0)

*本期由站点自动生成 · 信息以原文为准。*
