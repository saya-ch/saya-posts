---
title: Gemini 4 Neon 拿到 61 分，27B 的 SRE-1 把事故排查成本压到三分之一
date: 2026-11-20T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期的主线是“更小、更便宜，但更贴近具体任务”。Google 的 Gemini 4 Neon 在 AA 智能指数拿到 61 分，输出速度翻倍到 110 token/s，定价只落在中档；ResolveAI 则用 27B 的 SRE-1 证明，强化学习能把事故根因排查的单次通过率从 6.9% 拉到 38.8%，五次集成追平前沿模型还便宜 37%。安全侧，AutoPI 与 PIPETTE 把红队指标从“攻破几次”改成“覆盖多少种手法”，指出高攻击成功率不等于防御真的变强。工程侧，pydantic-ai、Unsloth、Open WebUI 与 CrewAI 都在补本地运行、依赖解耦与沙箱边界。想看清每个数字和升级代价，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Gemini 4 Neon 发布：AA 智能指数 61 分，输出速度翻倍到 110 token/s**
11 月 18 日，Google 发布 Gemini 4 Neon，定位在 Hydrogen 与 Carbon 之间，支持 100 万 token 上下文和文本、图像、视频、语音输入，同步输出上限 6.4 万 token，后台长文生成可到 100 万。Artificial Analysis 测得最高思考档 61 分，比 Carbon 高 1 分、排全榜第三；输出速度 110 token/s，是 Carbon 的两倍，但单任务耗时 21.5 分钟，中位数只有 12.4 分钟。API 定价为每百万 token 输入 3 美元、输出 15 美元，12 月 1 日 Carbon 促销结束后反而便宜 25%。Google 自己的表格里，Neon 在 Terminal-Bench 4.1 以 65.2% 略输 Claude Opus 5.5 的 65.8%，AutomationBench 则以 56.3% 领先 Opus 的 42.5%。
[原文](https://blog.google/innovation-and-ai/models-and-research/gemini-models/introducing-gemini-4-neon/)

**2. ResolveAI 用 27B 模型做事故根因：单次通过率从 6.9% 提到 38.8%**
11 月 18 日，ResolveAI Labs 发布 SRE-1：从 Qwen3.8-27B 开放权重出发，只用强化学习后训练，就得到一个专查生产事故根因的 27B 稠密模型。在 49 个来自真实生产的困难案例上，单次调查通过率从基座的 6.9% 升到 38.8%，提升超过五倍；五次调查集成达到 71.4%，追平其测试过的最强前沿模型，成本却低 37%。训练数据包含 1,503 个真实事故与 1,774 份存档调查拆出的 7,716 个问题，所有工具都被钉在告警触发那一刻，防止模型读到事后修复记录；最终训练跑在最多 512 张 GPU 上，耗时 7.4 天。权重不公开，成绩也来自自家基准。
[原文](https://resolve.ai/labs/sre-1/paper)

**3. AutoPI：不只测“能不能攻破”，还统计 6 类注入手法覆盖了多少**
11 月 18 日的 arXiv 论文提出 AutoPI，把提示注入红队从“刷高攻击成功率”改成“测覆盖度”。它用 PIAtlas 手法分类表指导攻击：攻击模型为每个子手法写一条注入，失败的再依据裁判反馈改写，全程黑盒、无需训练。在 AgentDojo 上，它对 Qwen3.6-35B 在全部 6 个类别、超过 90% 的技术上都找到成功注入；面对最稳的 Claude Haiku 5.5，攻击成功率仍比近期最强自适应红队高 20%–50%，类别级覆盖率高 25%–400%。作者同时开源代码与生成的攻击样本，便于复现按覆盖度计算的评测。
[原文](https://arxiv.org/abs/2611.24313)

**4. PIPETTE 指出 RL 红队会“模式坍塌”：攻击成功率涨了，防御却没变强**
同日的 PIPETTE 论文补上了 AutoPI 的另一半：如果红队反复发现相似攻击，多出来的成功样本并不会让对抗训练更鲁棒。作者实测 RL-Hammer 与 PISmith 两种主流强化学习红队方法都存在模式坍塌，于是改用基于字典学习的奖励鼓励策略多样性，再接一条面向间接提示注入的在策略对抗训练流水线。在 AgentDojo 与 ASB 上，训练出的防御者对静态攻击的成功率接近零，也能扛住较强的自适应攻击，同时良性任务完成率保持在未防御基线附近。两篇论文合起来说明，红队报告该报告“测了多少种”，而不只是“破了几次”。
[原文](https://arxiv.org/abs/2611.24362)

**5. 低资源翻译量化实测：INT4 省 49%–61% 显存，延迟反而涨 1.3–5.3 倍**
11 月 18 日的量化试点研究覆盖豪萨语、斯瓦希里语与约鲁巴语，比较 Qwen2.5-1.5B、Aya-23-8B 与 NLLB-200-distilled-1.3B 在 FLORES-200 上每个方向 1,012 句的表现，硬件是消费级 T4。结论有几处反直觉：专用翻译模型只有 8B 指令模型的约六分之一大，却在豪萨语和斯瓦希里语上高出 32.7–43.3 chrF，约鲁巴语也高 12.9–24.7；INT8 与 NF4 把翻译质量稳定在约 1 chrF 内，最差 2.19，INT4 省下 49%–61% 显存，但因缺少快速内核，延迟反增 1.3–5.3 倍。压缩不是低资源部署的免费桥梁，架构与硬件适配仍占主导。
[原文](https://arxiv.org/abs/2611.24043)

## 开源与工程

**6. pydantic-ai 2.71.0 发布，上一版刚修掉一个高危 ReDoS**
11 月 19 日发布的 pydantic-ai 2.71.0 把 `PeerNetwork` 改名为 `PeerConnection`，新增 Stav 欧洲推理网关与 Otari 兼容网关，并让 `ToolOutputLimits` 支持 `grep_tool_result` 与按字节读取溢出内容。更值得升级的是 2.70.0 修掉的高危公告 GHSA-m4cr-m4qc-9hh3：开启 `strip_ansi=True` 时，旧版用二次复杂度的正则清理 ANSI 转义，一段几百 KB、含大量未闭合转义序列的工具输出就能占住事件循环数十秒到数分钟。新版本还让延迟工具调用把 `validated_args` 暴露给审批人，避免“批准时看到的参数”和“真正执行的参数”不一致。
[原文](https://github.com/pydantic/pydantic-ai/releases/tag/v2.71.0)

**7. Unsloth 0.1.920 支持 AMD NPU：Q4NX 模型可直接加载，每焦耳效率高 2–4 倍**
11 月 19 日的 Unsloth 0.1.920-beta 把 AMD Ryzen AI NPU 的支持补成可用状态：在导出页转换出的 Q4NX 模型现在能直接在 NPU 上加载，每个模型会标出“可在 NPU 运行”“需先转换”“转换后也跑不了”或“不支持”，判断来自本机安装的 FastFlowLM。官方给出的对比是 NPU 每焦耳每 token 的效率比 GPU 高 2–4 倍；扩散模型 LoRA 训练每步最多快 42%，并首次支持 T4；Strix Halo 上 FLUX.3 Klein 在 Windows 每步快 28%。此外新增 LTX-2.5、MOSS-Image 的 LoRA 训练和 Cosmos3 Nano/Edge FP8 视频模型加载。
[原文](https://github.com/unslothai/unsloth/releases/tag/v0.1.920-beta)

**8. Open WebUI 0.13.4 引入 Tether：把本机模型和终端接进来，沙箱只放开一个目录**
11 月 17 日的 Open WebUI 0.13.4 用 Tether 取代 Direct Connections：本机的 Ollama、LM Studio 或任意 OpenAI 兼容服务，以及 OpenAPI、MCP 工具服务和 Open Terminal，都能只对当前用户出现在模型选择器与工具菜单里，地址、命令和密钥不离开本机。Tether 0.1.13 起还能把终端锁进沙箱，在 macOS 与部分 Windows 设备上只允许访问用户选定的一个文件夹，默认离线、不接受入站连接，也不需要 Docker、虚拟机或管理员权限；如果沙箱无法强制生效，终端会保持停止而不是悄悄裸跑。本地 MCP 服务可走 stdio，无需 mcpo 代理或额外端口。
[原文](https://github.com/open-webui/open-webui/releases/tag/v0.13.4)

**9. CrewAI 1.15.42 把 ChromaDB 变成可选依赖，知识库默认改用本地 Qdrant**
11 月 18 日的 CrewAI 1.15.42 调整了默认存储栈：ChromaDB 不再是必装依赖，Knowledge 默认落到本地 Qdrant，存储层与 ChromaDB 解耦，独立 Agent 也可以先建好、之后再挂工具。修复项里有两处值得注意：LanceDB 在超过 5 万行时的作用域计数现在能算准，`max_execution_time` 不会再空等工作线程；此外还处理了 CSV 换行、UTF-8 BOM 配置文件和流式响应中夹带用量统计的情况，并把 soupsieve 下限提到 4.0 以避开 CVE-2026-126190。对自托管团队来说，这是一次减少外部服务依赖的升级。
[原文](https://github.com/crewAIInc/crewAI/releases/tag/1.15.42)

*本期由站点自动生成 · 信息以原文为准。*
