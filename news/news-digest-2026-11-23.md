---
title: 黎曼猜想写成 1700 万行 Lean，AMD 旗舰 GPU 的 PPA 最高提升 14%
date: 2026-11-23T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 本期最重的一条来自数学：Alpöge 把黎曼猜想的证明写成 1700 万行 Lean，175,187 个模块全部编译通过，Lean 与 Mathlib 社区称在其库中成立，但离 Clay 的正式认定还差合格期刊发表、两年观察期与数学界普遍接受。工业侧，OpenAI 让模型改写 AMD 旗舰 GPU 的 SystemVerilog，PPA 最高提升 14%，其首席科学家又提出用困惑度给全行业设开发上限。工程侧，一次 200 万条金丝雀测试揭开了推理转售链的泄密面，OpenRouter 已下线 3 家供应商、摘掉 125 个端点的零留存标签。想看清每个数字的边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 黎曼猜想写成 1700 万行 Lean，175,187 个模块全部编译通过**
11 月 20 日，Levent Alpöge 把黎曼猜想的证明写成一份 1700 万行的 Lean 库；次日 Lean 与 Mathlib 社区发表联合声明，称在其发布的库中陈述与证明都成立。MIT 的 Andrew Sutherland 复算全部 175,187 个模块无报错，五个主定理只依赖三条标准公理，两个独立检查器还各自带形式化可靠性证明。但 Clay 研究所的规则要求先在合格期刊发表、再等两年并获数学界普遍接受，Alpöge 也还没给出人类可读的论文，Tao 称消化这套证明需要时间。
[原文](https://leanprover-community.github.io/blog/posts/riemann-hypothesis/)

**2. OpenAI 首席科学家提出用困惑度给全行业设开发上限**
11 月 22 日，Jakub Pachocki 发表《Perplexity and Intelligence》，主张把困惑度（等价于交叉熵损失）做成跨实验室的共同标尺，并据此给训练设行业上限。他给出 OpenAI 模型自 GPT-3.5 以来的困惑度下降曲线，提出在理解与监控能力达到某个里程碑前，各家承诺不越过按困惑度划定的红线。问题在于困惑度依赖分词器与评测语料，低困惑度不直接等于下游能力，Pachocki 自己也承认这条尺度的下端仍未知。
[原文](https://openai.com/index/perplexity-and-intelligence/)

**3. 模型改写 AMD 旗舰 GPU 的 SystemVerilog，PPA 最高提升 14%**
11 月 18 日，OpenAI 披露其模型已参与 AMD 旗舰 GPU 核心 IP 的设计：模型重写描述电路的 SystemVerilog RTL，在对方深度优化过的基线上把功耗、性能与面积（PPA）最多改善 14%，部分改动已进入流片提交文件。OpenAI 同时说这套方法先用在自己的 Jalapeño 芯片上，功能正确性交由 Synopsys 的验证工具确认，双方还将合作训练会用 EDA 工具的 GPT-Synopsys。14% 是上限而非均值，PPA 也是三项指标的合成，读这个数字时要看具体基线。
[原文](https://openai.com/index/advancing-chip-design-with-our-models/)

**4. Ai2 开源研究智能体 Theorizer：先读文献，再给带引用的理论**
11 月 19 日，Ai2 在 Asta 平台上线研究智能体 Theorizer，并把代码开源：它先按问题检索文献、生成抽取模板，再把跨论文的发现拼成带引用的解释，并标出自己认为新颖的论断。官方给出的两个案例里，神经科学家一次拿到 50 条关于 VIP 中间神经元的候选理论，而他原本只期待一两条；肺癌团队则用它把“20%–30% 无已知驱动突变”的腺癌拆成三条可检验假设。Ai2 明确提醒它会漏检论文、误判原创性，理论仍要研究者自己核对。
[原文](https://allenai.org/blog/theorizer-in-asta)

## 开源与工程

**5. 200 万条金丝雀提示揭出转售链泄密，OpenRouter 下线 3 家供应商**
11 月 21 日，Ars Technica 报道 Dragos 的 HD Moore 在 7 月至 11 月向推理链投放约 200 万条带唯一金丝雀的提示：至少 3 家托管商被入侵，攻击者在入口处复制请求，偷到的 AWS 密钥被调用数万次，部分标记为零留存的请求同样被抄走。在 OpenRouter 点名的 MiniMax 链路上，系统在超过一半的提示里回抓了秘密 URL，零留存请求中约三分之二也未能幸免，抓取通常在两三天后出现。11 月 22 日 OpenRouter 下线 3 家供应商、把 125 个端点移出零留存名单，Together AI 也承认有样本被未授权下载。
[原文](https://arstechnica.com/security/2026/11/a-leaky-ai-supply-chain-is-exposing-secret-user-data-at-an-alarming-rate/)

**6. Claude Code 2.1.330 接入企业后台，定时任务上限延到 30 天**
11 月 20 日，Claude Code 发布 v2.1.330：登录 Anthropic Enterprise Admin Console 管理的组织后，会自动套用该组织的设置并走其云厂商或网关；新增内置图表 mod“easel”，在 Pro、Max 以及部分 Team/Enterprise 的 macOS 与 Linux 上按需画图。自托管 runner 新增“工单到注册”“工单到首个会话”两项耗时指标，定时任务与 /loop 的运行上限从 7 天延长到 30 天（会话需持续运行）。它还修掉了超长对话压缩后卡死数分钟、以及多进程同时写用户设置导致丢失的问题。
[原文](https://github.com/anthropics/claude-code/releases/tag/v2.1.330)

**7. agentmemory 用一套本地记忆服务喂饱十几个编码代理**
11 月 22 日的 GitHub 日榜上，agentmemory 以 30,517 星成为记忆层的代表：它在本地跑一个记忆服务，用 12 个钩子抓取会话、17 个技能教代理何时取用，并向 Claude Code、Codex CLI、Copilot CLI、Cursor、Gemini CLI 等暴露同一套 54 个 MCP 工具。项目以 Apache-2.0 开放，无模型密钥时退回 BM25；其 LongMemEval-S 组件测试的 recall-all@5 为 84.7%，比词法基线高 12.6 个百分点，any-hit@5 为 96.2%。这些是检索指标，不等于最终回答质量。
[原文](https://github.com/rohitg00/agentmemory)

**8. Replit 开源 Amuse：把个人代理装回自己的电脑**
11 月 19 日，Replit 用官方账号开源 Amuse：一个只跑在个人电脑上的自托管代理，Apache-2.0，Node 22.9+ 与 PostgreSQL 14+，一条 docker compose 命令起服务。它用 gpt-6.2-sol 做主模型、gpt-6-luna 处理快速步骤、gpt-image-2 出图，能开一个可接管的活动浏览器、做网页调研、设提醒，并在发送、提交表单或付款前弹审批卡。项目没有登录，默认只监听本机，官方明确说不要把它暴露到网络；凭据放在加密 vault 里，模型只看到替身 token。
[原文](https://github.com/replit/amuse)

**9. Kiro IDE 1.5.11 把 SSH、Dev Container 与云会话收进桌面**
11 月 20 日，Kiro IDE 1.5.11 把远程工作区做进桌面端：可通过 SSH 连自己的 Linux 机器（x86-64 或 ARM64），也可用仓库里的 devcontainer 在 Docker 中打开项目，或直接把已有的 Kiro Cloud 会话拉进新窗口。编辑器留在本地，文件、终端与代理跑在远端，断开连接后代理仍可继续干活；三种模式共用同一套聊天与工具链。
[原文](https://kiro.dev/changelog/ide/1-5/)

*本期由站点自动生成 · 信息以原文为准。*
