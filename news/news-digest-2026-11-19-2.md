---
title: OVH 把 7B 模型塞进 8GB 显卡，Meta 的调度器少用 43% 的 H100
date: 2026-11-19T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头对准“把能力塞进更小的机器、把调度交给真实信号”。OVHcloud 开源 Olala mini，7B 总参数只激活 1B，8GB 显卡就能跑到 160 token/s，给欧洲开源模型补上一个可本地部署的选项；Meta 的 Cap-and-Track 则用拒绝信号跟踪推理容量，生产环境里让一个分类服务少用 43% 的 H100。工程侧，Imbue 以 MIT 开放并行编码智能体桌面端 Sculptor，AgentScope 2.0.12 把“运行中插话”做成中间件，Qwen Code 0.28.0 把 headless、TUI 与子智能体收到同一条循环。另有 RAA 用图片分辨率管理智能体记忆、Memdoor 的单二进制编码智能体与 Orama Lake 的自托管 AI 栈。想看清每条的数字、代价与升级坑，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. OVH 开源 Olala mini：7B 总参只激活 1B，8GB 显卡跑到 160 token/s**
11 月 17 日，OVHcloud 发布首个 LLM 家族 Olala 的首个成员 Olala mini 7A1B：总参数 7B，每层 256 个专家，每个 token 只激活 1B，混合注意力把 Differential-TPA 与 Mamba-3 拼在一起，BF16 权重以 Apache 2.0 在 Hugging Face 开放。官方给出的部署数字是 8GB 显存即可运行，RTX A4500 上 160 token/s，纯 CPU 也有 50 token/s，上下文 32K；模型在 Jupiter、MeluXina 与 Jean Zay 三台欧洲超算上训练，折算约 700 万欧元机时。Qwant 已把它用于闪答，称回答快 20%、成本降到三分之一；年底的 1.1 版会把上下文扩到 128K，2.0 版规划超过 100B 参数。
[原文](https://corporate.ovhcloud.com/en-sg/newsroom/news/ovh-olala-mini/)

**2. 把智能体记忆渲染成图片：RAA 按相关度调分辨率，历史 token 省 57.5%**
11 月 18 日提交的 arXiv 论文提出 Resolution as Attention：把每轮交互渲染成图片，再用一个 0.6B 重排器估计它与下一步决策的相关度，按相关度动态调整读取分辨率，重要的交互保留细节，不重要的只留粗读。作者在 ClawGym-Bench 的 200 个任务与三个骨架上测到历史 token 最多降 57.5%，任务表现基本不变，峰值 KV 缓存降 47.8%；把它接到现有记忆方法上还能再省 54.1%。代价是依赖骨架构图的 OCR 能力，且每步重设分辨率会削弱前缀缓存复用，端到端延迟反而长了 4.3%，省 token 不等于省时间。
[原文](https://arxiv.org/abs/2611.24422)

**3. Meta 的推理调度器：只看拒绝信号，分类服务少用 43% 的 H100**
11 月 18 日提交的 arXiv 论文 Cap-and-Track 把推理调度从预设速率改成跟踪容量上限：请求在真正饱和的资源上快速失败，多余的活进入持久优先级队列，调度器只根据拒绝信号向上探测或回退，不必预测输出长度，也不用按机队规模反复调参。Meta 在生产 A/B 中用它跑一个分类器，比排队加固定速率的前代少用 43% 的 H100；稳态回放里 goodput 与调好的固定速率相差 1% 以内，拒绝比例约为后者的四分之一。系统每天承载 3 亿次以上在线请求，最忙的 24 小时处理 11 亿次；代价是回填高峰时低优先级任务等得更久。
[原文](https://arxiv.org/abs/2611.24891)

## 开源与工程

**4. Imbue 开源 Sculptor：用 Git worktree 并行跑多个编码 Agent**
Imbue 11 月 17 日公开 Sculptor 源码并以 MIT 协议授权，这是一个本地桌面应用，把多个编码 Agent 的并行会话、隔离检出、diff 审阅与编辑器同步收进一个界面；它直接接 Claude Code 与开源 Pi 的 harness，能读取结构化事件，做到崩溃后恢复会话、显示上下文占用比例。项目内部打磨约两年，公开仓库已有 2,100 多次提交；此前两个原型被放弃，原因是更强模型让额外脚手架失去意义。目前只提供 Apple Silicon macOS 与 Linux 构建，仍是研究预览，官方也坦言没有余力审大量外部 PR。
[原文](https://github.com/imbue-ai/sculptor)

**5. AgentScope 2.0.12：回复跑到一半，也能把新输入插进推理边界**
11 月 16 日发布的 AgentScope 2.0.12 把“运行中补充输入”做成正式能力：SteeringMiddleware 与 SteeringQueue 在推理边界注入新消息，不必中断正在执行的模型或工具调用，回执会区分已注入、未消费与失败；服务端同时补上可编辑的 FIFO 消息队列、客户端工具与后台任务状态接口。依赖侧切到 MCP SDK 2.x，模型卡新增 Gemini 4 Argon/Carbon 与 Kimi K3.1 Pro，实时语音加入 OpenAILiveModel 与 Gemini 3.8 Live Extended Thinking，另新增统一 ASR 接口。对做多轮 Agent 服务的团队，这次升级的重点不是新模型，而是把“人随时插话”变成可编排的运行时语义。
[原文](https://docs.agentscope.io/zh/versions/2.0.12/release-notes)

**6. Qwen Code 0.28.0：headless、TUI 与子 Agent 收到同一条 Agent loop**
11 月 17 日发布的 Qwen Code 0.28.0 把三条执行路径收拢到同一条 Agent loop：headless 会话、Ink 终端界面与子 Agent 默认共用同一套循环和工具批处理，便于在 CLI、Web Shell 与受管会话之间复现同一条轨迹；同时加入 Codex 风格的托管 worktree 会话、DingTalk 渠道适配，以及 qwen4-max 的原生 effort 梯度、缓存 TTL 与图片细节控制。发布说明还列出跨会话消息、会话产物头与按轮次 hook 统计等改动，并明确本版没有已知破坏性变更。它想解决的是编码 Agent 从单机 CLI 走向多入口服务后，行为在各表面之间漂移的问题。
[原文](https://github.com/QwenLM/qwen-code/releases/tag/v0.28.0)

**7. Memdoor 把编码 Agent 做成单二进制：一个决策模型先筛上下文**
这周公开的 Memdoor 以 Apache 2.0 开源，主打“一个二进制、无需 Docker 与 Python”，macOS、Linux、Windows 都能本地跑，也能通过 SSH 在远端机器上执行；它在 Agent 与模型之间加了一个决策模型，用来筛搜索命中、文件片段与日志，并决定本轮需要哪些工具。作者公布 44 次运行（22 对）的对比：决策开关两种设置下任务都通过，其中一次代码库问答的输入 token 少了 74%，但编辑与修复类任务没有节省，说明收益强依赖任务形态。另一组自测里，五个 Qwen3.8-27B Agent 在 LiveCodeBench Hard 前十题、五次运行下平均 9.8/10，作者自己的 harness 为 9.6，单次调用的 Fable 5 为 8.0；样本很小，数字应视为作者自测。
[原文](https://github.com/guregodevo/memdoor)

**8. Orama 把整条 AI 栈以 MIT 放开：C++26 引擎、差分版本控制与 0.6B 基座**
Orama 11 月 16 日以 MIT 协议发布 Orama Lake v0.1.1，把数据引擎 zetta、差分源码控制与构建工具 liquid、Kubernetes operator、Agent 运行时与 0.6B 基座模型 base-agent-0.6b 打成一个可自托管的栈，官方称从笔记本上的单节点 k0s 到空气隔离集群都能装，且 100% 能力可由 MCP 与 API 调用。它用 C++26 写核心，训练与推理可跨 CPU、GPU 和不同云做混合部署，并附带签名溯源、SBOM 与密钥扫描等供应链控制。需要留意的是入口仓库只放版本锚点、不含产品源码，v0.1.1 仍是早期版本，“替代 Palantir 与 Databricks”的说法也还没有独立验证。
[原文](https://github.com/oramasearch/lake-platform)

*本期由站点自动生成 · 信息以原文为准。*
