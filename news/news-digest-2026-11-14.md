---
title: Gemini 4 Hydrogen 上线，百度双模型押注稀疏注意力
date: 2026-11-14T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期的主线是“把模型塞进真实工作流，而不是再刷一次总榜”。谷歌把 Gemini 4 Hydrogen 放进 Antigravity，日常编码提速 2.5 倍，还能与 Carbon 组成 Orchestrator，按任务难度自动分工；百度则用反馈式 KV 压缩的稀疏注意力，把搜索级深度思考模型的推理成本压下来，并用 4000 字文本驱动微表情视频。资本侧，Mirendil 以 10 亿美元 A 轮把“几百个 agent 替一个研究员做实验”推上台面，3.5 个月、不到 1000 万美元就训出百亿参数级编码底座。开源侧，Agno 3.2 把 A2A 推到 1.0，Chrome 的 Modern Web Guidance 一天涨 1875 星，Anthropic 则把整套 Agent 控制面搬进企业自己的机房。想看清每条的数字与边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Gemini 4 Hydrogen 上线 Antigravity：日常编码快 2.5 倍，终端榜 57%**
11 月 12 日，谷歌在 Antigravity 上线 Gemini 4 Hydrogen，并新增 Orchestrator 选项，把 Hydrogen 与 Gemini 4 Carbon 按任务难度组合。官方称日常编码任务比 3.8 Flash 快 2.5 倍，思考深度分 low、medium、high、xhigh 四档；Artificial Analysis 智能指数 51 分（最高档 54 分），Terminal-Bench 4.1 得 57%，是 3.8 Flash 18.2% 的三倍多。同日起向 Gemini 应用、搜索与 AI Studio 推送，API 定价为每百万 token 输入 1.5 美元、输出 7.5 美元。
[原文](https://antigravity.google/blog/gemini-4-hydrogen-in-google-antigravity/)

**2. 百度发布文心闪电与文心一镜：稀疏注意力压成本，视频生成便宜 50%**
11 月 12 日百度世界大会发布文心闪电深度思考与文心一镜微表情两款应用模型，均上架千帆平台。闪电用反馈式 KV 压缩的稀疏注意力 BSA，在解码中按历史输出与状态压缩记忆，官方称可大幅降低推理成本，并兼容 DSA、HCA 等高压缩注意力；一镜用文本、视觉、语音三态建模，支持 4000 字文本驱动，覆盖 6 大维度、200 个子维度的描述体系，视频生成成本较业界低 50%，极速版可 1:1 实时出片。
[原文](https://www.ncsti.gov.cn/kjdt/kjrd/202611/t20261113_263211.html)

**3. Mirendil 拿 10 亿美元 A 轮：一名研究员 3.5 个月训出编码底座**
11 月 12 日，前 DeepMind、Anthropic 研究员创办的 Mirendil 宣布 10 亿美元 A 轮，Kleiner Perkins 与 Clearlake 联合领投，谷歌等跟投；公司未披露估值，彭博此前称其按 50 亿美元融资。其 Lor 平台让数百个 agent 并行写代码、跑实验，官方称一名研究员借助 agent 在 3.5 个月内、不到 1000 万美元算力从零训出百亿至五千亿参数的 Mirendil-Base，agent 写了 99% 以上代码；衍生模型 Fern 支持 1000 万 token 输入，约为 Claude Opus 5.5 的 10 倍。
[原文](https://siliconangle.com/2026/11/13/mirendil-raises-1b-in-funding-to-speed-up-ai-model-development/)

**4. arXiv 论文：不会行动的决策模型，仍能当好后训练老师**
11 月 12 日挂上 arXiv 的论文比较 3 个决策模型与 3 个通用 LLM 在 ALFWorld 上的表现：作为 actor 时，六个模型都会被更容易的子目标带偏；但作为 judge 看完整轨迹，它们能把真正推进目标的步骤挑出来，关键在问“这一步有没有帮助”，而不是“任务会不会成功”。用 JEV 当 teacher 做 GRPO，Qwen2.5-1.5B 在 ALFWorld 从 72.8% 提到 82.9%，WebShop 从 56.8% 提到 71.4%。
[原文](https://arxiv.org/abs/2611.17167)

**5. arXiv 论文：Bandit 路由四个模型，API 成本最多降 65.6%**
11 月 13 日的论文 Ballad 挑战“模型越贵越好”的假设：在 5 个 NP-hard 组合优化问题上，更大更贵的模型并没有稳定优势。它把 LLM 选择做成在线 bandit，用 DAG 沿启发式解的“家谱”传播信用，并对过期观测打折；在 4 个不同能力与价位的模型池里，最终匹配或超过事后最优的单模型，部分任务上超过所有单模型，API 成本相对始终调用最强模型最多降 65.6%。
[原文](https://arxiv.org/abs/2611.17241)

## 开源与工程

**6. Agno 3.2 把 A2A 推到 1.0：任务可订阅续跑，审批默认只认发起人**
11 月 12 日，Agno 发布 v3.2.0，把 A2A 协议从草案推到官方 1.0：/v1/message:* 与 /v1/tasks:* 路由被移除，AgentOS 与 A2AClient 必须一起升级；A2A 任务改为入库，支持 send、stream、get、cancel、list、subscribe 与暂停续跑。Super Agent 不再只做路由，而是直接列取并运行 agent、team、workflow。另有三个容易踩的破坏性变更：OSManager 的 tool_call_limit 默认 8 变 24、num_history_runs 7 变 5，Slack 审批默认只允许发起者点通过。仓库约 4.3 万星。
[原文](https://github.com/agno-agi/agno/releases/tag/v3.2.0)

**7. Chrome 的 Modern Web Guidance 一天涨 1875 星：把 232 项特性塞进 agent**
11 月 13 日，GoogleChrome/modern-web-guidance-src 单日涨 1875 星、总数到 4158，244 次 fork。它把 232 项现代 Web 特性与 188 个真实开发场景做成 agent 技能，通过 npx modern-web-guidance@latest install 注入编码 agent 的上下文，并提供 legacy-scan 扫描旧模式；项目由 Chrome、Edge 团队与社区维护，Apache 2.0，明确是预览版。它解决的是训练数据里 legacy 代码太多、模型明明知道新 API 却写不对的问题。
[原文](https://github.com/GoogleChrome/modern-web-guidance-src)

**8. 英伟达 SkillSpector：装技能前先扫 17 类、68 种漏洞模式**
英伟达的 SkillSpector 把“装技能前先安检”做成 CLI：覆盖 17 类、68 种漏洞模式，包括提示词注入、数据外泄、权限提升、供应链、过度代理、记忆投毒、MCP 工具投毒等，支持 Git 仓库、URL、zip、目录与单文件，默认静态扫描，可选 LLM 语义比对“描述与代码是否一致”。结果可输出 terminal、JSON、Markdown 与 SARIF，能直接接 CI；依赖漏洞走 OSV.dev，断网时回退离线库。它把 agent 技能从“看起来无害”变成可审计的发布门。
[原文](https://docs.nvidia.com/skills/scanning-agent-skills)

**9. Anthropic 开放自托管 Managed Agents：控制面搬进企业机房**
Anthropic 把 Claude Managed Agents 的自托管版本开进私有测试：harness、沙箱、凭证库与会话历史全部跑在企业自己的云或机房，模型请求则可发往 Anthropic、Amazon Bedrock、Google Agent Platform、Microsoft Foundry 或自建网关。它延续四条设计原则：大脑与手分离、凭证不进沙箱、按域名白名单限网、每步留可导出的 OpenTelemetry 记录；ZDR 协议同样适用于发往 Claude 的请求。适合那些 agent 已经能改构建、跑发布、处理告警，但代码不能出内网的团队。
[原文](https://claude.com/es/resources/articles/self-hosted-claude-managed-agents)

**10. Vercel 公测 UI SDK：给 agent 行为零件，不给现成组件**
11 月 12 日，Vercel 把 UI SDK 开进公测，Apache 2.0、版本 0.1.0-beta.1。它不给 agent 现成组件，而是给 createPopup、createSelection、createTypeahead 这类行为零件，官方称 50 多种，实测 91 个，覆盖 React、Vue、Svelte、Solid 与 Web Components；单个弹层约 36 KB。装上后再给 agent 加对应 skill，让它按零件拼界面，而不是从零写键盘导航与焦点管理。
[原文](https://github.com/vercel-labs/ui)

*本期由站点自动生成 · 信息以原文为准。*
