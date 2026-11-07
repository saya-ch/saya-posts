---
title: 560B 多模态只上 API，21K 美元训出 2B 文生图
date: 2026-11-08T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 本期最值得对比的是两条路线：蚂蚁 Ling-3.2-flash 把 560B 总参、25B 激活与 1M 上下文做成只租不卖的 API，前两周免费，权重仍未开放；Linum 则用约 21K 美元训出 2.18B 的 P-JiT，权重以 Apache 2.0 直接放出。工程侧，OpenRouter 重写 Auto 路由，Auto Code 在编码基准上与 Opus 5.5 持平却便宜 74%；llama.cpp 两天连发十多个构建，修掉可让服务退出的会话崩溃。研究侧，Anthropic 拆解 20.7 万次科研会话，发现数据分析占三分之一、写作远超自报时间，并估算现有能力可让科研产出加速 1.35–2.02 倍。当算力与入口越来越贵，开放权重和小成本训练还能换来多少主动权，逐条数字见正文。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 蚂蚁 Ling-3.2-flash 上线：560B 总参、25B 激活，1M 上下文限免两周**
11 月 6 日，蚂蚁百灵发布原生多模态 MoE 模型 Ling-3.2-flash：560B 总参数、25B 激活，上下文 1M，原生支持文本、图像与视频，重点强化复杂视觉理解、长程多模态 Agent、3D 与医疗场景。官方自测在 OmniDocBench 1.5（92.21 对 90.56）、ZEROBench（43.71 对 39.52）与 Vision2Web（85.86 对 80.20）三项超过 Claude Opus 5，DesignArena 图像转前端以 1238 Elo 进入前十。但截至 11 月 7 日只开放 API，兼容 OpenAI 与 Anthropic 协议，权重尚未放出；前两周免费，之后输入每百万 token 0.30 美元、输出 0.90 美元。上一代 Ling-3.1-flash 曾以 MIT 协议放出约 572GB 权重，这一代先 API 后开源的节奏因而格外受关注。对想迁移的团队，这是低成本试错窗口，也说明“560B”目前只能租、不能自己部署。
[原文](https://m.ithome.com/html/1023043.htm)

**2. Anthropic 拆解 20.7 万次科研会话：数据分析占三分之一，写作远超自报时间**
Anthropic 11 月 5 日发布报告，分析 8 月 2 日至 9 月 30 日的 207,049 次科研会话，其中 Claude 128,914 次、Claude Code 78,135 次，并配套调查 528 名科学家。数据分析占 Claude 会话 33.7%、Claude Code 48.1%，而科研人员自报只有 27.6% 的时间花在这上面；写作占 32.0%，是自报 16.0% 的两倍；想法生成仅 4.2%，低于 7.1% 的时间占比。样本只覆盖消费者套餐，任务分类依托 SciNet 的 7,262 项细粒度任务，因此更像“真实使用画像”而非能力评测。报告估算，广泛采用现有能力可释放 27%–52% 的科研时间，产出加速 1.35–2.02 倍；若自动化全部非物理步骤，计算机与统计可达约 15 倍，实验科学仍被约 40% 的物理步骤卡在 3–4 倍。
[原文](https://www.anthropic.com/research/how-researchers-use-claude)

**3. 清华与伯克利开源 uni-robot-agent：Qwen3.8-Flash-Next 在 RoboDojo 反超 GPT-6 Astra**
论文 11 月 1 日提交、3 日更新，28 位作者来自清华、伯克利等团队。这个统一 harness 让开放权重模型直接做机器人操作，并支持继续训练；在不给特权输入的条件下，Qwen3.8-Flash-Next 取得 RoboDojo 31.39 分，高于 GPT-6 Astra-RoboProbe 公布的 28.97 分，并在 19 个真实任务上有竞争力。作者把有效机器人操作拆成四项通用能力，认为后续训练应联合培养，而不是只堆视觉语言数据；harness 与基线一并开源，方便在相同输入下复现这组对比。
[原文](https://arxiv.org/abs/2611.01560)

**4. 《Route to Remember》：把长期记忆写进权重，每次查询读更少 token**
11 月 3 日提交的论文提出用 on-policy self-distillation 训练一个无上下文的“学生”，把用户事实从文本记忆迁到权重，再用 verify-then-regenerate 路由器决定查询走权重还是走检索。在 HaluMem 基准上，该方法匹配或超过全上下文提示与检索基线，同时每次查询读取的 token 更少。论文入选 NeurIPS 2026 PALM workshop，价值在于给出“权重记忆 + 文本记忆”混合的训练与评测协议，而不是宣称文本记忆可被完全替代。代价也明确：权重记忆更新要靠训练，文本记忆则更容易审计与删除，两者混用才接近生产要求。
[原文](https://arxiv.org/abs/2611.05636)

## 开源与工程

**5. 21K 美元训出 2.18B 文生图模型，权重以 Apache 2.0 开放**
Linum 11 月 3 日开源 P-JiT 21K：2.18B 像素空间 DiT，配 Qwen3.5-4B 文本编码器，在 660M 样本上训练，512px 阶段 610M、1024px 阶段 50M，总计算成本约 21K 美元。权重 fp32 共 8.7GB，文本编码器另占 8GB，需要 CUDA 显卡，支持五种约 1MP 画幅；采样用 50 步 Euler 与 guidance 15，官方明确它是研究检查点，不是生产模型，且训练时刻意滤掉文字图像，因此写不出海报与招牌。对预算有限的小团队，这份配方比“又一个排行榜”更有参考价值。
[原文](https://github.com/Linum-AI/pjit-21k)

**6. OpenRouter 重写 Auto 路由：p90 决策 0.65 秒，Auto Code 比 Opus 5.5 便宜 74%**
11 月 5 日，OpenRouter 上线新一代 Auto Router 与面向编码的 Auto Code。新路由每次请求评估 11 个因素，再结合实时价格、供应商健康度与缓存状态选模型，官方称基准表现最高提升 4 倍，p90 路由耗时从 1.96 秒降到 0.65 秒。Auto Code 在五项编码基准上与 Claude Opus 5.5 持平，单任务成本低 74%；测试期内近九成 Agent 轮次不换模型，以免重建缓存，无效工具调用只有此前的四分之一。平台每月处理 2.5 万亿 token、600 多个模型，路由正在变成一层可计价的中间件；对团队而言，选型也从一次架构决策变成每次请求的运行时决策。
[原文](https://openrouter.ai/blog/announcements/auto-and-auto-code-routers/)

**7. llama.cpp 两天连发十多个构建：修掉会话崩溃，补上语音端点**
11 月 4 日起，llama.cpp 连续发布 b12128–b12140 十多个构建。b12137 拒绝非对象形式的实时会话更新（#31714），修掉可让 llama-server 直接退出的未捕获异常；同期推进的 /v1/audio/speech 让本地服务开始兼容 OpenAI 语音合成接口。性能侧，ClassLRU 专家缓存（#31233）在 Qwen-Flash-Next 上提升解码约 6%、GLM 上约 7%，专家上传少 8%。同期的 PR #31730 则提议把 HF_TOKEN 限制在 huggingface.co 域，避免令牌被发往任意 --model-url。这类日更项目没有“大版本”，但一次崩溃修复往往比新算子更影响线上可用性。
[原文](https://github.com/ggml-org/llama.cpp/releases/tag/b12137)

**8. Cloudflare 一次清退 24 个 Workers AI 模型，截止日推到 11 月 20 日**
Cloudflare 11 月 5 日更新模型目录，宣布 11 月 20 日起停用 16 个 LLM 与 8 个其他模型，请求将返回 HTTP 410 与错误码 5028；原定 11 月 15 日，现延后五天。名单包括 DeepSeek-V4 Pro/Flash 旧版、Kimi K2.6/K2.7-Code、GLM-5.2/5.3-Flash、Qwen3-30B-A3B 等，官方给出四个替代：DeepSeek V4.1 Flash、GLM-5.3、Gemma 4 26B-A4B 与 Qwen3.8-27B；图像与语音也有对应替换。替换名单里既有开源权重模型，也有托管版本，迁移时还要重测上下文长度与单价。对把模型名写死在代码里的应用，这是一次必须在本周完成的迁移。
[原文](https://developers.cloudflare.com/changelog/post/2026-11-05-planned-model-deprecations)

*本期由站点自动生成 · 信息以原文为准。*
