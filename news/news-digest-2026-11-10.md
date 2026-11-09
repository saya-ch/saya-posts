---
title: Fable 5.2 降价两成登顶编码榜，65B 开源模型只花十分之一成本
date: 2026-11-10T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期的主线是单位智能的价格。Anthropic 把 Fable 5.2 与 Mythos 5.2 的 API 降到每百万 token 8/40 美元，FrontierCode 2.0 以 49.1% 登顶，Mythos 的 CVE 复现率则到 89.6%；同周 Ricursive 用 64.7B 的 Ripple-65B 拿到 Terminal-Bench 2.1 的 80.52，成本只有上一个开放权重纪录的十分之一，Cognition 的 SWE-2 Flash 也把价格压到 SWE-2 的五分之一。另一端，GPT-6 Cyber 以十倍于 Sol 的价格只卖给通过审批的机构，说明最强能力仍在收紧准入。工程侧，Codex CLI 0.167.0、Claude Code 2.1.319 与 Ollama 0.40.16-rc0 都在补后台任务、后端切换与本地多模型这些“能不能稳定跑”的细节，Hugging Face 则把 80 万 GPU 小时的 MoE 训练经验写成 7 万字手册。价格与能力同时变化时，真正值得读的是每条路线各自的边界，逐条数字见正文。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Claude Fable 5.2 与 Mythos 5.2 同发：编码榜 49.1%，API 降价 20%**
Anthropic 在 11 月 9 日把两款模型一起推上台：同一个底座，Fable 面向所有付费用户，Mythos 只对通过网络安全与生命科学验证的机构开放。FrontierCode 2.0 在 Xhigh 档拿到 49.1%，高于 Opus 5.5 的 45.5% 与 GPT-6 Astra 的 42.8%，FrontierSWE v2 为 69.9%，RoboDojo 为 64.9%。标准价从每百万 token 10/50 美元降到 8/40 美元，降幅 20%，Fast 模式 16/80 美元、速度最高 2.5 倍，官方称输出比 5.1 快 30%；Mythos 的 CVE 复现率到 89.6%，沙箱外凭据使用率从 5.1 的 21% 降到 3.1%。两款模型共用同一底座，Fable 保留网络安全与生物安全分类器，Mythos 走验证计划；官方称 1,000 个 Agent 用早期快照在 30 小时内形式化了费马大定理的 Lean 证明，Opus 团队同期只完成约 27%。
[原文](https://www.anthropic.com/claude-fable-and-mythos-5-2)

**2. Ripple-65B Preview：64.7B 稠密模型，Apache 2.0 开放 1M 上下文**
Ricursive Intelligence 11 月 4 日放出 Ripple-65B Preview：64.7B 稠密参数、12 万亿 token 预训练，上下文 104 万 token，Apache 2.0，权重已在 Hugging Face 与 Fireworks 上线。官方称 Terminal-Bench 2.1 得 80.52，超过 501B 的 Beam（80.1）、816B 的 Muse Spark Open（76.4）与 975B 的 Inkling（63.8），参数量只有它们的约八分之一到十五分之一；开发成本约为上一个开放权重纪录模型的十分之一，把 Terminal-Bench 2.1 推到 80.5 只用了 140 万次试验，约为 Beam 的七十分之一。权重可直接自托管，适合代码生成、Agent 这类需要私有数据的场景；但 64.7B 稠密模型全精度仍要多卡，Fireworks 只提供按需部署。
[原文](https://huggingface.co/ricursiveAI/Ripple-65B-Preview)

**3. SWE-2 Flash：276B-A12B 只激活 12B，成本压到五分之一**
Cognition 11 月 4 日发布 SWE-2 Flash：以 Thinking Machines 的 Inkling-Small（276B 总参、12B 激活）为基座继续中期训练与强化学习，在 Devin Desktop 与 CLI 里按 SWE-2 五分之一的价格、约三倍的速度提供服务，生成速度 200 token/s 对 60 token/s。Terminal-Bench 4.0 从基座的 0.9% 升到 20.0%，FrontierCode 2.0 从 3.2% 升到 24.0%，ExploitBench 从 11.3% 升到 34.8%；官方称它超过所有美国开放权重模型，但权重不公开发布，只通过审核计划向行业伙伴与研究者共享，11 月 30 日前在 Devin 内免费。基座只有 12B 激活，说明小激活模型在编码 Agent 上仍能靠中期训练与强化学习追上大模型；代价是权重不开放，团队只能在 Devin 里按用量付费。
[原文](https://cognition.com/blog/swe-2-flash)

**4. GPT-6 Cyber：95% 远程漏洞利用成功率，价格是 Astra 两倍**
OpenAI 11 月 5 日上线 GPT-6 Cyber，专攻漏洞发现与利用开发，上下文 105 万 token，定价每百万 token 输入 20 美元、输出 100 美元，是 GPT-6 Astra 的两倍、GPT-6.1 Sol 的十倍，且必须通过 Daybreak Red 与单独模型审批。官方给出 95% 的远程漏洞利用任务完成率、99.93% 的授权范围内比例，Preparedness 框架把它的网络安全风险定为 Critical；它对普通开发者不可见，只在 API 与 Codex 中向首批客户开放。这也意味着前沿能力的分发开始按客户资质分层：同样是 GPT-6，Astra 公开可买，Cyber 只对经过身份核验的安全团队开放。
[原文](https://llm-stats.com/blog/research/gpt-6-cyber-launch)

## 开源与工程

**5. Codex CLI 0.167.0：终端里切后端、预览图片，补上 GCP 向导**
OpenAI 11 月 6 日发布 Codex CLI 0.167.0：新增 codex agents --no-daemon 的命令中心，会话选择器可在已连接后端之间切换并保留草稿，终端支持本地图片附件预览，/experimental 里能开启 macOS 与 Windows 系统代理，gcp_setup_wizard 则用应用默认凭据或凭据文件接 Google Cloud Vertex AI。修复项包括切换模型提供方后已保存会话仍可见、Ctrl+L 清屏但 Ctrl+T 保留历史、失败响应的上下文压缩计入保留输出；这一版全是 TUI 与运行时改动，没有配置 schema 或 hooks 变更，升级风险较低。命令中心的意义在于把多个后台 Agent 会话收进一个面板，不必为每个任务开一个终端；对同时跑审查、修 CI、写文档的团队，切换成本比新增功能更实际。
[原文](https://github.com/openai/codex/releases/tag/rust-v0.167.0)

**6. Claude Code 2.1.319：一次 90 项修复，MCP 长任务转后台**
Anthropic 11 月 6 日的 2.1.319 把 MCP 任务调用移到后台：服务器把工具调用当任务跑时，Claude Code 立刻转后台，完成后再把结果告诉模型，同时给出 CLAUDE_CODE_DISABLE_MCP_TASK_BACKGROUND，让需要同步等待的人可以关掉。新增 permissions.denyInsteadOfPrompting，把原本会卡在权限弹窗的调用直接拒绝并继续回合；/claude-api latency-optimize 用来定位 Claude API 代码里的等待点。整版含 16 项改进与 90 项修复，其中一项是 claude -p 在动作被拒后仍错误返回 0，对无人值守代理影响直接。把长任务移出前台，解决的是 MCP 工具跑几分钟时整个会话被占住的问题；配合拒绝而非询问，无人值守批处理不会因为一个弹窗停到超时。
[原文](https://github.com/anthropics/claude-code/releases/tag/v2.1.319)

**7. Ollama 0.40.16-rc0：MLX runner 接住 Ripple，补 CUDA 13.5 ARM64**
Ollama 在 11 月 10 日凌晨放出 0.40.16-rc0：给 MLX runner 加上 Ripple 的模型、渲染器、解析器与导入注册，并支持 DFlash 草稿的逐层 NoPE，同时融合 MLP 激活、value blending 与 attention gating；同一版把 Qwen3.8 Flash Next 在 MLX 上的显存占用压下来，Windows ARM64 补上 CUDA 13.5 与 SM 103，OpenAI 兼容层会转发 responses 请求的 truncation 选项、无模型时返回空数组。它仍是预发布版，想在本机试 Ripple 的人可以用 ripple-preview:65b-mlx-bf16，但生产环境建议等正式 tag。Ripple 的 MLX 路径先落地，说明本地运行时对开放权重的跟进已从数月缩短到几天；但 rc0 的回归测试覆盖有限，显存与量化兼容仍要自己验证。
[原文](https://github.com/ollama/ollama/releases/tag/v0.40.16-rc0)

**8. Hugging Face 把 MoE 训练写成 7 万字手册：80 万 GPU 小时换来的配方**
11 月 5 日上线的《The Ultimate MoE Training Handbook》由 Nouamane Tazi、Leandro von Werra 与 Thomas Wolf 编写，约 7 万字、55 张交互图，配 Megatron-LM 可运行配方，结论来自 Hugging Face 在 Alps 集群上约 80 万 GPU 小时的实验。手册把专家放置、负载均衡、路由精度、通信与显存、量化与服务逐段拆开，例如路由层建议用 fp32、单专家上采样的 MoE 在 15 步内与稠密模型的差距只有 0.00087；对想从稠密模型切到 MoE 的团队，这是一份能直接照做的检查表。手册还给出通信与显存占用的实测曲线，解释专家并行度提高后 all-to-all 为什么会先成为瓶颈；这类一线数据比论文里的架构图更有参考价值。
[原文](https://huggingface.co/spaces/huggingface/moe-blogpost)

*本期由站点自动生成 · 信息以原文为准。*
