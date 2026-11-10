---
title: 智能体运行时补上预算与回滚，机器人边跑边改
date: 2026-11-10T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头从模型分数挪到跑起来的系统。Nous Research 的 Hermes Agent v0.23.0 用约 1 万次提交把桌面端搬进浏览器、把多端会话收进单一 gateway，并给付费工具加上价格标签与预算上限；Hugging Face 的 LeRobot v0.7.0 让机器人在运行中被自然语言改任务，同时把训练与推理拆到远端 GPU。模型侧，Meta 的 Muse Ember 以 100 万上下文和每百万 token 0.1/0.5 美元切入高频 agent 调用，SuperCLUE 11 月榜则显示 19 个模型的能力已经按类别分裂。另有 CWE-bench v2 的 100 个安全修复任务与 OpenClaw 的 166 次提交，都是“能不能稳定交付”的细节。想看每条的数字与边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Meta 发布 Muse Ember：100 万上下文，每百万 token 0.1/0.5 美元**
Meta 11 月 9 日推出 Muse Ember，把“小模型”做成 agent 的高频执行层：上下文 100 万 token，单次最多输出 512K，原生读文本、图像、音频与视频，输入/输出定价 0.10/0.50 美元每百万 token，缓存输入 0.004 美元。官方称它在 Artificial Analysis 智能指数拿 36 分，与 Claude Haiku 5.5 的满档持平，但每个任务成本约 0.136 美元，只有后者的三分之一；AutomationBench-AA 在 657 个 Zapier 工作流上完成 37.8%，Terminal-Bench 4.0 为 40.9%。这不是又一个“最强模型”，而是把子 agent、路由、抽取这类重复调用压到可批量部署的价格；超过 200K 输入后价格升到 0.30/1.50 美元，长上下文仍要算账。
[原文](https://research.meta.ai/blog/muse-ember)

**2. SuperCLUE 11 月榜：19 个模型，第一与第二各守一半类别**
SuperCLUE 11 月 5–6 日放出 19 个模型的评测，综合第一是 Gemini 4 Argon，第二是 GPT-6.1-Sol，Kimi K3.1 Pro 第三。分项却完全分裂：GPT-6.1-Sol 以 69.23 拿 agentic coding 第一、76.32 拿数学与科学推理第一；Gemini 4 Argon 以 83.19 拿幻觉控制、95.63 拿超长上下文，并与 Sol 并列 68.97 的精确指令跟随。开放权重里，DeepSeek V4.1 Flash 以 65.38 领跑编码，Darwin-180B-RSI-R3 以 73.68 领跑数学与科学。榜单由评测团队自己发布，分数不是独立复核，但它说明“总分第一”已经很难概括一个模型，选型必须回到具体任务。
[原文](https://huggingface.co/posts/SeaWolf-AI/235085300016491)

**3. Ai2 开源 ToyBlocks：85,336 条演示，把机器人失败变成可复现实验**
Ai2 11 月 6 日发布 ToyBlocks，作为 FoundationWorlds 的第一个受控桌面环境：首版含 85,336 条长程演示，每条串起约 6–12 个技能，合计近 1,600 小时、1.44 亿帧，全部由仿真自动生成，单张 GPU 上可跑到实时的 24 倍。代码 Apache-2.0、数据 ODC-BY，附带 Molmo2-4B-tb 与 ManiFlow-tb 两个示例模型；ManiFlow 技能在仿真里成功率 80.0%、真机 81.1%，但分层积木拼写任务从仿真 55.5% 掉到真机 50.0%，说明 sim-to-real 的缺口仍在。更有意思的是让 agent 自己写技能说明：成功率从默认描述的 24% 升到 63%，高于人类专家写的 36%。
[原文](https://allenai.org/blog/toyblocks)

## 开源与工程

**4. Hermes Agent v0.23.0：一次 1 万次提交，把桌面端搬进浏览器**
Nous Research 11 月 5 日发布 v0.23.0“Hearth”：自 v0.22.0 起累计约 10,080 次提交、1,950 个合并 PR、8,720 个文件改动、666 位贡献者。最实质的改动是运行时收口——每台机器只跑一个 gateway，CLI、TUI、Desktop、编辑器、bot 与 cron 共用同一份会话，避免同一条消息被写两次；Desktop 可整体在浏览器标签里打开，`/rewind` 能回到任意消息重发或分叉，Group Chat 支持自动路由，付费 MCP 工具会在调用前显示单价与累计花费，超过预算先询问。插件目录从约 600 条增到 1,050 条，内置技能从 58 个减到 44 个，单次运行少发约 1,570 个 prompt token。多端同源解决的是状态一致性，预算与回滚解决的则是“敢不敢让它长时间跑”。
[原文](https://github.com/NousResearch/hermes-agent/releases)

**5. LeRobot v0.7.0：机器人跑到一半，可以改指令、提问、换目标**
Hugging Face 11 月 5 日发布 LeRobot v0.7.0，核心是把语言放进控制回路：交互式 rollout 允许在机器人不中断的情况下切换子任务、询问它看到了什么、或用 VLM 规划器自动纠偏。这一版同时接入四个 world action model（Cosmos 3、Long-WAM、FLUX 3 Action、LaWAM）与四个新 VLA，训练栈补上 FSDP2、HSDP、分布式 checkpoint 与 FP16/EMA；推理可把大模型放到远端 GPU 的 `lerobot-policy-server`，机器人端只留轻量计算。数据侧支持从 Hub、Storage Bucket 或 LanceDB 流式读取，不必先落盘；所有仿真器收进一个 vector-env core，RoboLab 的 120 个桌面任务可直接跑。Cosmos 3 Nano 16.5B 与 Edge 4.6B 在 LIBERO 上分别报 97.90% 与 97.95%，但真实价值是“边跑边改”把调试循环从重启机器人缩短到一句话。
[原文](https://huggingface.co/blog/lerobot/lerobot-release-v070)

**6. Mistral 把 Managed Agents 放进 Studio：同一份 Python 代码，三种跑法**
Mistral 11 月 5 日扩展 Studio，Managed Agents 进入公开预览：开发者用 Python Agents SDK 定义模型、指令、typed tools、MCP 连接器、skills、secrets、guardrails、hooks 与子 agent，同一份代码可在本地进程、`agents serve` 自托管、或 `agents deploy` 交给 Mistral 托管。平台补上 workflow builder、人工审批、Apps CLI、托管搜索索引、OpenTelemetry 追踪与离线/在线评测；MCP server 让 Vibe 或其他编码 agent 直接测试并一键部署。ABANCA、Accenture、Alten、BNP Paribas、Capgemini、CMA CGM 与 TCS 被列为生产用户。它卖的不是模型，而是“agent 跑几天、崩了能续、审批能等”的运行时；预览期的 API 与限额仍可能变，接入前要留迁移余量。
[原文](https://mistral.ai/news/build-with-studio)

**7. CWE-bench v2：100 个安全修复任务，最强 agent 仍漏 19%**
Collinear AI 11 月 3 日在 Hugging Face 开放 CWE-bench v2：100 个公开任务覆盖 91 种 CWE、8 种语言，另有 150 个隐藏任务，合计 185 种弱点；评的是 agent 在真实开源仓库里自己定位并修漏洞，且不能破坏原有功能。11 个模型里 Gemini 4 Argon 以 67.8% pass@1、81.0% pass@4 领先，Claude Opus 5.5 为 67.3%/78.0%，DeepSeek V4.1 Flash 以 57.0% 领跑开放权重；95 个任务至少被一个模型解出，但冠军在每次尝试里仍会漏掉 19%。成本前沿也很直白：DeepSeek 每次 rollout 0.04 美元，Opus 以 1.83 美元换约 10 个点，Gemini 再往上到 3.92 美元。任务、评分器与 harness 全开放，意味着这类分数第一次可以自己复跑。
[原文](https://huggingface.co/blog/cwe-bench)

**8. OpenClaw v2026.11.2：166 次提交，补的是消息丢失与状态错位**
OpenClaw 11 月 8 日发布 v2026.11.2：166 次提交、113 个 PR、64 位贡献者，新增 Knowledge Scribes 的捕获工作流、可选 PostgreSQL 状态后端与迁移工具，并重做 Control UI 的移动端与桌面导航。修复项更值得看：WhatsApp 发送者映射晚到时私聊会丢、Telegram 轮询重试会拖住 Gateway 关闭、Discord 回调在内存里过期、记忆检索拒绝模型用 `limit` 参数、非拉丁频道名解析到错误房间。对一个要长期在线的 agent 网关，这些不是边角料，而是“消息到底有没有送到”的账本；PostgreSQL 后端则把单机状态变成可迁移、可备份的部署选项。
[原文](https://docs.openclaw.ai/releases/2026.11.2)

*本期由站点自动生成 · 信息以原文为准。*
