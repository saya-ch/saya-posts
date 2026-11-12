---
title: Qwen4-Max 开 1M 上下文预览，AlphaGenome 2 因果变异识别率翻倍
date: 2026-11-13T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期的主线是“能力开始按场景拆开卖”。阿里云把 Qwen4-Max 放进 11 月 11 日至 25 日的限时预览，1M 上下文、39 万输出 token，只走 Token Plan；DeepMind 的 AlphaGenome 2 用 1.07 亿份单细胞图谱把因果变异识别率从 16% 提到 30%，代价是单次预测更贵。安全侧，OpenAI 的 Control Room 已覆盖研究基础设施，从下达停止指令到切断输出最长 35 秒。工程侧，Codex Security 0.10.0、Astryx v0.7.16 与 Pydantic AI 2.68.0 都在补“能不能放心交给 agent”的细节，微软则在同一日退役五款 Copilot Studio 模型。如果只想先看一条，建议从 Qwen4-Max 的窗口期与 AlphaGenome 2 的算力账读起，逐条数字与边界见正文。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 阿里云开放 Qwen4-Max 预览：1M 上下文，最大输出 39.3 万 token**
11 月 11 日 10:00 至 11 月 25 日 24:00（UTC+8），Qwen4-Max-Preview 以限时预览形式上线，仅支持 Token Plan 调用、不开放按量付费，预览期内模型行为冻结，结束后模型标识直接下线。它支持文本、图像与视频理解，上下文 1M token，思考与最终答案共享 393,216 token 的输出预算，并兼容 OpenAI Chat Completions、Responses 与 Anthropic Messages 三套接口。对需要长文档与视频分析的团队，这十几天刚好够做一轮同题对比，但把预览模型 ID 写进长期工作流要谨慎。
[原文](https://www.alibabacloud.com/help/en/model-studio/qwen4-max-preview)

**2. DeepMind 发布 AlphaGenome 2：1.07 亿单细胞图谱，因果变异识别率 16% 升至 30%**
新模型改用“一段 DNA 加一个条件”的架构，每次只预测某一种组织、细胞类型或细胞状态下的调控效应，输入上限为 100 万碱基。它在 TenK10K 基准上把因果变异识别率从 16% 提到 30%，方向性基因效应预测从 23% 提到 43%；学术用户可免费走 API，商业机构需申请私有预览，官方也提醒更高分辨率意味着更高算力成本，因此新模型与原版并行而非替代，全基因组粗筛仍用旧模型更划算。
[原文](https://www.techrepublic.com/article/news-google-deepmind-alphagenome-2-genetic-variants/)

**3. OpenAI 上线 Control Room：停止指令到切断模型输出最长 35 秒**
这套集中式系统已部署在研究基础设施上，统一追踪训练与评估中的 agent 采样、数据处理等推理负载，并在判定风险时阻断输出。35 秒的延迟主要来自推理授权缓存层，OpenAI 称正把它压到零；Control Room 还被设计成在其他系统故障、甚至自身宕机时仍能执行停止动作。这相当于把“能不能停”从流程承诺变成可测量的时间指标。
[原文](https://www.theregister.com/ai-and-ml/2026/11/11/openai-builds-a-kill-switch-for-its-research-models/5309006)

**4. arXiv 论文：让 agent 用普通文档自己整理记忆，三项基准最多高 3.1 个百分点**
SoMem 不再依赖预定义记录与固定归类规则，而是让模型自己决定摘要什么、如何分组、何时修订主题，每条摘要都回链到原始交互作为证据，运行时再校验更新并派生时间、关系与近期视图。在 LoCoMo、BEAM-100K、LongMemEval-S 上，它分别比最强基线高 2.66、3.10、2.00 个百分点，整理好的记忆还能跨模型复用。潜台词是记忆结构不必由工程师预先写死，模型可以像整理笔记一样维护它。
[原文](https://arxiv.org/abs/2611.11929)

**5. 微软今日退役五款 Copilot Studio 模型，Standard 线路统一迁往 GPT Chat Latest**
11 月 12 日起，Standard harness 上的 GPT-4.1、GPT-4.1 Mini、GPT-5 Chat、GPT-5 Auto 与 GPT-5 Reasoning 自动迁移到 GPT Chat Latest；GitHub Copilot harness 的 GPT-5 Chat 改迁 GPT-6 Luna，GPT-5.6 Sol 改迁 GPT-6.1 Sol。美国以外受跨地域数据限制的环境继续保留 GPT-4.1 与 GPT-5，官方建议先盘点受影响的 agent，再重跑工具调用与追问缺失信息两类用例。对把 agent 放进生产的企业，这不是改个模型名，而是重新验收一次行为。
[原文](https://msmessagecenter.com/MC1506318/)

## 开源与工程

**6. Codex Security 0.10.0 发布：CLI 加 TypeScript SDK，扫描默认 5 美元上限**
11 月 12 日的 0.10.0 版本把安全扫描做成可编排的工程流程：`cs scan` 支持指定路径、Git diff 与深度模式，能验证候选漏洞、生成补丁并复核修复，还新增 CrowdStrike FCS IaC 发现导入，结果可导出 SARIF、JSON 或 CSV。仓库自带 GitHub Action 示例，默认给单次扫描设 5 美元预算，后端可切 OpenAI、Bedrock、OpenRouter 等；项目当日收 663 星，总数 12,126。把安全扫描接进 CI 的门槛，也从“买平台”降到了“装一个 CLI”。
[原文](https://github.com/openai/codex-security)

**7. Meta 开源 Astryx：内部养了八年、13,000 个应用在用的设计系统**
Astryx 以 MIT 协议进入 Beta，基于 React 19 与 StyleX，提供 150 多个无障碍组件、7 套主题、暗色模式与配套 CLI，v0.7.16 的变更集在 11 月 12 日生成。它的 API、文档和 CLI 被有意设计成“人怎么写、agent 就怎么建”，组件源码可用 swizzle 整体弹出；仓库当日涨 1,789 星，总数 26,034。设计系统愿意为 agent 重写文档与命令行，说明入口之争已经蔓延到组件库。
[原文](https://github.com/meta/astryx)

**8. CopilotKit OpenBot 1.0.2：给每个 AI 同事一台独立电脑，策略默认拒绝**
OpenBot 以 MIT 模板开放，每个 Bot 拥有独立容器、浏览器配置与工作目录，浏览器和 shell 调用必须先过 CEL 策略、先写审计行再执行，deny 优先于 allow，策略缺失时一律拒绝。它通过 AG-UI 接入 LangGraph、CrewAI、Claude Agent SDK 等执行端，提供 macOS、Windows 桌面版与 Kubernetes Helm 部署；仓库 11,275 星，单日 +291。它的取舍很明确：能力可以给足，但每一步都要留下可追溯的记录。
[原文](https://github.com/CopilotKit/OpenBot)

**9. Pydantic AI 2.68.0：把审批与追问收进一次 run，截断时自动切换备用模型**
11 月 11 日的版本新增 RequestInput，让 agent 在运行中向人请求批准或补充信息，AskUser 被放进延迟管线并标记回答者；同时加入 OutputTokenLimitExceeded，当响应撞上 max_tokens 时按 ModelFallback 顺序切到下一个模型。子 agent 任务可作为 Temporal、DBOS 的子工作流运行，Workspace.run 也支持为每条命令单独指定工作目录。这些改动指向同一件事：agent 的失败要能被人类接住，而不是只报一个超时。
[原文](https://github.com/pydantic/pydantic-ai/releases/tag/v2.68.0)

*本期由站点自动生成 · 信息以原文为准。*
