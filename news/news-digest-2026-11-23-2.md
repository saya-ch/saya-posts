---
title: 同一模型省下 28% token，代理评测开始考模型与 harness 的共同进化
date: 2026-11-23T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头对准代理的第二层：同一套模型，换一个 harness 就能省下可观的 token。Strands 开源通用 harness，官方称六项基准成本低 28%，在 Fable 5 上比 Claude Code 省 77%，Terminal-Bench 2.1 分数还更高；CoEvoBench 则把模型与 harness 的共同进化做成 24 小时赛题，最强模型缩小了五分之四以上的差距，却在长程编码上几乎全军覆没。研究侧还有冻结权重、只进化 harness 的 GEPA 路线，以及把不确定性留在隐空间的 LProp。工程侧，Dify 1.20 接上 OpenTelemetry 与文件扫描，GPUWorks 修掉 SGLang 长 prefill 误杀，Orca 与 Claude-BugHunter 则代表代理工具层的两种扩张方式。想看每条的具体数字与边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. CoEvoBench 把模型与 harness 的共同进化做成 24 小时赛题**
11 月 20 日提交的论文提出 CoEvoBench：在 5 个代理领域里，研究者从同一基座模型与初始 harness 出发，24 小时内自行训练模型、修改工具与提示，再提交一对组合到独立测试任务上评分。5 个前沿模型参与，最强的一个缩小了与指令微调参考系统之间超过五分之四的差距；交互类任务普遍成功，长程编码几乎全部失败。作者据此判断，当前模型能做出第一次有效改进，却很难把改进后的系统反馈变成下一轮改进。
[原文](https://arxiv.org/abs/2611.27116)

**2. 冻结权重只改 harness，GEPA 在 SWE-bench 上追回大部分收益**
11 月 19 日的论文把领域特化从“再训一个 checkpoint”拆成两条路：一条更新权重，一条只进化 harness，即系统提示、工具 schema、技能库与规划脚手架。作者用反射式进化优化器 GEPA 在 Codev 上跑 SWE-bench Verified、OfficeBench 与 Terminal-Bench，结论是权重更新单路径收益最强，harness 进化只用很小一部分算力就追回大部分收益，两者叠加最好；论文还指出 SFT 特化会让推理链变短、截断增多。
[原文](https://arxiv.org/abs/2611.25380)

**3. LProp 让不确定性随证据一起传下去，八种传播设置里 token 最省**
11 月 17 日的论文提出 Latent Uncertainty：不再用标量分数或长篇反思描述不确定性，而是在隐空间保留对后续修正有用的信息；LProp 再用继承来的表示与新证据更新下游答案分布，不需要额外采样轨迹。作者在选择题、是非题、开放问答与数学推理上评测，称其在多数指标上优于基线，并在全部 8 种传播设置中平均推理 token 最低。消融显示，真正带来错误恢复的是继承的隐内容，而不只是答案概率。
[原文](https://arxiv.org/abs/2611.23564)

**4. 长文生成改用段级评分，SH-PRM 在同等预算下压过结果级监督**
11 月 20 日的论文把长文生成拆成语义段，每段在给定前缀下生成，再用覆盖度、连贯性与相关性三项 rubric 打分，据此计算逐步优势并做局部策略更新。为单独观察监督粒度，作者在同一 GRPO 框架下对比结果级监督与 SH-PRM，在多个模型和两个长文基准上，SH-PRM 都稳定胜出，推理期强基线也被压过。作者把增益归因于信用分配变准，而不是预算变多。
[原文](https://arxiv.org/abs/2611.27348)

## 开源与工程

**5. Strands harness 开源：同模型六项基准省 28% token，Fable 5 上比 Claude Code 省 77%**
Strands Agents 团队发布 Apache-2.0 的通用代理 harness，Python 与 TypeScript 都只要一行 create_harness 即可接入 Bedrock、Anthropic、OpenAI、Google、Ollama 或 LiteLLM。官方在 EC2 上用 Harbor 分布式跑六项基准，称同样用 Claude 或 GPT 模型时成本低 28%、准确率基本持平；换到 Fable 5，成本比 Claude Code 低 77%，Terminal-Bench 2.1 分数还更高。省下来的主要来自默认上下文策略：工具结果超过约 1500 token 就截断，上下文用到 85% 触发摘要压缩。它默认带 shell、文件读写与网页工具，也能加载技能、把子任务交给内置助手；CLI 里用 /export 可直接导出 Python 或 TypeScript 代码。
[原文](https://github.com/strands-agents/harness-sdk)

**6. Dify 1.20.0 把追踪接到任意 OTel collector，并给知识库上传加病毒扫描**
11 月 18 日发布的 Dify 1.20.0 新增 OpenTelemetry provider，用 OTLP/HTTP 把应用与 Agent 的链路送到任意兼容 collector，同时适配 11 月 16 日退役旧接入点的 Langfuse Cloud 与只收新事件的 Langfuse v4。知识库上传可开启 FILE_SCAN_ENABLED，接入 Cloudmersive、Scanii、OPSWAT 或自定义端点，扫描失败默认拦截文件。人工审批被推进 Slack、飞书、Teams、钉钉与企业微信，人工输入节点的全局超时设为 604800 秒，也就是 7 天。
[原文](https://github.com/langgenius/dify/releases/tag/1.20.0)

**7. GPUWorks v26.11.4 修掉长 prefill 误杀：健康检查连败 3 分钟才退役实例**
11 月 18 日的 v26.11.4 针对 SGLang 的 PD 分离部署：此前长分块 prefill 期间 /health 会等调度器输出而超时，agent 在约 13 秒内连看三次失败就把实例判死，decode 侧于是报出 “Lost connection with prefill instance”。新版本让 SGLang 实例设置 SGLANG_ENABLE_HEALTH_ENDPOINT_GENERATION=0，/health 只反映进程是否在服务；实例启动后要连续 3 分钟健康检查失败才退役。升级顺序是先 server 后 agent，本次没有数据库迁移。
[原文](https://github.com/flitai-io/gpuworks/blob/master/docs-en/releases/v26.11.4.md)

**8. Orca 把十几个 CLI 代理收进一个桌面：独立 worktree 隔离，MIT 开源**
11 月 22 日的 GitHub 周榜上，Orca 以 131,943 星、7 天净增 7,179 星排进前十。它把自己定位成代理编排桌面环境：每个任务分到独立 Git worktree，多个 Claude Code、Codex、Grok、Cursor 等 CLI 代理可并行跑，结果回到同一界面审 diff；还支持 SSH 主机、云端 VM、手机端接管，以及从 Slack、Linear、GitHub 直接派活。项目以 MIT 开源，桌面端覆盖 macOS、Windows 与 Linux，手机端可接收代理求助并直接回话；代理的长期记忆写成可读可改的 SOUL.md 与 MEMORY.md。
[原文](https://github.com/stablyai/orca)

**9. Claude-BugHunter 把 681 条公开漏洞模式做成 83 个技能，一周涨 1 万星**
11 月 22 日的周榜里，Claude-BugHunter 以 15,977 星、7 天净增 10,159 星位列第二。它是一套 Claude Code 技能包：83 个技能、15 条斜杠命令、681 条公开漏洞报告模式（其中 433 条逐条标注来源），覆盖 24 类核心漏洞，另附 M365/Entra、Okta、vCenter 等企业边界攻击矩阵。技能采用通用 SKILL.md 格式，可同时装进 Claude Code、Codex CLI、OpenCode、Hermes 与 AntiGravity，描述目标后按主题自动加载；代码 MIT、内容 CC BY 4.0。
[原文](https://github.com/elementalsouls/Claude-BugHunter)

*本期由站点自动生成 · 信息以原文为准。*
