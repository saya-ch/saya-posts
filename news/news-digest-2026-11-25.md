---
title: Inkling 2 Mini 三圈循环换 79.2 分，GLM-5.4 把 28 层跑两遍
date: 2026-11-25T07:00:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期的主线是用计算深度换参数规模。Thinking Machines 的 Inkling 2 Mini 只保留 264 亿参数，让中间 12 层循环三遍，在 SWE-bench Verified 上拿到 79.2%，约 20GB 的权重可以在笔记本上跑。智谱 GLM-5.4 则把 28 层跑两遍，以 750B 规模对标 2T 到 3T 模型，并把 100 万 token 上下文写进文档。开放权重侧，腾讯 Hy4 flash 用 220B 总参数、12B 激活和 1M 上下文，把单任务成本压到上一代预览版的十三分之一，工程侧，NanoClaw 把密钥改成按代理授权，Claude-BugHunter 把 681 条披露报告整理成 84 个技能。循环深度能不能替代参数堆叠，是这一周最值得盯的变量。想核对每个数字的边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Inkling 2 Mini：26.4B 参数只激活 3.4B，中间 12 层跑三遍**
11 月 20 日，Thinking Machines 以 Apache 2.0 放出 Inkling 2 Mini：24 层解码器里第 6 到 17 层共享权重连跑三次，一次前向完成 48 次层计算；26.4B 总参数只激活 3.4B，按三遍折算约 5.4B 等效激活，上下文 1M，NVFP4 与 MXFP8 混合的 checkpoint 约 20GB，可在 32GB 以上的 Apple 芯片或单台 DGX Spark 上推理。模型卡给出的 SWE-bench Verified 为 79.2%、Terminal-Bench 2.1 为 71.6%、BrowseComp 为 79.0%，AA Index 20.1 与 Qwen3.8-27B 持平。官方也承认循环让思维链更难监控，建议下游给代理加沙箱，并对删除、覆盖等不可逆操作要求确认。它把“参数量”与“每次前向的计算量”拆成两个可调旋钮，也把开源小模型的部署门槛压到消费级硬件。
[原文](https://thinkingmachines.ai/model-card/inkling-2-mini/)

**2. GLM-5.4 把 28 层跑两遍：750B 规模对标 2T 到 3T**
11 月 23 日，智谱发布首款 Loop Transformer 旗舰 GLM-5.4：取基座中间 28 层共享权重循环两次，靠逐 token 门控、共享 KV 加局部滑窗、复用稀疏注意力索引，把 1M 上下文的额外缓存压到一个滑窗。官方称 750B 规模可对标 2T 到 3T 参数模型；30B 同参对照在 GPQA-Diamond 从 47.58 升到 56.26，平均输出长度反降 11.7%，总训练算力只多约 10%。目前 GLM Coding Plan 全量开放，API 与权重待安全评估后放出，官方给出的吞吐是闲时 200 token/s、高峰 70 到 80 token/s。代价是旗舰会出现很长的“马拉松思考”，官方也提醒单看流式速度会低估一次任务的等待时间。
[原文](https://z.ai/blog/glm-5.4)

**3. 腾讯 Hy4 flash 开源：220B 总参数、12B 激活、1M 上下文**
11 月 18 日，腾讯混元开源 Hy4 flash 系列，包含标准版、FP8 版与预训练 Base：220B 总参数含 64B Engram，每 token 激活 12B，其中文本 11.3B、视觉 0.43B；42 层里 1 层稠密、41 层 MoE，512 个路由专家取 16 个，另带 0.85B 的 DFly2 草稿模型做长上下文投机解码。官方称混合注意力与 LatentMoE 把 KV 缓存压到 Hy3 的 1/24、Hy4 的 1/6，WorkIndex 42.1 略高于 Hy4 preview 的 39.7，单任务成本仅为后者的 1/13，解码超过 150 token/s。权重按腾讯 Hy License 发布，并非 Apache 协议，商用前要单独读条款。对需要在私有数据上做长文档与视频理解的团队，这类中等体量开放权重比闭源旗舰更容易算清总成本。
[原文](https://github.com/Tencent-Hunyuan/Hy4-flash)

**4. 用统计物理给代理状态建模：把后训练看成势函数浮现**
11 月 20 日，Zhuo-Yang Song 等人在 arXiv 提交《Potential emergence in LLM agents: a statistical physics perspective》，把代理的中间状态当作可观测量，用配分函数描述 KL 正则下的理想转移分布，并跟踪势结构、平均奖励与状态熵。作者在 Qwen3.5-9B 上做了安全测试筛选与物理推理审计两项任务，观察到后训练过程中势结构按框架预测增强，奖励与状态熵的联合变化提供一致性证据。论文 21 页、5 图、5 表，属于解释性工作，不提供新模型或榜单成绩，适合拿来判断“后训练到底改了什么”。它的价值不在分数，而在于给“后训练改变了代理的哪一部分”提供一个可测量的中间层。
[原文](https://arxiv.org/abs/2611.27618)

## 开源与工程

**5. NanoClaw 2026.11.0：密钥改成按代理授权，数据库补上独立密码**
11 月 21 日，NanoClaw 发布稳定版 2026.11.0，此前在 beta 通道跑了 9 个预发布：OneCLI 升到 1.45 后，新建代理组只拿到自己的模型密钥，其他工具密钥要显式授予；OneCLI 数据库从固定默认密码改为随机密码，堵住任意代理容器可登录并接管网关的漏洞。该版本还把 Cursor 接为代理提供方，内置 Claude Code 2.1.321 与 Haiku 5.5，并把 webhook 默认改成只监听本机回环地址。升级清单里 8 项需要手动处理，属于典型的安全修复伴随破坏性变更。对自托管代理平台来说，密钥边界比模型能力更先决定能不能放进生产。
[原文](https://github.com/nanocoai/nanoclaw/releases/tag/v2026.11.0)

**6. Claude-BugHunter 扩到 84 个技能：681 条披露报告变成可执行套路**
11 月 24 日，elementalsouls 更新 Claude-BugHunter：84 个技能、15 个斜杠命令，把 681 条 HackerOne 披露报告整理进 24 类漏洞，其中 433 条逐条引用、可回溯审计，覆盖 XSS、SQLi、SSRF、OAuth、M365 与 Entra、Okta、vCenter 以及 SSL VPN 等外网攻击面。项目同时支持 Claude Code、OpenCode、Codex CLI、Hermes 与 AntiGravity，代码用 MIT、内容用 CC BY 4.0。它把红队经验从一段提示词变成按主题自动加载的技能包，也把授权范围与 7 问门禁写进流程。不过技能包再全，也不能替代授权与范围确认，项目把这一点写在了显眼位置。
[原文](https://github.com/elementalsouls/Claude-BugHunter)

**7. Transformers v5.24.0：一次加入 Inkling 2 Mini、Dory 与 Apertus 1.5**
11 月 21 日，Hugging Face 发布 Transformers v5.24.0，新增 Inkling 2 Mini、NVIDIA 的循环模型 Dory、瑞士 Apertus 1.5、腾讯 HY4-VL，以及覆盖 1,600 多种语言的 Omnilingual ASR，后者提供 300M 到 7B 四档。该版本把 past_key_values 与 cache_params 统一改名为 cache，旧名保留到 v5.40.0，generate 接口保留到 v6.0.0，并把 peft 最低版本提到 0.22.0。对推理栈来说，这意味着循环模型、混合注意力与多模态输出类开始进入同一个加载接口，升级前要先查自定义模型的 cache 签名。Dory 还允许在同一个 checkpoint 上调整循环次数，用推理时计算换质量。
[原文](https://github.com/huggingface/transformers/releases/tag/v5.24.0)

**8. Ollama 0.40.18 给 Inkling 2 提速：Metal 上融合残差混合**
11 月 24 日，Ollama 发布 0.40.18：在 Metal 后端融合 Inkling 的残差混合，让 MLX runner 跨架构共享 tensor helper 与模型初始化，并把 llama.cpp 升到 b12651、移除 Jetpack 5 支持。官方称 Inkling2 在 Metal 上的 decode、prefill 与 draft 轮次都更快。这类小版本不改变模型能力，却决定 20GB 级循环模型能否在笔记本上把首 token 与长上下文吞吐跑顺，升级前要先确认旧显卡的驱动与 Jetpack 版本。本地运行器的竞争，最后往往落在驱动兼容与一次升级要改多少配置上。
[原文](https://github.com/ollama/ollama/releases/tag/v0.40.18)

*本期由站点自动生成 · 信息以原文为准。*
