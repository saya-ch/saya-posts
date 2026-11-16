---
title: 1.6 万亿 LongCat-2.5 放开权重，编码工具链补上评审与验收
date: 2026-11-16T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头对准“权重之后”的两件事。美团把 1.6 万亿参数的 LongCat-2.5 以 MIT 许可放开，Terminal Bench v2.1 自评 88.4 分，但自托管指南只写了 194 卡昇腾方案；NVIDIA 则把 2B 的 Alpamayo 2 Edge 放到 Hugging Face，端到端延迟压到 160–190 毫秒。另一条线是 Agent 交付链：Meta 的 Astryx 用 150+ 组件统一人和 Agent 的设计参考，agent-browser、elephant-goldfish 与 Plannotator 分别补上浏览器验收、独立评审和结构化回传。开放权重越来越强，真正的差距却落在能不能被部署、被 review、被回滚；这九条恰好把部署、评审与验收三段串了起来，正文逐条给出数字与原始链接。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 美团 LongCat-2.5 放开 1.6T 权重：MIT 许可，Terminal Bench v2.1 自评 88.4**
11 月 9 日，美团 LongCat 团队在 Hugging Face 放出 LongCat-2.5 权重，总参数 1.6 万亿、每个 token 约激活 480 亿，MIT 许可允许商用、修改与再分发，仓库体积约 3.56 TB。官方自评 Terminal Bench v2.1 为 88.4 分，对照表里 Claude Opus 5 是 89.1、DeepSeek-V4.1-Flash 是 90.6；BrowseComp 拿到 91.9，但评测用的是自研轻量 harness、只开放两个浏览工具，模型卡也注明未标星成绩“仅供参考”。更值得部署方注意的是自托管指南：194 卡昇腾 Atlas A2 集群、8 台 Decode 加 4 台 Prefill，通篇基于 CANN，没有给出 Nvidia GPU 路径。
[原文](https://huggingface.co/meituan-longcat/LongCat-2.5)

**2. NVIDIA 放出 2B 的 Alpamayo 2 Edge：车端到轨迹 160–190 毫秒**
11 月 16 日，NVIDIA 在 Hugging Face 发布 20 亿参数的 Alpamayo 2 Edge 权重，面向 DRIVE AGX Thor、按 10 Hz 车载节奏设计；经 FP8 校准、ONNX 导出、TensorRT 与 DFlash2 推测解码后，从摄像头输入到轨迹输出的延迟为 160–190 毫秒。在 AlpaSim 闭环里，两次由模型决策导致的事故之间平均行驶 15.15 公里，对比 40 亿参数 Qwen-Drive-1.0 的 0.25 公里；最小平均位移误差 0.53 米对 1.68 米。训练数据约 11.5 万小时多摄像头视频、370 万条因果链，闭环强化学习把性能再抬 48.9%；权重用 OpenMDW-1.1，代码 Apache 2.0。
[原文](https://i.gasgoo.com/news/70478112.html)

**3. SO-Arena 用 2,919 个任务拆开“能力”与“可靠”，21 个模型参评**
新加坡国立大学 11 月 13 日公开 SO-Arena，覆盖 11 个领域、5 类能力家族，共 2,919 个任务，评了 21 个 System One 模型。Fastino 的 GLiDE（27B）与 PelaAI 的 KnowLine-31B 以 70.2 并列开放权重榜首，GPT-6 Sol 的参考分为 82.3；但能力第一的 GLiDE 在决策可靠性上只排第七，可靠性第一的 JevAny-27B 能力只排第八。所有开放权重模型都能在单张 H100 上 0.3 秒内完成任务，说明“快”已不是差异点；真正拉开差距的是选项顺序稳定性与“没有正确答案”时的识别率。
[原文](https://arxiv.org/abs/2611.15777)

**4. NeurIPS 2026：让模型用可执行代码解释自己，一致性可自动验证**
11 月 10 日上传的《Code as Explanations》提出用一段程序解释模型在一批输入上的行为，而不是为每个样本生成一段思维链；程序能运行，就能自动核对它是否复现模型输出。作者定义 Consistency 指标，在 3 个开源代码模型、8 个任务上验证，发现部分设置下模型能写出高度一致的代码解释，且比逐条文本解释更容易验证和复用。论文被 NeurIPS 2026 接收，共 30 页、11 张图；它把“解释”从一段说服人的文字，变成一个可以跑、可以失败的工件。
[原文](https://arxiv.org/abs/2611.14704)

## 开源与工程

**5. Meta 开源 Astryx 设计系统：150+ 组件，本周新增 13,501 星**
Astryx 是 Meta 在内部养了 8 年的设计系统，支撑 13,000+ 应用，本周以 MIT 许可开源后新增 13,501 星、总量到 28,878，位列 GitHub 周榜第一。它提供 150+ 可访问组件、七套主题、暗色模式、模板与 CLI，基于 React 19 和 StyleX，目前仍是 Beta。它的定位不是“又一个组件库”，而是让 API、文档和 CLI 共用一套参考：Agent 先查组件清单再写代码，而不是凭印象拼 UI；代价是 React 19 门槛、StyleX 迁移成本，以及 charts 等包还没有稳定版。
[原文](https://github.com/meta/astryx)

**6. agent-browser 0.43.0：把页面读成带引用的紧凑文本，让编码工具自己验收**
Vercel Labs 的 agent-browser 在 11 月 12 日发布 0.43.0，本周新增 4,217 星、总量到 51,635。它是 Rust 写的浏览器 CLI：snapshot 把页面读成带 @e1 引用的紧凑文本，Agent 点击引用后再 snapshot 检查结果，浏览器在命令之间保持打开；点击目标被遮挡、禁用或不可见时直接报错，不会假装成功。项目 Apache 2.0，支持 CDP 连接、多 session 隔离和 MCP。它补的是“单测过了但页面还是坏的”这一步，适合把前端验收交给 Agent 跑，但不能替代 Playwright 的稳定断言。
[原文](https://github.com/vercel-labs/agent-browser)

**7. Google 的 elephant-goldfish 0.1.1：让“金鱼”不带上下文审设计**
Google 开源的 EGM 插件在 11 月发布 0.1.1，本周新增 5,060 星、总量到 19,781，Apache 2.0，并注明不是 Google 官方支持产品。它把流程拆成两个角色：Elephant 保留项目上下文、起草规格；Goldfish 评审者刻意不带前面的对话，只看草稿和引用材料，专门找未说明的假设与实现风险；人负责架构取舍。它还提供 drift report，把设计文档和当前代码对比，报告哪里已经偏离。边界同样清楚：评审需要能创建全新子 Agent 的会话，且不能替代测试。
[原文](https://github.com/google/elephant-goldfish-model)

**8. Plannotator 0.28.65：把评审意见结构化地送回编码 Agent**
Plannotator 在 11 月 15 日发布 0.28.65，本周新增 1,719 星、总量到 11,848，采用 Apache 2.0 / MIT 双许可。它是本地浏览器里的评审面：Agent 提出计划、规格、HTML 或改完代码后自动打开，人直接在页面上标注、评论、审 diff，再一键把带位置的反馈发回原来的会话；支持 Claude Code、Codex、Copilot CLI、Gemini CLI 等十余种 agent，也能审 GitHub PR 与 GitLab MR。它的价值不是替人做 review，而是把“复制粘贴一段意见”变成结构化输入，代价是 review 质量仍取决于人。
[原文](https://github.com/backnotprop/plannotator)

**9. SGLang 0.5.23 在 Hopper 上解码退化 33–45%，建议先锁回 0.5.22**
11 月 12 日公开的 issue #49336 给出了可复现的回归：在 8 张 H20-3e 上跑 DeepSeek-V4.1-Flash，从 v0.5.22 升到 v0.5.23 后，并发 12 时每步解码时间在 8K 上下文增加 44.8%、16K 增加 45.2%、32K 增加 40.0%、64K 增加 33.4%，而并发 1 基本不变。关掉 Hopper 上默认走到的 DeepSelect 能收回 21%–24% 吞吐，但主要来自接受长度而非步进时间；剩余 26%–35% 的差距在核外。GPU profile 显示内核总时间只涨 9.6%，作者因此怀疑调度与批处理的主机侧开销，建议生产先锁 v0.5.22。
[原文](https://github.com/sgl-project/sglang/issues/49336)

*本期由站点自动生成 · 信息以原文为准。*
