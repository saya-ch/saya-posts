---
title: 模型学会直接改写权重，Agent 开始给自己搭推理栈
date: 2026-11-06T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 本期两条主线都指向“少改一点、多做一点”。MIT CSAIL 让 GPT-6 Astra 只改 Qwen3.5-0.8B 的 0.016% 权重，就把 MATH-500 从 7.9% 拉到 48.1%；CMU 的 ServerBench 则让编码 Agent 直接设计推理服务，Claude Sonnet 5.5 交出的系统达到 vLLM 参考的 4.87 倍性价比。工程侧，ZStack 把能管 1,003 台 GPU 的 GPUWorks 以 Apache-2.0 开源，Vercel 的 agent-desktop 用系统无障碍树替 Agent 省下 78%–96% 的 token。这些进展说明，Agent 的竞争正从“会不会做”转向“能不能改到关键处、能不能把成本压下来”。九条精选把从权重编辑到集群调度的链路拆开讲，值得逐条对照。想核对每条数字与原始链接，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Astra 只改 0.016% 权重，Qwen3.5-0.8B 的 MATH-500 从 7.9% 升到 48.1%**
MIT CSAIL 11 月 2 日发布 Weight-Space Programming：不给梯度、不许写训练循环，只让 GPT-6 Astra 在 Python 里读写权重。它把 0.8B 模型反复“思考打转”的问题定位到第 15、19 层 MLP 的 64 个输出列，改动 131,072 个权重（0.016%），MATH-500 从 7.9% 升到 48.1%，GSM8K 从 13.8% 升到 67.1%，IFEval 从 21.0% 升到 62.3%；同一套方法还给 Breakout 手写了一个 17,253 个权重的网络，作者称其表现超过训练 1 亿步的 PPO 策略。
[原文](https://rikiyat.github.io/blog/weight-space-programming)

**2. NorthCUA 开源 397B 电脑操作模型：OSWorld-Verified 88.64，Apache-2.0**
NorthCUA 11 月 2 日一次放出 Nano 9B、Mini 35B-A3B 与 Pro 397B-A17B 三个电脑操作模型，全部 Apache-2.0，基于 Qwen3.5，必须配官方 Harness 使用。Pro 在 OSWorld-Verified 上拿到 88.64% 成功率、WebArena-Verified 73.89%，ScreenSpot-Pro 93.30%；代价是 122 个权重分片、约 793.63 GB 磁盘占用，官方提醒“激活参数不等于显存需求”。同门的 Nano 与 Mini 在同一评测下分别为 78.90% 和 85.30%，说明这套“经验回路”在小尺寸上也能复用。
[原文](https://huggingface.co/beipo-ai/NorthCUA-Pro)

**3. CMU 让编码 Agent 自己设计推理服务：Sonnet 5.5 交出 vLLM 的 4.87 倍性价比**
CMU 11 月 4 日的 ServerBench 把“搭一套 LLM 服务系统”整个交给编码 Agent：给定模型、GPU 与负载，Agent 自行改实现、调并发，再在保留会话依赖的回放中按“每美元延迟合格 token 价值”计分。11 个 Agent、3 种部署下，33 次运行有 22 次通过独立验证、13 次超过最强基线；Claude Sonnet 5.5 在 Gemma E4B 上做到 2.97 美元/美元，是 vLLM 参考的 4.87 倍。作者总结的三条经验是：扩容要与限流同步、实验要贴合真实负载、验证要覆盖新引入的执行路径。
[原文](https://arxiv.org/abs/2611.06841)

**4. RoboColosseum 开放真实机器人考场：100+ 台机器人、10,000 小时免费评测**
由 Hugging Face、BitRobot、新加坡具身智能中心与 NUS 的 MAGIC Lab 共建的 RoboColosseum 11 月 4 日上线，覆盖 100 多台真实机器人、140 多个野外场景与 4 类平台，向学术团队提供 10,000 小时免费评测；官方称已完成 20,000 多次物理试验、6,800 个任务。评测采用双策略盲测，失败录像也一并公开；其 11 月 2 日的论文在 124 个站点做了 5,699 组配对比较，Unitree G1 的任务完成率从 13.3% 提升到 23.2%，说明“能做出部分动作”和“能完成任务”仍有明显距离。
[原文](https://www.humanoidsdaily.com/news/robocolosseum-robot-ai-benchmark-unitree-g1)

**5. 多轮 RL 并没有让 Agent 更会“走每一步”：单步更优仅 42.8%**
11 月 5 日的 arXiv 论文重新拆解多轮强化学习：在 Qwen3-4B/8B 上，RL 后的 Agent 只在 42.8% 的 FrozenLake 状态和 46.8% 的 DeepCoder-MT 状态里选出比基座更好的单步动作，平均单步质量甚至略低；真正带来提升的是纠错，同样的失败状态下 RL 模型有 51.3% 能恢复成功，基座只有 38.6%。作者提出的 BudgetMix 把交互预算当作训练变量，在 25%、50%、75% 预算下分别带来 7.5、5.3、4.0 个百分点的提升，满预算时不掉点。
[原文](https://arxiv.org/abs/2611.06483)

## 开源与工程

**6. ZStack 开源 GPUWorks：40 MB 控制面管 1,003 台 GPU，Apache-2.0**
ZStack 11 月 4 日把 GPUWorks 以 Apache-2.0 开源，控制面含 Web UI 约 40 MB，单个静态二进制、状态存一个 SQLite 文件。官方在阿里云跨 7 个地域的演练中，用 1 台服务器管理 1,003 台租用 GPU 机器，4 分钟内拉起 500 多个推理副本，网关扛住 2,000 并发请求，服务端峰值 9 核 CPU、1.6 GB 内存；内置 390 个模型、1,224 条启动规格，NVIDIA、昇腾、PPU、海光 DCU 与 Apple Silicon 可混布在同一集群，节点通过出站 WebSocket 接入，不需要开放入站端口。
[原文](https://github.com/flitai-io/gpuworks)

**7. Vercel 开源 agent-desktop：用系统无障碍树替代截图，token 省 78%–96%**
Vercel Labs 的 agent-desktop 11 月 3 日更新到 1.1.0，用 Rust 写成 CLI 与 C 库，让 Agent 读取 macOS、Windows、Linux 的无障碍树，拿到带稳定 ref 的 JSON 再点击、输入、回读验证，而不是靠视觉模型猜坐标。项目提供 60 个命令（56 个可用），默认无头执行、不移动鼠标也不抢焦点，稠密 Electron 应用可用 skeleton 逐层下钻；官方自报 token 消耗因此减少 78%–96%，代价是目标应用必须暴露可用的无障碍信息。
[原文](https://github.com/vercel-labs/agent-desktop)

**8. Scrapling 0.5.1：给 Agent 的抓取层，页面改版后仍能找回元素**
D4Vinci 的 Scrapling 11 月 4 日发布 0.5.1，GitHub 约 8.6 万星。它把单次 HTTP 请求、Playwright 动态渲染、Cloudflare Turnstile 绕过与全站爬取放进一套 Python API：解析器会记住元素的上下文指纹，页面改版后按相似度重新定位，并用 adaptive=True 复用旧选择器；内置 MCP Server 与 Agent Skill，抓取结果先去隐藏内容和提示注入再交给模型。0.5 版把工具箱整体重写给 Agent 使用，不过清洗隐藏文本并不等于内容可信，下游仍需校验。对需要把网页变成 Agent 数据入口的团队，这层“会自我修复”的抓取比换模型更直接。
[原文](https://github.com/D4Vinci/Scrapling)

**9. KServe 进入 Thoughtworks 雷达 Trial：把 vLLM、SGLang 收进 Kubernetes**
Thoughtworks 11 月 4 日的新一期技术雷达把 KServe 放进 Trial 环，看中的是它不自己做推理，而是用 Kubernetes 自定义资源编排 vLLM、SGLang、Triton 等运行时，把模型服务的生命周期与执行引擎解耦。KServe 已是 CNCF 孵化项目，新的 LLMInferenceService 比旧的 InferenceService 更贴近生成式负载；雷达同时提醒其生态仍在演进，适合在能承受风险的项目上先试。它把模型服务拆成声明式资源与可替换运行时，团队可以按负载选择引擎，对已经在 Kubernetes 上跑推理的平台团队，这相当于多了一层控制面。
[原文](https://www.thoughtworks.com/radar/platforms/kserve)

*本期由站点自动生成 · 信息以原文为准。*
