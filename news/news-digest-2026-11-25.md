---
title: SWE-2.1 把漏洞利用拉进编码模型，124B 开源模型只激活 5.5B
date: 2026-11-25T07:00:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期的主角是两条相反的路线，一条把能力收进产品，一条把权重放给社区：Cognition 的 SWE-2.1 把漏洞利用与长程自主写进编码模型，FrontierCode 2.0 从 24.1 升到 41.3，Lightning 版靠 Cerebras 提速 10 倍；蚂蚁则把 124B 的 Ling-3.2-air 以 MIT 协议放到 Hugging Face，每次只激活 5.5B，却把自动化基准从 15.7% 拉到 61%。开放权重侧，Sarvam 用 105B、Apache 2.0 和每百万 token 0.15 美元贴住 Flash-Lite 价格带；研究侧的两篇论文，一篇拆反事实世界模型，一篇算长上下文的显存账。两条路线没有互相取代，却都在把能力换算成单位成本与可审计的交付。想核对每个数字的边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. SWE-2.1 把漏洞利用拉进编码模型，Lightning 版提速 10 倍**
11 月 24 日，Cognition 发布 SWE-2.1，仍在 Kimi K3 上做纯强化学习，把训练任务从几分钟拉长到数小时：FrontierCode 2.0 从 SWE-2 的 24.1 升到 41.3，ExploitBench 达 80%，ExploitGym 在 869 个任务、6 小时时限下拿到 27.3% 的 flag。与 Cerebras 合作的 Lightning 版把同一模型提速 10 倍，12 月 8 日前在 Devin 桌面、云与 CLI 免费；官方同时向防御方开放限制更少的 SWE-2.1 Cyber，并披露沙箱逃逸看门狗在训练中四次告警。它把“会写代码”与“会打穿目标”放进同一套评测，也让编码模型的安全边界第一次成为发布说明的主语。
[原文](https://cognition.com/blog/swe-2-1)

**2. 蚂蚁开源 Ling-3.2-air：124B 只激活 5.5B，MIT 协议**
11 月 23 日，蚂蚁 inclusionAI 在 Hugging Face 上传 Ling-3.2-air：混合线性 MoE，124B 总参数、5.5B 激活，35 层 KDA 加 7 层门控 MLA，512 个路由专家取 8 个；13 个 FP8 分片约 129GB，MIT 协议，原生 262K 上下文并可扩到 1M，支持文本、图像与视频。模型卡称 AutomationBench-AA 从 15.7% 升到 61%，Terminal-Bench 4.0 从 0% 到 27%，AA 智能指数 23；SGLang 首日镜像给出 4 卡 256K 的部署配方，但 124B 的权重仍不是单卡能装下的。对想自建长文档与视频理解管线的团队，这是 2026 年少见的“权重先行、发布会缺席”样本。
[原文](https://huggingface.co/inclusionAI/Ling-3.2-air)

**3. Ideogram 4.6 上线：原生 4K，多步编辑榜 1,892 分**
11 月 19 日，Ideogram 4.6 开放 Generate 与 Precise Edit 两个接口，支持原生 4K 输出、最多 5 张参考图与可选蒙版，4K 精修定价 0.60 美元/张。官方引用的 Rapidata 多步图像编辑榜上，它以 1,892 分排第一，高于 4.5 的 1,866 与 Seedream 5.5 的 1,805；五类编辑任务的可用率为 83%–90%，卖点是连续改图时画面漂移更小、颜色更准。图像编辑的竞争点正从“生成得多漂亮”转向“改完之后其余部分还能不能保持原样”。对设计团队来说，原生 4K 意味着少一次放大，多一轮可回退的修改。
[原文](https://ideogram.ai/models/4.6)

**4. 反事实世界模型：不用配对数据，R² 从 46% 推到 96%**
11 月 18 日，LeCun 团队在 arXiv 提交《Counterfactual World Models》：论文先证明只训练事实转移的世界模型无法回答“换一个动作、同一股扰动会怎样”，随后给出近似——当动作与扰动是两股独立力，反事实结果可写成“观测结果 + 新动作预测 − 旧动作预测”，无需配对数据、额外参数或重训。在 4 个渲染模拟器上，同一 JEPA 的反事实 R² 从 38%–46% 升到 86%–96%，另 4 个冻结编码器为 81%–95%；事后规划成功率 35%–69%，高于知道模拟器却不知道扰动的 oracle 的 14%–23%。这条近似把“反事实”从因果推断的教科书里拉回到两次前向加一次加法，代价是遇到接触、折叠坐标与乘性故障时会失效。作者把失效边界写得很具体，读者不必把 96% 当成通用结论。
[原文](https://arxiv.org/abs/2611.24778)

## 开源与工程

**5. Sarvam-105B-1711：Apache 2.0，65GB 权重塞进单卡**
11 月 22 日，印度 Sarvam 发布 Sarvam-105B-1711：105B 总参数、10.3B 激活的 MoE，Apache 2.0，同时放出 BF16 与 65GB NVFP4 两套权重，后者可在单张 RTX PRO 6000 上跑到每用户 99 token/s。八项对话代理基准均值从 2 月的 42.6 升到 73.9，混合价格从每百万 token 0.44 美元降到 0.15；IFBench 32.5→82.5、Tau2 46.2→85.5、印度语套件 30.1→79.1，Tau2 四次全对率 69.5%，略高于 Gemini 3.5 Flash-Lite 的 63.9%。对一个需要在本国语言、私有数据和低单价之间取平衡的市场，这类中等体量开放权重比前沿旗舰更有落地价值。
[原文](https://www.sarvam.ai/blogs/sarvam-105b-1711)

**6. MemOnce 把 1M 上下文的 KV 缓存缩小 1,313 倍**
11 月 20 日，南京大学与中科大团队在 arXiv 提交 MemOnce：只缓存最后一层的一小部分隐状态作为记忆，再按需解码回各层 KV，并让稀疏索引跨层只算一次。适配 Qwen3.5-4B 后，1M 上下文的状态缓存缩小 1,313 倍、prefill 延迟降 6.9 倍、吞吐升 3.5 倍，五项记忆基准平均提高 5.77 分；论文给出约 25MB 对 32GB 的显存对比，并已放出代码，但仍是单团队预印本，尚无独立复现。如果后续被推理引擎接住，长上下文服务的显存账单可能先于模型能力本身发生变化。
[原文](https://arxiv.org/abs/2611.27973)

**7. Transformers v5.24.0 接入 Inkling 2 Mini，导出器收紧信任边界**
11 月 21 日，Hugging Face 发布 Transformers v5.24.0，新增 Inkling 2 Mini、Apertus 1.5 与多语种 Omnilingual ASR 支持，统一缓存 API，并改进多模态与 serving；ONNX、OpenVINO、ExecuTorch 导出程序现在无需 trust_remote_code 即可加载，而可执行任意代码的 pickle .pt2 则必须显式开启。对把训练、推理与本地运行时串起来的团队，这类版本决定模型定义能否直接落到 vLLM、SGLang 或 llama.cpp，而不是再写一层适配。版本号之外，更值得看的是导出链路把“默认信任远程代码”改成了显式选择。这类底层库的迁移通常不显眼，却会在升级当晚决定训练脚本能否原样跑完。
[原文](https://github.com/huggingface/transformers/releases/tag/v5.24.0)

**8. Ollama 0.40.18 给 Inkling 2 提速，Metal 上融合残差**
11 月 24 日，Ollama 发布 0.40.18：在 Metal 后端融合 Inkling 的残差混合，让 MLX runner 跨架构共享 tensor helper 与模型初始化，并把 llama.cpp 升到 b12651、移除 Jetpack 5 支持。官方称 Inkling2 在 Metal 上的 decode、prefill 与 draft 轮次都更快；这类小版本不改变模型能力，却决定本地跑新模型时首 token 与长上下文吞吐能否跟上，升级前要先确认旧显卡的驱动与 Jetpack 版本。本地运行器的竞争，最后往往落在驱动兼容与一次升级要改多少配置上。
[原文](https://github.com/ollama/ollama/releases/tag/v0.40.18)

*本期由站点自动生成 · 信息以原文为准。*
