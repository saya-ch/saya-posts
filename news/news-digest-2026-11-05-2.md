---
title: 2.8T Kimi 与万亿 Mistral 一周内放开权重，编码 Agent 长出技能层
date: 2026-11-05T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 过去一周开放权重接连迎来重量级交付：Kimi K3.1 Pro 把 2.8T 总参数、104B 激活与 1M 上下文摆上台面，Terminal-Bench 4.0 拿到 67%；Mistral 则放出 1T 总参数、52B 激活的 Large 4 权重，并把价格压到每百万 token 0.7/2.1 美元。腾讯 Hy4 用 770B 参数和 Unreal、Unity、Godot 引擎适配切入游戏开发，FrontierCode 2.0 得 31 分。工程侧，llama.cpp 0.9.0 一次加入四种新架构与 FP8 E4M3，GitHub 周榜则被编码 Agent 的“技能层”占据：T3 Code、Superpowers、ThinkRail 与 Impeccable 分别从控制面、方法论、IDE 和设计审查切入。两条线共同说明，权重不再是唯一卖点，能否被工具链接住、被 Agent 复用，才是这一周真正的分水岭。想看清这轮竞争的真实数字与边界，往下翻正文。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Kimi K3.1 Pro 放开 2.8T 权重：1M 上下文，Terminal-Bench 4.0 拿 67%**
Moonshot 11 月 4 日把 K3.1 Pro 的权重上传到 Hugging Face，模型总参数 2.8T、激活 104B，支持 1M token 上下文与图文输入，评测重点落在长程编码与 Agent 工具调用。Artificial Analysis 智能指数给到 55 分、排第 4，Terminal-Bench 4.0 从 K3 的 13% 拉到 67%，GDPval-AA 1,846 Elo 只落后 Claude Opus 5.5 与 Muse Wonder Full；定价为每百万 token 输入 2 美元、输出 10 美元、缓存命中 0.20 美元，比 K3 低 33%，单任务成本约 1.98 美元，单任务输出 55K token、比 K3 多 14%，权重开放后开放与闭源榜首的差距从 12 分缩到 3 分。
[原文](https://huggingface.co/moonshotai/Kimi-K3.1-Pro) · [榜单](https://artificialanalysis.ai/articles/kimi-k3-1-pro-launch-benchmarks)

**2. 腾讯 Hy4 发布：770B 参数，FrontierCode 2.0 得 31 分**
腾讯 11 月 4 日上线 Hy4，770B 参数，主打游戏开发与长程任务，官方称其为当前最强开放权重模型之一，并针对 Unreal、Unity、Godot 做了引擎适配，腾讯互动娱乐事业群参与了训练数据与游戏评测的构建；API 报价为每百万 token 输入 0.83 美元、输出 2.50 美元、缓存命中 0.042 美元，输入价约为 DeepSeek V4.1 Flash 高峰价的 2.8 倍。FrontierCode 2.0 得分 31，社区对比认为与 GPT-6.1 Sol、Opus 5.5 medium 接近，Plotly 的数据分析基准则把它放在 GPT-6.1 Sol 与 Terra 之间；腾讯同时开源 HyCell 与 HyPDE 两个科学基础模型，分别用于单细胞表征和物理场建模。
[原文](https://hy.tencent.ai/research/hy4)

**3. Mistral 放开 Large 4 权重：1T 总参数、52B 激活，许可限定“可自托管、不可转售推理”**
Mistral 10 月 29 日发布 Mistral-Large-4-1T-A52B 权重，距 10 月 6 日的预览版仅 23 天，总参数 1T、激活 52B，采用修改版 MIT 许可：自托管、研究和内部商用免费，但对外提供模型即服务需单独授权，限制主要针对转售推理的托管商，因此不算 OSI 意义的开源。同步降价后，输入、输出、缓存命中分别为每百万 token 0.7、2.1、0.07 美元，约为预览版的一半；官方称其对恶意网络安全请求的拒绝率 96.6%，在开放模型中最高，但 4bit 量化仍需约 610GB 显存，消费级硬件基本无缘。
[原文](https://mistral.ai/news/mistral-large-4-oss/)

**4. LoopVQ 把循环 Transformer 的 KV 缓存压到 0.5 bit**
11 月 3 日上传 arXiv 的 LoopVQ 针对循环 Transformer 的 KV 缓存冗余：这类模型反复复用同一组层，每个 loop step 仍各存一份 KV，缓存随步数线性增长，团队因此把多个 step 的通道联合量化，在 0.5 bit/元素下优于现有 KV 缓存向量量化方法，相当于把 BF16 的每元素 16 bit 压到 1/32，与 TaSQ 组合后 0.25 bit/元素仍能保留大部分 BF16 精度。作者在 SGLang 中实现后，32K 上下文解码延迟接近减半，Ouro 1.4B 与 2.6B 的峰值吞吐分别提升至 4.3 倍和 6.1 倍，这套方法也可与现有量化方案正交叠加。
[原文](https://arxiv.org/abs/2611.05304)

## 开源与工程

**5. llama.cpp 0.9.0：一次加入四种架构，FP8 E4M3 进主干**
llama.cpp 11 月 4 日发布 0.9.0，新增 OCP FP8 E4M3 量化、flash_attn_ext_rows 多序列统一 KV 缓存优化，以及通用批量张量拷贝 API；一次支持 Clover、Limite、Metis-1.6、Mistral Large 4 四种新架构，其中 Clover 是 35B-A3B 扩散模型，Metis-1.6 用上 Mamba-2/GQA 混合与 LatentMoE。ggml 升到 0.29.0，带来 NCCL 多机张量并行和 CPU、CUDA、SYCL、Metal、Vulkan 等后端的性能改动，Gemma 4 E2B/E4B 的预填充也做了大幅优化，音频侧补上 Chatterbox TTS、Nemotron 3.5 Transcribe 与 Pocket TTS 预置音色，对多机推理和端侧部署都是直接收益。
[原文](https://github.com/ggerganov/llama.cpp/releases/tag/v0.9.0)

**6. T3 Code 一周涨 1.25 万星：把编码 Agent 装进手机**
T3 Code 把自己定位为“Agent harness 控制面”，用 iOS、Android、Web 与 Electron 四个客户端接管本机的 Claude Code、Codex、Cursor、Grok Build、OpenCode、Antigravity 与 fx 七种 Agent，本周 GitHub 趋势榜新增 12,542 星。它不绑定模型订阅，只复用本机已登录的 CLI，把会话、审批和远程操作搬到移动端，在手机上就能查看进度、批准命令，README 明确不卖服务、代码可自行 fork，适合离开工位后仍需盯长任务的开发者。
[原文](https://github.com/pingdotgg/t3code)

**7. Superpowers 把“先写规格再写码”做成 16 个 Agent 的公共技能**
Superpowers 是一套面向编码 Agent 的开发方法论，用可组合 skills 强制 Agent 先追问需求、把规格拆成能读完的小块、再出实现计划；README 列出 Claude Code、Codex、Cursor、Gemini CLI、Kimi Code、Qwen Code 等 16 种客户端接入方式，本周新增 5,994 星。它把“设计先行”从人的习惯变成 Agent 的流程约束，把方法论固化成可安装的技能包，并允许团队把同一套技能包版本化复用到多个 harness，减少换工具就重学一遍的损耗。
[原文](https://github.com/obra/superpowers)

**8. JetBrains 的 ThinkRail：用 living specs 让 Agent 记住项目**
JetBrains 孵化项目 ThinkRail 本周登上 GitHub 趋势榜，新增 3,898 星；它把 pi 编码 Agent 包进一个轻量 IDE，用独立 workspace 隔离任务，并用随项目演进持续更新的 living specs 与可复用 skills 保存项目知识。官方路线图里的下一步是完整扩展系统，让 IDE 根据使用习惯自己生成工具；对多仓库、长周期的项目，这意味着项目上下文不再随一次会话结束而清零，也解释了它为何值得被放进周榜持续观察。
[原文](https://github.com/JetBrains/thinkrail)

**9. Impeccable 一周涨 1.14 万星：给编码 Agent 补一节设计课**
Impeccable 以 `npx impeccable install` 安装，给编码 Agent 增加一套设计词汇：`polish`、`bolder` 负责调风格，`critique`、`audit` 负责审查，目标是在交付前拦住“AI 味”排版；README 列出 Claude Code、Cursor、Codex CLI、Copilot、Devin 等 19 种可接入工具，本周新增 11,405 星，安装后以 `/impeccable init` 初始化。它用同一模型、同一份 brief 对比有无技能包的输出，把“好不好看”拆成可复述、可审查的检查项，而不是留给模型自由发挥，这对频繁生成落地页的团队尤其直接。
[原文](https://github.com/pbakaus/impeccable)

*本期由站点自动生成 · 信息以原文为准。*
