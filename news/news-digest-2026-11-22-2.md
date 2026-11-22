---
title: 扩散模型 32 token 一块出字，vLLM 把 KV 缓存交给实测显存
date: 2026-11-22T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 本期把镜头从“谁更大”移到“怎么交付”。NVIDIA 的 Nemotron-Labs-Sigma 用 14 个开放检查点把连续扩散语言模型做成可下载的 3B/8B 家族，每轮去噪 32 个 token；两篇 11 月 19 日的基准则分别考“模型会不会自己造 Agent”和“Agent 会不会自己搭 RAG”，最高分只有 59.1% 与 56.1。训练侧，814 次 RL 实验说明裁判的单题知识几乎不传，对类别的偏见却会整体迁移。工程侧，vLLM 0.34 按实测空闲显存分 KV 缓存，SGLang 把 1.0 的兼容策略摆上桌面，Cloudflare 与 univer 则分别放开抽取模型权重和表格内核。想看清每个数字的边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. NVIDIA 放出 14 个连续扩散检查点：3B/8B，每轮去噪 32 个 token**
11 月 16 日公布的 Nemotron-Labs-Sigma 不逐 token 自回归，而是在 16 维连续嵌入空间里去噪，每轮提交 32 个 token 的块，共 14 个开放权重检查点，覆盖 3B 与 8B 的 base、instruct、NFE 2/4/8/16 蒸馏版与自推测变体。8B base 在 NVIDIA 六任务套件上均分 71.16，高于 Qwen3-8B 的 68.91，8B instruct 的 GSM8K 为 91.96，但 AIME25 只有 17.00；单张 H100 报告 120–140 token/s，8B 自推测在 GB200 上平均为自回归解码的 3.46 倍。蒸馏版把“数学每步 8 token、代码每步 4 token”写进模型卡；许可证允许商用，代价是上下文仅 4,096 token、英文限定，且 vLLM 与 SGLang 目前都不支持这一架构。
[原文](https://research.nvidia.com/labs/genair/sigma/)

**2. AppSci-Bench：让模型自己造智能体，最高分只有 59.1%**
11 月 19 日的 AppSci-Bench 把“造智能体”拆成七类真实业务域，其中幻灯片生成、订单到收款、合规审查、客服通话洞察、CRM 洞察为新增，另两域改写自 tau3-bench，每域都配一套留出评测。七个前沿模型各给三次搭建机会，最好的 Claude Opus 5.0 平均最低分只有 59.1%，需求澄清一项为 65%；更棘手的是模型自建评测的饱和率高达 65%，即自测过关却过不了留出集。基准代码已公开，它指向的问题不是会不会写代码，而是会不会定义验收标准。
[原文](https://arxiv.org/abs/2611.25360)

**3. AutoRAGBench：六小时、单张 H800，智能体从零搭 RAG 拿到 56.1**
同日的 AutoRAGBench 把 RAG 系统开发做成九道开放题：不给起始代码，智能体要在单张 H800、六小时预算内读数据、搭管线并迭代调优。九种智能体配置里，GPT-6.1 Sol 配 Codex CLI 平均 56.1 分，高于人类 SOTA 组合的 44.4；在七道已有人工系统的题上，它从现成系统出发还能再涨 16.6 分。所有配置共用同一套六小时预算与单卡限制，避免靠堆算力拉开差距；论文同时分析开发轨迹，说明增益主要来自实验反馈驱动的多轮修改，而不是一次性生成。
[原文](https://arxiv.org/abs/2611.25582)

**4. 814 次 RL 实验：裁判的单题知识几乎不传，类别偏见却会迁移**
11 月 18 日的论文用 814 次 RL 实验，教 7–8B 模型回答闭卷题或选择弃答，追问“裁判的知识到底传了什么”。结论是单题知识几乎不传：裁判认可的错答只被记在那一道题上，换更严裁判也只相当于每百题多纠正约 1 题；但裁判对“某一类题”的偏见会整体迁移——12 道正确首府题里有 11 道被判错，模型便学会对新题拒答 98%，手工改这 12 个判定可把拒答率从 5% 拉到 90%。把“我不知道”选项挡住后，这些首府题仍有 94% 能答对，说明丢的是类别许可而不是知识。
[原文](https://arxiv.org/abs/2611.25191)

## 开源与工程

**5. vLLM 0.34.0 默认按实测空闲显存分 KV 缓存，旧 Runner 退场**
11 月 16 日的 vLLM 0.34.0 合并 1,061 个提交、412 位贡献者，默认打开可扩展 KV 缓存：缓存不再按 `gpu_memory_utilization` 估算，而是按预热与 CUDA graph 捕获后的真实空闲显存分配，MiniMax-M3 在 H200 上从 433k 扩到 672k token。代价是部分模型 TPOT 上升 6%–18%；Model Runner V1 被删除，默认 block size 从 16 改为 64，ROCm 7.2 镜像停发，`huggingface_hub` 需 2.0 以上。性能侧，Step5 在 4×GB200 上解码快 1.95 倍，GLM-5.3-Flash 吞吐提升 1.6 倍；同时新增 Mistral Large 4（1T 总参、52B 激活）等 10 个模型；升级前还要把 V1 的 logits processor 迁到新路径。
[原文](https://github.com/vllm-project/vllm/releases/tag/v0.34.0)

**6. Cloudflare 开放 Clairic 权重：27B 抽取模型加 9B flash**
Cloudflare 11 月 18 日把 Clairic 与 9B 的 Clairic-flash 开放权重，前者为 27B，专做字段抽取，新增图片与 PDF 输入，单次最多四张图或四页，表格行可对齐返回。官方内部评测里，跟踪客户断言 1,263/1,263 全中，并称图片读取的内部评测从 207 处错误降到 0；延迟上 flash 的文本 p50/p95 为 155/328 ms，Clairic 为 358/736 ms，均按输入 token 计费，flash 每百万输入 0.10 美元，12 月 1 日前两者同价。
[原文](https://blog.cloudflare.com/open-clairic/)

**7. univer 发 v1.6.0：表格内核单周涨 7,248 星，Agent 直接改单元格**
11 月 21 日 univer 发布 v1.6.0，仓库以 Apache-2.0 开放表格与文档内核；GitHub 周榜给它 +7,248 星、总量 30,438，是当周增量第一。它的卖点不是又一个聊天框，而是把公式引擎、Canvas 渲染、命令与撤销栈、OT 协同做成可嵌入 SDK，Agent 通过 Facade API 直接读写单元格并保存快照，不必先把表格转成文本。需要留意 AI SDK、Server SDK 与图表等商业插件另行授权，开源仓只覆盖 Web SDK 核心。
[原文](https://github.com/dream-num/univer)

**8. SGLang 1.0 追踪页：把“静默改配置”改成启动即报错**
SGLang 的 1.0 追踪页本周仍在进行时：定位是稳定性版本，不重构核心，只把“静默改配置”改成启动即报错，并首次写清 1.x 的兼容策略。页面列出 9 类已知静默行为，包括 `max_new_tokens` 被 KV 容量悄悄截断、DFLASH 把显式步数改成 1；sgl-router 0.1.0 已发布并取代 sgl-model-gateway，管理接口改用 admin API key，`--disable-cuda-graph` 与 `--enable-dp-attention` 两个旧参数被移除；请求字段也开始做 fuzz 校验，非法字段返回 4xx 而不是拖垮进程，0.5.x 升级要逐项对照。
[原文](https://github.com/sgl-project/sglang/issues/50045)

*本期由站点自动生成 · 信息以原文为准。*
