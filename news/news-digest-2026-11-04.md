---
title: 千问接入 Apple 智能今日开测，Gemini 4 Carbon 登顶，DeepSeek 开源 Super-agent
date: 2026-11-04
tags: [资讯, 每日更新]
status: published
summary: 本期最大看点是国行 Apple 智能落地：11 月 4 日起随 iOS 开发者 Beta 开放测试，千问以系统级扩展接入 Siri、写作工具与视觉智能，12 月随 iOS 27.3 正式版推送。谷歌两天连发两款模型，旗舰 Gemini 4 Carbon 以 59 分登顶 Artificial Analysis 综合智慧指数，世界模型 Genie 4 则首次原生生成音频并支持实时改写场景。开源侧，DeepSeek 公开 Super-agent 双层调度架构并同步开源原型，GitHub Trending 上用纯 C 在消费级硬件跑 MoE 大模型的 colibri 星数冲上 4.7 万。此外还整理了 OpenRouter 中国模型调用量连续 27 周领先、Kimi K3.1 Pro 权重开放节点等动态。点击阅读正文查看全部 9 条精选。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 国行 Apple 智能今日开测：千问接入 Siri、写作工具与视觉智能**
苹果 10 月 31 日官宣与阿里达成合作，首批 Apple 智能功能 11 月 4 日起随最新 iOS 开发者 Beta 开放测试，12 月随 iOS 27.3 正式版面向中国用户推送；千问以「千问创作」「询问千问」等形态嵌入系统级入口，用户请求与回复不会被用于模型训练。
[原文](https://www.sohu.com/a/1095272930_115978)

**2. Gemini 4 Carbon 登顶：59 分反超 Claude Opus 5.5 与 GPT-6 Astra**
谷歌 10 月 28 日发布旗舰模型 Gemini 4 Carbon，在 Artificial Analysis 综合智慧指数中以 59 分超越 Claude Opus 5.5（58 分）与 GPT-6 Astra（57 分）重返榜首；漏洞挖掘评测 SEC-bench Pro 取得 88.2%，无依据陈述较上代减少约一半，本周先向网络安全防御人员开放。
[原文](https://news.cnyes.com/news/id/6640535)

**3. 谷歌 Genie 4 世界模型预览：原生生成音频，可实时「改戏」**
Google DeepMind 10 月 27 日推出世界模型 Genie 4 研究预览版，是 Genie 系列首个实时生成中同步产出环境音、音乐与对话的模型；用户可用自然语言在中途改天气、调镜头、加角色，场景一致性延长至约 2 分钟，并由 Gemini 3.8 Flash 担任实时引导智能体。
[原文](https://m.36kr.com/p/4044846951420800)

**4. OpenRouter 数据：中国大模型 Token 调用量连续 27 周超美国**
乌镇峰会披露的最新统计显示，10 月 26 日至 11 月 1 日当周中国大模型 Token 调用量连续第 27 周超过美国，全球周调用量前五的模型中中国占四席、DeepSeek 位居榜首；硅谷多家公司及德国近半数中小企业都在使用中国开源模型。
[原文](https://www.163.com/dy/article/LASLPTOI0514R9OJ.html)

**5. Kimi K3.1 Pro 权重承诺今日前开放：2.8 万亿参数、百万上下文**
月之暗面 10 月 24 日发布的 K3.1 系列首款模型采用 2.8T 总参数、104B 激活的 MoE 架构，支持 100 万 token 上下文，Terminal-Bench 4.0 得分 65.7%，高于 GPT-6 Astra；官方承诺完整模型权重于 11 月 4 日前开放，届时有望成为该榜单排名最高的开放权重模型。
[原文](https://tech.ifeng.com/c/8xTWmzaVbfg)

## 开源与工程

**6. DeepSeek 开源 Super-agent 架构：让系统接管会话调度**
DeepSeek Harness 团队 10 月 31 日公布「Super-agent」架构构想并同步开源原型，采用常驻 Router 层分派任务、多 Worker 会话并行执行的双层设计，辅以事件驱动、存算分离与外部 Watchdog；实验中 p50 任务完成时间 12.4 分钟、p95 为 15.7 分钟，代码以 MIT 协议开源。
[原文](https://finance.sina.com.cn/tech/csj/2026-11-01/doc-inixepty4335109.shtml)

**7. GitHub Trending：colibri 用纯 C 在普通硬件上跑 MoE 大模型**
分层推理引擎 colibri 本周持续霸榜，星数约 4.7 万：它把热专家放在 GPU、温专家放内存、冷专家从 NVMe 流式读取，纯 C 实现零依赖，让 DeepSeek V4、Kimi K3 等前沿 MoE 模型在既有消费级硬件上运行。
[原文](https://github.com/JustVugg/colibri)

**8. beacon.sh 登上 GitHub Picks：给 Claude Code、Cursor、Codex 加统一记忆层**
开源项目 beacon.sh 提供跨 harness 的记忆与追踪层，让 Claude Code、Cursor、Codex 等编码智能体共享上下文与执行轨迹，11 月 3 日以 363 赞位居 Product Hunt AI 榜首，本地 CLI 可免费使用。
[原文](https://pmer.cn/en/ai/projects)

**9. 微软 Agent Framework 推进：Python 核心包更新至 1.24.0**
agent-framework-core 于近日发布 1.24.0 版本，整合 AutoGen 的多智能体编排与 Semantic Kernel 的企业级特性，支持 MCP 与 A2A 协议，并提供带规划、待办跟踪、上下文压缩与可观测性的 Harness Agent 抽象。
[原文](https://pypi.org/project/agent-framework-core)

*本期由站点自动生成 · 信息以原文为准。*
