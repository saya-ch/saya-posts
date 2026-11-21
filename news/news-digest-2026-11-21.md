---
title: Genie 4 开放 API 按秒计费，20GB 的 Inkling 2 Mini 跑出百万上下文
date: 2026-11-21T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 本期最值得看的是两条把大模型拉回可交付形态的消息：谷歌把 Genie 4 世界模型接进 Gemini API，720p/24fps 的交互世界按每秒约 0.1 美元计费，单次会话封顶 120 秒；Thinking Machines Lab 的 Inkling 2 Mini 则用 26.4B 总参数、3.4B 激活和 20GB 权重，在 32GB Mac 上跑出 1M 上下文与 79.2% 的 SWE-Bench Verified。开源侧，0.5B 的 Rumble-2-Flash 把七语平均词错率压到 7.09%，NanoClaw 把每个 Agent 的密钥收窄，vLLM 26.11 给 Rubin 开了 GA，111GB 的 180B MoE 也学会按需从 SSD 取专家。三条线指向同一个变化：竞争焦点正从参数规模转向「能不能在真实硬件和真实权限下交付」。想看清每个数字背后的代价，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Genie 4 开放 API：720p 世界按秒卖，单次会话封顶 120 秒**
谷歌本周把 Genie 4 世界模型接进 Gemini API 的 Interactions 接口与 AI Studio，开发者可用文字、参考图或两者组合生成 720p/24fps 的可交互场景，并同时得到原生立体声音频；低延迟网络下键盘操控响应 0.2–0.5 秒，用文字改天气、光照或物体则要 1–2 秒。计费按 token 走，含音频的视频输出 17.50 美元/百万 token，折合约 0.10 美元/秒、6.05 美元/分钟，不设免费层，单次会话上限 120 秒且必须显式调用终止接口，否则关掉网页也会继续扣费；目前仅美国数据中心提供服务，付费档按 2/5/10 个并发会话分三级。
[原文](https://www.itsdw.cn/news/50639.html)

**2. Inkling 2 Mini 放开 20GB 权重：26.4B 总参数，1M 上下文**
Thinking Machines Lab 11 月 20 日以 Apache 2.0 放出 Inkling 2 Mini，24 层 Transformer 的中间 12 层循环三次，每层 128 个专家里选 8 个，26.4B 总参数只激活 3.4B（三次循环等效 5.4B），20GB 的 NVFP4/MXFP8 权重可装进 32GB Mac 或单台 DGX Spark。官方模型卡给出 SWE-Bench Verified 79.2%、Terminal Bench 2.1 71.6%、BrowseComp 79.0%，上下文上限 1M token；代价是它明确弱于更大的 Inkling 2 Small，事实召回和长对话稳定性都留了警告；权重可从 Hugging Face 下载，也能在 Tinker 上做微调。
[原文](https://thinkingmachines.ai/model-card/inkling-2-mini/)

**3. Rumble-2-Flash 用 0.5B 做 40 多语种转写，七语平均 WER 7.09%**
Reson8 11 月 18 日以 Apache 2.0 开放 Rumble-2-Flash，5 亿参数的语音识别模型覆盖 40 多种语言，官方在 Open ASR 多语测试的七语平均词错率 7.09%、英语约 5%，10 种低资源语言平均 17.6%，而 Whisper Large V3 在同一测试为 39.82%。它把「小模型只配跑英语」的默认假设撕开一个口子：模型强调电话线、噪声环境下的鲁棒性，API 起价 0.01 欧元/小时音频，权重可直接下载本地部署；自定义词表仍只留给旗舰 Rumble-2，Rumble-2-Flash 早在 11 月 3 日先上了 API。
[原文](https://siliconangle.com/2026/11/18/reson8-releases-rumble-2-flash-speech-model-as-a-free-open-weights-download/)

**4. 小模型“堆料”实验：五个改动同台，复跑两个种子才看得出增益**
arXiv 11 月 18 日的一篇论文把 MoE、Muon 优化器、Gated DeltaNet 混合、额外归一化和注意力门控五个改动放进同一模型、同一数据和同一训练预算，做带重复的因子实验，每个组合跑两个随机种子。结果显示前三者增益最大、额外归一化也稳定有效，参数预算匹配的注意力门控平均贡献很小，部分交互效应第一次被统计判成噪声、复跑后才显出来，作者因此建议先把候选组件一起测再复跑，别把单项消融当结论；这套五个组件、两个种子的做法，也解释了为什么小模型配方很难靠单篇论文复现。
[原文](https://arxiv.org/abs/2611.24540)

## 开源与工程

**5. 180B MoE 压成 111GB GGUF，32GB 笔记本靠 SSD 流式跑 4.17 token/s**
FINAL-Bench 把 180B 的 Darwin-180B 压成 111GB 的 4-bit GGUF（BF16 为 360GB），模型有 512 个路由专家、每 token 只选 10 个、约 3B 参数参与计算；它在一台 32GB 内存、8GB 显存加 NVMe 的 RTX 5060 笔记本上用 llama.cpp 内存映射、按需从 SSD 取专家，跑到 4.17 token/s。换成 16 线程 EPYC、权重常驻内存则有 18.4–21.0 token/s、峰值 78.8GB，4-bit 与 BF16 在 2000 题 MMLU-Pro 上同为 87.65%，说明这套「存储当显存」的做法至少在这项评测上没掉点；代价是 111GB 下载之外还要至少 120GB 空闲 NVMe，量化时也只重写了自改进训练动过的 300 个张量。
[原文](https://huggingface.blog/blog/final-bench-data-center-ai-now-on-a-laptop-pocket-darwin-180b/)

**6. NanoClaw 2026.11.0：每个 Agent 只拿自己的密钥**
开源 Agent 运行时 NanoClaw 11 月 19 日发布 2026.11.0，安全模型改成「每个 Agent 只拿你给的密钥」：OneCLI 升到 1.45 后新组默认只有模型凭证，OneCLI 数据库改用独立密码，Webhook 默认只监听本机回环地址。功能侧加入 Cursor provider、用模板创建子 Agent、Slack 原生 KPI 卡片，本地镜像推进到 Claude Code 2.1.321 与 Node 22.23.5；release notes 列了 8 项升级前必须手动处理的破坏性变更；1.45 也是最后一个能把旧安装转换过来的 OneCLI 版本，跳过它就得重配密钥。
[原文](https://github.com/nanocoai/nanoclaw/releases/tag/v2026.11.0)

**7. GitHub 周榜：13 个项目加 6.1 万星，Agent 工具链占了一半**
11 月 13 日至 20 日的 GitHub Trending 周榜里，13 个项目合计新增约 61,364 星，dream-num/univer 以 +7,248 居首（总 30,438），Meta 的 astryx +4,977、阿里 open-code-review +4,492、ag-ui +3,455 紧随其后。榜单超过一半是代码审查、Agent-前端协议、设计系统和技能库，而不是又一个模型包装器；plannotator 与 chrome-devtools-mcp 分别 +3,468、+3,229，四个技能包项目合计 +11,560，社区这周押注的是「Agent 写完的东西怎么审、怎么验、怎么交付」，这也是 univer 这种 2022 年老项目加上 AI 操作层后突然冲顶的原因。
[原文](https://news.qiniu.com/articles/github-weekly-univer-agent-toolchain-boom)

**8. NVIDIA vLLM 26.11：Rubin 进 GA，容器带 vLLM 0.33**
NVIDIA 的 vLLM 26.11 容器把 vLLM 0.33.0、FlashInfer 0.7.2、Transformers 5.20.2 与 CUDA 13.5 打包，首次给出 Rubin GPU 架构的 GA 支持，并补上 Nemotron Super V3、Nemotron 3 Nano Omni 与 DeepSeek V4 的适配。工程上更关键的是把 Blackwell 上 NVFP4 MoE 的后端切到 FlashInfer CUTLASS，修掉 autotune 崩溃；上一版已带 Rust 前端与 vllm-rs，这一版补上 manylinux_2_34 wheel，已知问题则是 vLLM 默认把显存吃到接近 100%，在 DGX Spark 这类统一内存设备上要手动把 --gpu-memory-utilization 调低。
[原文](https://docs.nvidia.com/deeplearning/frameworks/vllm-release-notes/rel-26-11.html)

**9. SGLang 把 1.0 当稳定性发布：静默改配置一律改成报错**
SGLang 的 1.0 追踪 issue 把这次发布定性为稳定性版本：不新增大功能，重点是把「静默改配置」改成启动或请求时直接报错并点名参数，管理路由必须带 admin API key，sgl-router 0.1.0 取代 sgl-model-gateway。升级代价写得很直白：0.5.x 的启动命令可能要改，--disable-cuda-graph 与 --enable-dp-attention 已移除，不支持的 DFLASH/DSpark 步数不再自动降级；对把 SGLang 放进生产流水线的团队，这比多一个模型更值得先读，而 1.0 也会带上 Kimi K3.1 与 sgl-router 0.1.0。
[原文](https://github.com/sgl-project/sglang/issues/50045)

*本期由站点自动生成 · 信息以原文为准。*
