---
title: Anthropic 让推理可验证，vLLM 与 SGLang 同周提速
date: 2026-11-05
type: news
tags: [AI, 开源]
status: published
summary: 本期最硬的一条来自 Anthropic：FlashProve 用 zkSNARK 为推理结果出证，400B 的 Llama 4 Maverick 在单张 H100 上做到 110 token/s，单份证明约 19MB，验证只需 CPU。研究侧，斯坦福把 50 条高引 LLM 结论放到 20 个模型上复测，44% 已经过期；CloudOpsBench 用 45 个真实 AWS 任务说明 Agent 仍卡在推理。工程侧，vLLM 0.33 与 SGLang 0.5.23 同周发布，前者一次合入 927 个提交，后者把低延迟解码再推高一截。三条线指向同一个判断：能力之外，可验证性与交付效率正成为新的比较维度。本期按"可验证、可评测、可部署"三条线选材，想逐条核对数字与原文，往下翻即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Anthropic 开源 FlashProve：用 zkSNARK 证明回复出自指定权重**
Anthropic 11 月 3 日公开 FlashProve，把信任从"相信服务商"换成"验证数学证明"：离线阶段先对权重做承诺（单模型约 39 分钟），在线阶段再按批为回复生成零知识证明，一份承诺可覆盖约 1600 万次请求。对 400B 参数的 Llama 4 Maverick（4bit 量化），单张 H100 的证明速度约 110 token/s，1024 token 请求的证明约 18.78MB，24 线程 CPU 验证约 17 秒；代价是证明耗时约为 vLLM 原生推理的 70–95 倍，代码、技术报告与 Lean 形式化证明均以 Apache 2.0 开放。局限也写得直白：证明只覆盖整数模型与贪心解码，采样解码或超过 1024 token 的上下文仍不在保证范围内。
[原文](https://www.anthropic.com/research/provable-inference)

**2. 斯坦福复测 50 条 LLM 结论：44% 已经过期**
斯坦福团队把 50 条高引 LLM 结论放到 2024 年 7 月至 2026 年 9 月发布的 20 个模型上重测，44% 已失效、34% 明显减弱，仅 16% 仍成立；22 条有明确期限的结论平均寿命只有 15.7 个月。"模型做不到"类判断老化最快，9 条推理结论中 5 条过期；引用幻觉从 GPT-3.5 的 55% 降到 GPT-6.1 Sol 的 0%，但 Qwen3.7-Flash 仍高达 69%。同一代模型之间，过期条目数从 11 条到 30 条不等，研究团队因此主张论文必须标注模型、版本、日期与提示词，方便后来者复测。失效率还随时间抬升：一年后 16%，两年后升至 39%。
[原文](https://the-decoder.com/rapid-llm-development-is-outrunning-the-research-meant-to-explain-it-study-finds/)

**3. CloudOpsBench 开源：45 个真实 AWS 任务，最高通过率 60.6%**
密歇根大学等团队发布 CloudOpsBench v1.0，45 个任务覆盖资源下线、迁移、配置变更与故障修复，同一套任务可在真实 AWS 和仿真器中重放，代码以 Apache 2.0 开放。榜首 Claude Fable 5.1 也只完成 60.6%，完整评测耗时 38.6 小时、花费 404.3 美元；失败分析显示约 80% 源于任务范围误判、漏查相关服务或过早收工，只有约 20% 是执行了错误动作。项目同时给出失败分类与参考解法，每个任务都带种子状态、评分器与参考解，后续计划把评测扩展到 GCP、Azure 与 OCI。
[原文](https://github.com/CloudOpsBench/CloudOpsBench)

**4. Anthropic 上线 Message Threads：长对话首 token 快约 35%**
Anthropic 11 月 2 日把 Message Threads 推上公测：过去每一轮都要重发全部历史，几十万 token 后延迟被迅速放大；新机制把会话状态留在服务端，后续请求只发送新增消息，50 万至 100 万 token 的请求首 token 时间中位数下降约 35%。线程在内存中加密保存、闲置 5 分钟过期，token 计费不变，已覆盖 Claude 平台以及 AWS、Microsoft Foundry、Google Cloud 三个云入口，对编码与 computer-use 类长任务收益最明显，也意味着 Agent 的多轮工具调用不必再反复传输历史上下文。
[原文](https://claude.com/resources/articles/message-threads)

## 开源与工程

**5. vLLM 0.33.0 发布：927 个提交，CUDA 12.9 正式退场**
vLLM 11 月 2 日发布 0.33.0，含 927 个提交、389 名贡献者（其中 122 名新人），升级 PyTorch 2.14，并只发布 CUDA 13.0 轮子，CUDA 12.9 镜像与 `-cu129` 标签停止更新。新版本为 Reflection Beam-1.0、Step-5 Preview 等模型提供 day-0 支持；DFlash/DSpark 不再预留草稿槽位，官方基准中 GB300 上 DeepSeek-V4.1-Flash 的输出从 1373 提升到 3390 token/s。破坏性变更还包括最低 Python 升到 3.11、移除 Medusa 与 MLP speculator，升级前需要逐项对照；镜像侧则新增 AWS EFA 变体，方便多机 KV 传输。
[原文](https://github.com/vllm-project/vllm/releases/tag/v0.33.0rc3)

**6. SGLang 0.5.23：1007 个 PR，低延迟解码最高快 75%**
SGLang 11 月 2 日发布 0.5.23，合入 1007 个 PR、来自 257 名贡献者，新增 Muse Spark Open、Beam-1.0、MiniMax-M3.1-Flash 等模型支持。低延迟模式用融合持久化内核执行 DeepSeek-V4.1-Flash 解码，GB300 单用户最高快 44%、MI350X 最高快 75%；MoE 模型在 H200 的 8K 提示上首 token 提前 30%–58%，分页专家机制则让超出显存的 MoE 比 CPU offload 快至多 30 倍。该版本还把 DeepSeek-V4.1 在 GB300 的预填充吞吐提高至多 1.5 倍，并新增 Google TPU7x 支持，AMD ROCm 10.0 与 CUDA 13 镜像同步更新。
[原文](https://github.com/sgl-project/sglang/releases/tag/v0.5.23)

**7. Claude Code 2.1.317：给 Agent SDK 钩子加"失败即阻断"**
Claude Code 2.1.317 把安全与成本控制继续前移：新增联网或调用 shell 前确认账号邮箱的检查，`onFailure: "block"` 让 Agent SDK 钩子失败时直接阻断动作，并开放按开发者、按天、按 IdP 组读取历史花费的成本报表 API。该版本还修复了 MCP 服务器首轮后断连、云会话权限模式回退、子代理模型继承错误等一批长尾问题。对把 Claude Code 接进 CI 或自建网关的团队，这些开关直接决定权限边界与账单可见性。
[原文](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

**8. Ollama 0.40.14：修好 OpenAI 兼容层的采样默认值**
Ollama 11 月 3 日发布 0.40.14，重点修兼容性：OpenAI 兼容端点不再把未传 `temperature`/`top_p` 的请求强设为 1.0，改为沿用模型自带的采样参数；`ollama launch claude` 会应用模型的输出上限，避免云模型撞上 128k 默认帽。另修复云搜索压缩响应的解码、`/v1/responses` 的输入保真，并加快 MLX 后端大词表上的 top-k 采样。这类小版本看似琐碎，却决定了同一客户端在本地与云端之间切换时能否拿到一致的输出。
[原文](https://github.com/ollama/ollama/releases/tag/v0.40.14)

*本期由站点自动生成 · 信息以原文为准。*
