---
title: Xing 4 用 8192 张昇腾训 285B，Phai-A1 把百万上下文塞进个人电脑
date: 2026-11-19T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期的主线是“把规模换成可部署的形态”。Xingchen AGI Lab 的 Xing 4 用 8,192 张昇腾 910C 训出 285B 总参数、每 token 激活 17B 的模型，原生支持 100 万 token 上下文；范式智能的 Phai-A1 同样把百万上下文带到个人电脑，官方称同等单卡下比 Qwen 27B 快 8–10 倍。云知声 U2-Flash-1118 只加 1B 视觉模块，就把 U2-SWE-Bench 做到 57.1 分，Token 消耗反降 37%。研究侧，Ask2Decide 发现信息缺失时前沿模型 58%–98% 的初始计划不可行，而追问出关键事实的比例并不高。工程侧，Dify 1.20.0、LiteLLM 1.110.0 与 n8n 2.48.5 都在补可观测性和路由底座。这些数字拼在一起，能看出“能训”与“能部署、能管住”之间仍有明显缝隙，想看清每条细节，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 范式智能发布 Phai-A1 与 Phai-M1：百万上下文可跑在个人电脑**
11 月 18 日，范式智能发布 Phai-A1 与 Phai-M1。Phai-A1 原生支持约 100 万 token 上下文，覆盖代码生成、缺陷修复、工具调用与多步任务；官方称在同等单卡、相同编码 Agent 任务下，响应速度约为 Qwen 27B 的 8–10 倍，并可在个人电脑本地部署。Phai-M1 面向真人影视，单流程生成画面与同步音频，720p、15 秒视频约 3 分钟。两款模型均针对昇腾平台做端到端后训练优化，权重计划近期开放，而非已经开源。
[原文](https://bizhub.vietnamnews.vn/phancy-launches-phai-a1-language-model-and-phai-m1-video-generation-model-post419004.html)

**2. 云知声 U2-Flash-1118：只加 1B 视觉模块，U2-SWE-Bench 拿 57.1 分**
11 月 18 日，云知声发布 U2-Flash-1118，语言基座参数量不变，仅新增约 10 亿参数视觉模块，原生支持 UI 截图、设计稿、图表与文档版面理解。官方称其在 U2-SWE-Bench v1.1 以 57.1 分排第一，TerminalBench 4.0 得 35.5 分、比上一代提升 12.9 分，GPQA-Diamond 93.9 分；相同任务下总 Token 消耗下降约 37.0%，模型调用次数减少约 44.8%。限时六折价格为每百万 token 输入 0.6 元、输出 1.2 元、缓存命中 0.02 元。
[原文](https://unisound.com/en/research/177)

**3. Xingchen AGI Lab 发布 Xing 4 技术报告：285B 总参数，8,192 张昇腾 910C 预训练**
11 月 17 日提交 arXiv 的 Xing 4 技术报告给出完整训练栈：模型总参数 285B，每个 token 激活 17B，支持 100 万 token 上下文，预训练使用 8,192 张昇腾 910C，软件栈为 MindSpore HyperParallel，结合拓扑感知混合分片与长序列激活管理。后训练侧引入 TeleGym，把执行、评测与轨迹构建接到策略优化，并用异步执行、受限过采样与部分回滚处理长短不一的任务。报告同时给出已发布的 29B 成员，说明这套国产算力栈可以跨参数规模复用。
[原文](https://arxiv.org/abs/2611.22115)

**4. Ask2Decide：信息缺失时，前沿模型的计划 58%–98% 不可行**
11 月 15 日更新的 arXiv 论文提出 Ask2Decide，用 43 个排班、派单类规划任务测试 Agent 是否会主动追问真正影响决策的信息。研究收集 19 名参与者、79 轮交互，并用优化求解器判定哪些事实会改变最优计划。四个前沿模型在信息缺失时，初始计划有 58%–98% 不可行；即使把所有事实一次性给全，这一比例仍有 5%–28%。更关键的是，Agent 追问出“会改变计划”的事实只比“不影响计划”的事实略多（39%–52% 对 32%–41%），说明会问问题不等于问对问题。
[原文](https://arxiv.org/abs/2611.13015v2)

**5. Kernel Reality Check：核化注意力差 softmax 2–10 分，效率优势也难复现**
NeurIPS 2026 接收的《The Kernel Reality Check》系统评测了核化注意力。效率侧，FlashAttention 在多数序列长度下训练耗时与显存都优于当前核化方法，论文还发现不少“加速”结果无法在现代基线上复现；只有因果语言模型推理显存是核化方法少有的明确优势。质量侧，统一蒸馏协议下核化方法与 softmax 仍差 2–10 分，且单纯放大模型规模不能填平。作者提出的 LARA++ 通过自适应提案的重要性采样，最多恢复 99% 的 softmax 准确率，但额外计算开销仍然很大，效率墙没有被真正推倒。
[原文](https://arxiv.org/abs/2611.22947)

## 开源与工程

**6. Dify 1.20.0：Trace 可导到任意 OTel collector，知识库上传先过病毒扫描**
11 月 18 日，Dify 发布 1.20.0。新版本把应用与 Agent 的 trace 通过 OTLP/HTTP 导出到任意兼容 OpenTelemetry 的后端，并适配 Langfuse v4；由于 Langfuse Cloud 已在 11 月 16 日停用旧摄取端点，旧配置必须升级才能继续上报。知识库新增上传文件恶意软件扫描，支持 Cloudmersive、Scanii、OPSWAT 与自定义端点，默认在扫描不可用时阻断文件。此外还有可实时查看 Agent 工具调用的 Chatflow、个人访问令牌与 6 个数据库迁移。
[原文](https://github.com/langgenius/dify/releases/tag/1.20.0)

**7. LiteLLM 1.110.0：Rust 路由核心替换 Python 实现，SpendLogs 成默认账本**
LiteLLM 发布 v1.110.0，自上一版以来累计 840 次提交。最重的改动是把 Python router core 换成 Rust 实现，网关只通过 host 边界调用，同时保留原有路由、回退与预算逻辑；记账侧把 SpendLogs ledger 设为默认路径，并支持按 key、team 关闭 prompt 存储。发布物开始提供 cosign 镜像签名校验，新增 Agent Skills Hub 的上传、版本历史与审核接口，以及模型弃用提醒。对一个常被当作“OpenAI 兼容代理”的项目来说，这意味着它正在往真正的 LLM 网关底座走。
[原文](https://github.com/BerriAI/litellm/releases/tag/v1.110.0)

**8. Gemma 4 的 K=V 缓存只存一份：解码显存压到 1/1.34–1/1.40**
11 月 17 日的 arXiv 论文指出，Gemma 4 31B 的全局注意力层里，value 就是归一化后的 key 投影，key 又是 value 乘固定权重，因此服务引擎没必要同时存两份 1,024 维向量。作者在 vLLM 原型中只存一条 640 维行，用两个重叠的 strided view 分别当 key cache 与 value cache，注意力 kernel 不改。32K–64K token 下，每请求稳态解码 KV 显存降为原来的 1/1.34–1/1.40；双 A100 上 16K prompt 的解码吞吐为原始的 1.38 倍，8K 为 1.24 倍。代价是同等 batch 下 eager step 慢 4%–9%，且暂不能与 8-bit KV 缓存叠加。
[原文](https://arxiv.org/abs/2611.23035)

**9. n8n 2.48.5：AI Agent 节点的工具调用历史按输入项隔离**
11 月 18 日，n8n 发布 2.48.5，重点修 AI Agent 节点：工具调用历史现在按每个输入 item 分开保存，避免批量输入时后一项读到前一项的工具轨迹；流式文本块之间不再被插入额外换行。其余修复包括 Qdrant 向量节点保留反向代理子路径、Microsoft Teams 触发器补齐批量请求的每条变更通知，以及沙箱表达式对小型对象的保留。对把 n8n 当 Agent 编排层的团队，这类小版本比新功能更值得先升。
[原文](https://github.com/n8n-io/n8n/releases/tag/n8n@2.48.5)

*本期由站点自动生成 · 信息以原文为准。*
