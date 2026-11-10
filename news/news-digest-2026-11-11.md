---
title: Gemini 4 Carbon 开放开发者，GR00T 2 五次示范到 95%
date: 2026-11-11T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期的主线是能力开放：Google 把 Gemini 4 Carbon 从网络安全小范围测试推向所有付费开发者，输入输出定价 2/10 美元，约为 Fable 5.2 的四分之一，并在 Antigravity 预览 agent 蜂群；英伟达在 CoRL 放出 GR00T 2，用 10 万小时人类视频把五次示范的成功率推到 95%。国内两条线同样值得看：百度 ERNIE Evolve 用自动研究改写训练脚本，在 NanoChat 与 NanoGPT 上超过公开最好成绩；美团把 1.6 万亿参数、百万上下文的 LongCat-2.5 以 MIT 开源。工程侧 RAGFlow 1.0 用 Go 重写后端，Hugging Face 则给决策模型定了统一接口。想看每条的具体数字与边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Gemini 4 Carbon 向所有付费开发者开放，Antigravity 预览 agent 蜂群**
Google 11 月 10 日把 Gemini 4 Carbon 从 Fairwind 计划的网络安全小范围测试推向 Gemini API、AI Studio、Android Studio、Gemini for business 与 Antigravity，官方称它是复杂长程任务最强的模型，Ultra 用户还可在 Gemini 应用和搜索 AI Mode 中使用；Antigravity 同步预览“蜂群”，把一条提示拆成一组协同 agent。定价为每百万 token 输入 2 美元、输出 10 美元，11 月 30 日后回到 4/20 美元，约为 Fable 5.2 的四分之一；Anthropic 口径下 Carbon 的 FrontierCode 2.0 为 41.1%、FrontierSWE v2 为 61.2%，低于 Fable 的 49.1% 与 69.9%，但 AutomationBench 以 53.9% 反超。
[原文](https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-4-carbon-now-available/)

**2. 英伟达 GR00T 2：10 万小时人类视频，五次示范成功率 95%**
英伟达 11 月 9 日在 CoRL 发布 GR00T 2，一个基于 Cosmos 的开放世界动作模型：它同时预测未来画面与机器人动作，训练数据含 10 万小时第一人称人类视频。官方测试中，50 个任务里有 40 个零样本至少完成一次，单次示范平均成功率 71%，五次示范升到 95%；配套的 Isaac Lab 3.0 接入 Newton，可模拟线缆与布料等可变形物体，视觉灵巧操作基准的吞吐比 2.3 版加 PhysX 高 2.5 倍。富士康、西门子与 Nexuni 已在试用，但都未宣布量产部署。
[原文](https://blogs.nvidia.com/blog/robotics-simulation-advancements-corl/)

**3. 百度 ERNIE Evolve：自动研究改写训练脚本，两项基准超过公开最佳**
百度文心 11 月 9 日发布自动研究系统 ERNIE Evolve，用 Director、Selector、Executor、Evaluator、Reviewer 组成双层循环，自主改代码、跑实验并按独立评测保留改进。它在 NanoChat AutoResearch 把验证集 BPB 降到 0.8763（社区公开最佳 0.8878），NanoGPT Speedrun Track 1 用 38.9 秒达标（39.9 秒），Track 3 用 2590 步（2690 步），SOL-ExecBench 为 0.819。接入 ERNIE-5.5-Flash 后，昆仑芯 P800 低时延生成提速到基线 623.6%，确定性推理吞吐三天提升 33.7%，极致解码再提 14.9%。
[原文](https://ernie.baidu.com/zh/research/posts/ernie-evolve/)

**4. 美团 LongCat-2.5 以 MIT 开源：1.6 万亿参数、百万上下文**
美团 11 月 10 日开源 LongCat-2.5：总参数 1.6 万亿、每 token 激活约 480 亿，支持 100 万上下文，是面向长程智能体的原生多模态模型，权重已上 Hugging Face，MIT 协议允许商用部署与二次训练。视觉侧用残差向量量化把图像变成多层级离散 token，与文本共享主干；推理采用 W4A4，把专家权重与激活同时压到 4 bit 并走原生 INT4 矩阵乘。针对单卡不足 80GB 的国产芯片，它用 FSDP 双重分片、混合词表并行与 MoonEP 动态专家复制，长序列 SFT 的单卡吞吐比 DDP 基线高 86%。
[原文](https://tech.meituan.com/2026/11/10/Meituan-LongCat-2.5-Open-Source.html)

**5. MiniMax H3-Image 开放权重：10 秒生成 4096×4096 图像**
MiniMax 11 月 10 日深夜发布并开放 H3-Image 权重，支持文生图、参考图生成与图像编辑，可在 10 秒内生成 4096×4096、约 1600 万像素的图像，官方称速度最高为主流方案的 5 倍。它最多可读取 16 张参考图，标准模式比同类快 1.5–2 倍、高速模式快 3–4 倍，API 定价为可比产品的 50%–70%，发布周 4K 及以上请求再打五折。开放权重意味着 4K 生图第一次能以本地或私有云方式部署，但 16MP 输出的显存与批处理成本仍要按实际负载核算。
[原文](https://www.minimax.io/blog/minimax-h3-image)

## 开源与工程

**6. RAGFlow 1.0 用 Go 重写后端：镜像小 10 倍，迁移不可逆**
RAGFlow 11 月 6 日发布 1.0 正式版，后端整体改用 Go 重写：Docker 镜像约小 10 倍、启动约快 10 倍、并发约高 100 倍，消息队列从 Redis 换成 NATS，缓存与检查点改用 Kvrocks，CLI 也重写为 Go。新的 Agentic Chat 取代原先 Low/Medium/High/Ultra 四档，通过多步检索、工具调用与证据充分性判断作答，配合 MiniMax M3 在 FRAMES 多跳问答上达到 91%。升级并不轻松：0.27.2 到 1.0 的数据迁移不可逆，旧 API 别名被移除，DeepDoc 暂时只支持 CPU。
[原文](https://github.com/infiniflow/ragflow/releases/tag/v1.0.0)

**7. Hugging Face 给决策模型定统一接口：输出 token 为 0**
Hugging Face 11 月 10 日为“决策模型”定下统一接口：输入 state 与若干带预设答案的问题，直接返回概率分布而不生成文本；Transformers 新增 DecisionPipeline、AutoModelForDecision 与 DecisionConfig，并提供 /v1/systemone 端点，llama.cpp 也支持同一 API；同一个 state 还能同时挂多个不同类型的问题。问题分 choice、score、noul 三类，choice 与 score 还会给出按选项数归一化的 confidence；官方示例里 GEV-26B-Decide 把重复扣款请求分到 billing 的概率约 0.998，退款判断 0.995，输出 token 为 0。接口统一了输出格式，但并不保证判断正确。
[原文](https://scopeai.cc/stories/hugging-face-%e4%b8%ba%e5%86%b3%e7%ad%96%e6%a8%a1%e5%9e%8b%e5%88%b6%e5%ae%9a%e7%bb%9f%e4%b8%80%e6%8e%a5%e5%8f%a3-transformers-%e4%b8%8e-llama-cpp-%e6%96%b0%e5%a2%9e-v1-systemon-mwau0zc6)

**8. 蚂蚁开源 LingBot-Touch：5000 小时触觉数据，拧瓶盖成功率到 90%**
蚂蚁灵波（Robbyant）11 月 10 日开源触觉基础模型 LingBot-Touch，用近 5000 小时真实触觉数据预训练，把磁性、视觉与压阻三种传感器的信号映射到统一的指尖坐标与触觉 token，并把人类戴触觉手套采集的数据与遥操作数据放进同一套表征训练。它在 26 项真实操作测试、两种灵巧手与四个动作模型上，把平均成功率提高 14–26 个百分点；拧瓶盖任务从无触觉的 56.25%、直接喂原始信号的 62.5%，提升到经 LingBot-Touch 处理后的 90%。轻量版仅 530 万参数，接入动作模型只增加约 0.13%–0.16% 的参数量。
[原文](https://interestingengineering.com/ai-robotics/robots-gain-touch-with-china-dataset)

**9. GitHub Agentic Workflows v1.2.1：跨 agent 续跑与更严的安全默认**
GitHub Agentic Workflows 11 月 6 日发布 v1.2.1 预发布版，自 v1.2.0 起累计 233 次提交，核心是让 agent 工作流更耐久：新增基于 artifact 的跨 agent 续跑（#75097）、规格感知的 PR 审查器（#75207）与实验性的 Atomic 声明式引擎（#75196）。repo-memory 支持 GitHub App 认证，工作流环境可配部署控制，更新工作流时会保留源引用与 frontmatter 格式；安全侧在 PR 审查未解决时默认失败关闭，并对动态 checkout ref 暴露宿主步骤给出警告，防火墙与 MCP gateway 分别升到 v0.29.7 与 v0.5.4。它仍是预发布，升级前要核对 workflow 锁文件与权限边界。
[原文](https://github.com/github/gh-aw/releases/tag/v1.2.1)

*本期由站点自动生成 · 信息以原文为准。*
