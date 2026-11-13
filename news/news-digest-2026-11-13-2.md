---
title: llama.cpp 发布 1.0，llm-d 把分布式推理做成标准件
date: 2026-11-13T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头放在“能不能稳定跑起来”，而不是又多了几个榜单分数。llama.cpp 用 v1.0.0 结束 0.x 时代，FP8 KV cache、MCP 工具调用取消与 OpenAI 兼容语音接口一起进主线；llm-d 1.0 在 CNCF 治理下把多租户 QoS、零停机升级和 agent 感知路由做成标准件，IBM 实测推理成本降到专有方案的 1/11.5。模型侧，Edge0-145B 以 10.12B 激活参数在单张 24GB 显卡上跑出 29GB 峰值内存，Duo 则用 708 毫秒一个块的全双工节奏把“什么时候开口”单独建模。三条线都指向同一件事：部署与运维的细节，正在取代单纯的参数竞赛。想看每条的数字与边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Edge0-145B 开源：145B 总参数、10.12B 激活，单张 24GB 显卡可跑**
11 月 12 日，Edge0 同时放出 145B 与 120B 两个开放权重模型，前者从 Qwen3.8-Flash-Next 派生，48 层、每层 512 个专家、每 token 激活 10 个路由专家加 1 个共享专家，原生按 4 位训练，上下文 256K，官方实测峰值内存 29GB。权重提供 Q3_K 与 NVFP4 两种构建，在 RTX 5090 D 24GB 上为 23.25 GiB 显存加 38.27 GiB 内存、agent 解码 121.2 tok/s；Terminal-Bench 2.1 得 86.52、DeepSWE 1.1 得 64.6，均高于同源 Qwen 的 83.3 与 58.7，但 HLE 35.4 略低于对方的 35.9。它把“大模型留在本机”从演示推进到了可日常使用的工程件，代价是仍需 NVIDIA GPU 与 CUDA 13，且许可证叠加了 Qwen 社区条款。
[原文](https://ai-tldr.dev/models/edge0-145b/)

**2. arXiv 论文：281 个可执行训练环境，教 agent 自己做完后训练**
RSI-Dojo 想解决的是“会调 prompt 不等于会训模型”：团队先用 RSI-Craft 从顶会论文自动抽取数据集、指标与依赖，过滤掉数据过小或执行不稳定的任务，得到 281 个可运行的模型训练环境，再用改进后的 harness 从前沿模型采集轨迹并按评分筛选。用这些轨迹微调 Qwen3.5-9B 与 Qwen3.6-35B-A3B，分别比基座高 8.74 与 16.87 个 PostTrainBench 点；35B-A3B 版本拿到 29.95 分、轻量版 53.29 分，已与 GLM-5.2、GPT-5.5 同级。论文 10 月 29 日首发、11 月 9 日更新到 v2，值得注意的不是分数，而是它把“选数据、训模型、诊断失败”拆成了可复现的执行环境。
[原文](https://arxiv.org/abs/2611.03698v2)

**3. Duo-1.0 静默开源：153.46 GB 权重，708 毫秒一个交互块**
11 月 12 日，蚂蚁 inclusionAI 上传 Duo-1.0-causal-base，80 个文件共 153.46 GB，配套论文 11 月 11 日挂上 arXiv。它把交互拆成 Intent 与 Realization 两个模块：前者每个块只决定“说”还是“等”，后者用扩散 Transformer 生成音视频，块长固定为 17 帧、24fps，也就是 708 毫秒，官方给出的端到端延迟是 8 卡下 330 至 650 毫秒。评测由作者按 6 项标准自评，180 项检查过 168 项、93.3%，其中内容响应性只有 73.3%；权重沿用 MiniMax H3 社区许可，明确排除欧盟、英国、韩国与美国，落地前必须先看许可证而不是显卡预算。
[原文](https://www.orcarouter.ai/blog/duo-1-0-causal-base-quiet-weights-release)

## 开源与工程

**4. llama.cpp 发布 v1.0.0：结束 0.x，FP8 KV cache 与 MCP 超时取消进主线**
11 月 12 日，这个从 2023 年跑到现在的小型推理项目把版本号推到 1.0.0， nightly 构建为 b12359，变更集从 v0.9.0 累计而来。最值得关注的是服务端能力补齐：新增 FP8 KV-cache（#28392）与 CUDA 的 OCP FP8 支持（#28413），加入 OpenAI 兼容的 /v1/audio/speech 接口（#31763），客户端断开或超时会取消正在跑的 MCP 工具调用（#32025），并集中修掉 CORS 代理与媒体 URL 的 SSRF 问题（#32115、#32098）。同期还合入 Kimi-K3.1-Pro 文本模型与 Qwen3.5 权重转换合并。对一个被当作“本地跑模型工具”的项目来说，1.0 的含义是它开始按服务器而不是玩具来定义自己的边界。
[原文](https://github.com/ggml-org/llama.cpp/releases/tag/v1.0.0)

**5. llm-d 1.0 发布：500 多名贡献者，把分布式推理交给 CNCF**
11 月 13 日，由 Red Hat、IBM、Google Cloud、CoreWeave 与 NVIDIA 发起、现由 CNCF 治理的 llm-d 发布 1.0，新增多租户 QoS、零停机运维、稳定配置 API 与 agent 感知服务，底层同时支持 vLLM 和 SGLang。项目现有 25 个以上组织的 500 多名贡献者；IBM 研究给出的案例是量子纠错硬件探索的推理成本比全专有方案低 11.5 倍，前缀缓存命中率约 98%，混合服务成本约每百万 token 0.11 美元。另一组演示用 NVIDIA GPU 部署约 753B 的 GLM-5.2，在数百会话并发下峰值每分钟输出 660 万 token、支撑 3000 个并发编码 agent 且零抢占。它把“推理栈”从某个引擎的附属功能，变成了 Kubernetes 上可独立演进的一层。
[原文](https://itbrief.co.nz/story/llm-d-1-0-aims-to-ease-production-ai-inference-at-scale)

**6. SGLang-Neuron v0.1.0 开源：Kimi-K3 跑在三台 Trainium3 上**
11 月 12 日，RadixArk 的 SGLang 团队与 Amazon Annapurna Labs 放出 SGLang-Neuron v0.1.0，以 PyTorch 原生插件形式把 SGLang 的服务栈搬到 AWS Trainium，公开预览期通过 Deep Learning Container 分发。它把每个解码步的整层计算压进一个 NKI megakernel，并用全异步运行时让设备在两次 launch 之间不空转；Kimi-K3 以 1 台 prefill 加 2 台 decode 的布局跨 3 个 Trainium3 节点运行，4K 至 1M 混合提示下满槽位可达每用户 67 tok/s，GLM-5.2（MXFP4）则支持 1M 上下文。HiCache 把被驱逐的前缀留在主机内存，GPT-OSS-120B 在 TP8、32K 上下文下输出吞吐提升 72% 至 75%。这是非 NVIDIA 路线第一次把“能跑”推进到“能按 agent 负载调参”。
[原文](https://www.lmsys.org/blog/2026-11-12-sglang-neuron-v0.1.0)

**7. Open WebUI v0.13.3：把编码 agent 挂进定时自动化**
11 月 12 日的 v0.13.3 做了 3 项新增、13 项修复，主线是让 agent 从聊天窗口走进自动化：自动化编辑器现在可以选择终端与工作目录，再挑一个该终端里可用的编码 agent，定时任务启动前会复查终端与 agent 是否仍在，结果回送到指定会话而不是频道。聊天中也能随时切换终端或目录，临时会话同样生效；Files 标签会跟着终端走，不再显示上一个终端的目录。修复项里比较关键的是 DeepSeek 思考模式在 llama.cpp Provider 下不再报缺少 reasoning 的错，以及知识库检索能正确处理“Art. 40”这类带标点的查询。
[原文](https://newreleases.io/project/github/open-webui/open-webui/release/v0.13.3)

**8. Ollama v0.40.16：Qwen3.8 Flash Next 在苹果芯片省下约 27 GiB 内存**
11 月 10 日的 v0.40.16 是一次偏“省资源”的维护版：官方称 Qwen3.8 Flash Next 在 Apple silicon 上少占用约 27 GiB 内存，同时把 Ripple Preview 带进模型库——这是 Ricursive Intelligence 的 65B 稠密模型，定位 agentic coding，可直接 ollama run。接口侧修了两个容易埋坑的行为：/v1/responses 开始尊重 truncation 参数、不再默认静默截断，/v1/models 在没装模型时返回空数组而不是 null；此外补上 Windows on ARM 的 CUDA 13.5 支持，并同步更新了内置 llama.cpp。对把 Ollama 当本地网关的团队，截断语义的变化值得先跑一遍回归。
[原文](https://github.com/ollama/ollama/releases/tag/v0.40.16)

*本期由站点自动生成 · 信息以原文为准。*
