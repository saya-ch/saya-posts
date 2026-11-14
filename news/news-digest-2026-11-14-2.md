---
title: 开源语音识别压到 2.28%，27B 机器人模型放开权重
date: 2026-11-14T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头放在“开放权重之后，能不能真的用起来”。Salesforce 的 SOV-Transcribe-1 把开源语音识别的 AA-WER 压到 2.28%，代价是 53GB 检查点、只支持 7 种语言且不做流式；Runway 则把 27B 的 Praxis-1 以 Apache 2.0 放开，RoboLab-120 成功率 46.2%。模型侧还有两条值得对照：IBM 用 LLMON 结构把间接提示注入成功率打到 0.02%–5.6%，Hugging Face 让多向量检索在 4 倍压缩后仍保留 97.8% 的质量。工程侧，SGLang v0.5.23 在 SM90 上出现 33%–45% 的解码回退，升级前最好先看复现数据。如果你正在选本地部署、语音转写或检索方案，这一期的取舍比分数更有参考价值。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Runway 开放 Praxis-1：27B 世界动作模型，RoboLab-120 成功率 46.2%**
11 月 11 日，Runway 以 Apache 2.0 开放 27B 的 Praxis-1，MoE 扩散 Transformer 同时预测视频与动作；在 NVIDIA RoboLab-120 的 1,200 个 episode 上成功率 46.2%，高于 FLUX 3 Action 的 42.9%、Cosmos3 Nano 的 36.8% 与 π0.5 的 28.0%。在未见过的 RT-1 数据上微调后，SimplerEnv 成功率 80.0%，原策略为 52.4%。相比九月只给 launch partners 使用，这次把权重、推理与 LoRA 微调代码一起放出，Apache 2.0 允许商用，但 27B 的 MoE 仍要 H100、B200 或 RTX 5090 级硬件。
[原文](https://runway.com/news/company-news/praxis-1-is-now-available)

**2. Salesforce 开源 SOV-Transcribe-1：AA-WER 2.28%，但只做非流式**
11 月 12 日，Salesforce 以 Apache 2.0 放出 SOV-Transcribe-1：26.6B 总参数、4.6B 激活，Whisper large-v3 音频编码器接 Gemma 4 26B A4B 解码器，BF16 检查点约 53GB。它在 Artificial Analysis 非流式榜拿到 2.28% AA-WER，低于 Nemotron 3.5 Transcribe 的 2.81% 与 Whisper Large v3 Turbo 的 4.62%；但只支持 7 种欧洲语言、单段最长 30 分钟且不做流式，单张 RTX PRO 6000 开启 DFlash 草稿模型后可每秒转写 1,039 秒音频。模型卡把取舍写得很直白：它适合夜间批量转写已录制通话，而不是实时字幕；30 分钟上限、缺少说话人分离与词级时间戳，意味着后处理仍要自己补。
[原文](https://www.orcarouter.ai/blog/sov-transcribe-1-salesforce-open-weights-speech-model)

**3. IBM 放出 LLMON 版 Granite：间接提示注入成功率降到 0.02%–5.6%**
11 月 12 日，IBM 在 Hugging Face 放出 Granite-4.1-3B/8B 的 LLMON 实验版本（Apache 2.0），用 4.6B token、440 万条结构化样本把指令与数据拆成带 ID 的块，只有被 exec 指针引用的指令才执行。论文给出的间接提示注入攻击成功率从 53%–100% 降到 0.02%–5.6%，3B 与 8B 的通用能力基本持平（3B 均为 66.7，8B 从 74.2 降到 72.8）。这套方法不改架构，只把六个保留分隔符塞进未使用词表槽位，训练 token 的 93.6% 仍是通用、RAG 与工具调用数据；代价是上下文长度增加约 7%–15%，直接攻击与多轮攻击的防御仍弱。
[原文](https://arxiv.org/abs/2610.38120)

**4. Hugging Face 把池化写进训练：4 倍压缩后仍保留 97.8% 检索质量**
11 月 11 日，Hugging Face 发布 Sentence Transformers v6.3.0 与 token pooling 训练方法：普通 ModernBERT 多向量模型在 pool_factor=4 时保留 94.6% 的未池化 nDCG@10，加入池化感知损失后升到 97.8%。同一组 12 万条 MS MARCO triplets、一个 epoch 的对照里，4 倍池化把平均文档表示从 193.22 个向量压到 48.68 个，池化感知模型仍保住 97.8% 的质量；配套 Ward 聚类核在 5,000 条 SciFact 标题与摘要上把池化时间从 7.00 秒降到 1.02 秒，约 6.9 倍。对 RAG 团队来说，这等于在索引体积和召回质量之间多了一个可训练旋钮，但评测只覆盖 13 个 NanoBEIR 数据集、单 seed，生产前仍要用自家语料复测。
[原文](https://github.com/huggingface/sentence-transformers/releases/tag/v6.3.0)

**5. 腾讯混元发布 Hy Realtime：语音模型开始会唱歌、会调工具**
11 月 13 日，腾讯混元发布实时语音模型 Hy Realtime，在语音理解、指令遵循与拟人化表达之外新增 Agentic 工具调用和唱歌能力，并已在腾讯元宝语音通话与腾讯客服上线，近期将登陆腾讯云开放 API。官方把下一阶段重点放在双工交互、长期个性化记忆与前后台 Agent 交互；目前普通用户可在元宝 App 选择“初语”音色体验，网页 Demo 只开放部分能力。它延续了国内语音模型“先上自家产品、再开 API”的路径：元宝验证实时通话，腾讯客服验证工具调用与任务闭环，腾讯云再承接企业客户；唱歌与工具调用能否稳定跨场景，目前还缺延迟、并发与成功率的公开数字。
[原文](https://www.ithome.com/1/026/825.htm)

## 开源与工程

**6. SGLang v0.5.23 在 SM90 上解码回退 33%–45%，问题仍开着**
11 月 9 日的 issue 记录：8 张 H20-3e（SM90）跑 DeepSeek-V4.1-Flash 与 DSPARK 推测解码，从 v0.5.22 升到 v0.5.23 后，并发 12 时每步解码在 8K/16K/32K/64K 上下文分别慢 44.8%/45.2%/40.0%/33.4%，并发 1 则基本不变（19.5→19.1ms）。把 Hopper 上的 use_deep_select_decode 强制关掉可回收 21%–24% 吞吐，但每步仍比旧版慢 25.8%–34.8%；GPU kernel 总时间只涨 9.6%，维护者尚未给出根因。复现实验把变量压得很窄：同一镜像、同一 patch、只改一行配置，8K、并发 12 的吞吐就从 862 升到 1,070 tok/s，说明瓶颈一半在 acceptance、一半在 kernel 之外的 host 调度；生产团队最稳妥的动作仍是先钉回 v0.5.22。
[原文](https://github.com/sgl-project/sglang/issues/49336)

**7. EngineAgent 把端侧推理适配交给 agent：12 个任务成功率 75%**
11 月 10 日的论文提出 EngineAgent，把引擎抽象层、跨层依赖与硬件后端知识做成可复用的层级索引，再用 builder 改代码、auditor 在真机上验证。覆盖 3 个移动端 LLM、4 组引擎-后端、CPU/GPU/NPU 共 12 个任务，两种底层 LLM 的端到端成功率均为 75%，最强 coding agent 基线为 33.3%；判定要求编译通过、真机执行成功且输出 logits 与参考实现一致，同时降低了单任务时间与 API 成本。它的价值不在于“让 agent 写代码”，而在于把移动端推理最耗人的跨层调试拆成可验证步骤：转换、构图、算子、后端任何一层出错，auditor 都要在真机上跑通并比对 logits；但覆盖范围只有 3 个移动 LLM 与 4 组引擎-后端，跨芯片迁移仍需重新验证。
[原文](https://arxiv.org/abs/2611.14356)

**8. Perplexity Decider V1.3 上线：不写回答，只返回概率与原文 span**
11 月 12 日，Perplexity 上线 Decider V1.3：不生成文本，而是对传入的文本、JSON 或图片返回带概率的类型化答案——二选一给 yes 概率，多选题最多 255 个选项各给概率，评分题给每档概率与期望分。新版本增加 locate，在 Extractions 路由上抽取文本与列表字段，并返回每个值的原文 span 与置信度；上下文 262K，当前免费，适合分类、路由、审核与评分这类由应用代码卡阈值的场景。它把“模型输出一段话、应用再解析”改成“模型直接输出可阈值化的分布”，对风控、工单路由和打分卡更友好；限制是只适合闭集问题，开放生成仍要另一个模型，免费额度能否支撑生产量级也还需观察。
[原文](https://openrouter.ai/perplexity/pplx-decider-v1.3:free)

*本期由站点自动生成 · 信息以原文为准。*
