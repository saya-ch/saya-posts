---
title: 推理栈的静默错误：vLLM 越界崩溃，SGLang 把 KV 缩放丢成 1.0
date: 2026-11-08T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头对准推理栈里那些不报错的错误。英伟达 11 月 3 日的论文实测，前沿开放权重模型生成后段 token 的成本可达前段的 7 倍，而频繁压缩上下文几乎不掉精度；vLLM 0.32 在 262,144 token 边界触发 CUDA 非法访存，官方只能建议先钉回 0.31.1；SGLang 则把 Qwen3-MoE 等三类模型的 FP8 KV 缩放静默丢成 1.0，输出依旧有限、数值却已失真。再往下，LiteLLM 的 MCP 失败标记、Ollama 的 Vulkan 加载崩溃、Unsloth 的逐文档嵌入，都是同一类“看起来正常”的坑。想看清每个数字与复现条件，展开正文逐条核对。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 英伟达论文：后段 token 贵出 7 倍，长上下文并非刚需**
11 月 3 日提交的 31 页技术报告由 Peter Belcak 等 12 位作者完成，实测前沿开放权重模型生成越靠后的 token 成本越高，差距最高可达 7 倍。作者在大量长程与长输入任务上用频繁的上下文压缩（compaction）把窗口维持在很短的长度，任务精度几乎不掉；即便每个上下文只允许执行一步动作，长程软件工程任务也没有崩，据此提出“语义状态空间模型”（SSSM）的方向。对工程团队的启示是，与其为百万级窗口付溢价，不如把状态管理、压缩时机和每步动作的粒度做对。
[原文](https://arxiv.org/abs/2611.05306)

**2. Kakao 开源 3.67B 决策模型：0.04 秒给出概率，比 GPT-6 Luna 快 43 倍**
Kakao 11 月 6 日在 Hugging Face 放出 Kanana-2-v-decide，基于自家 Kanana-2-v-3b，视觉编码器 0.49B、语言模型 3.18B，共 3.67B 参数，支持文本、JSON 与对话输入，最多 10 张图、20,480 token。它不生成句子，而是用嵌入相似度加 softmax 直接返回 choice、noul、score 三类概率分布；单张 H200 上官方称平均响应低于 0.04 秒，比 GPT-6 Luna 快约 43 倍。自建的 15,909 道韩语 Kanana Decision Bench 得 72.50，公开 Decision Index 61.34、视觉版 61.12，均值 61.23，采用允许商用的 Kanana Open License；需要留意这些数字均为厂商自测，且头号基准偏韩语。
[原文](https://huggingface.co/kakaocorp/kanana-2-v-decide-3b-2611)

**3. MEMOIR：把“用过再评分”写进记忆生命周期，在线写入零 LLM 调用**
11 月 5 日提交的论文提出一套免训练的记忆生命周期：压缩前先由 judge gate 检验答案证据是否被保留，查询后再按归因结果更新被检索记忆的优先级，并用 UCB 式探索 bonus 防止高频记忆挤掉冷门证据。在 1,905 条去重后的 LoCoMo 问答上，完整检索栈把 evidence recall@5 比裸 cosine 提高 11.92 个百分点，置信门控反馈在不使用参考答案、不训练重排器的情况下再加 1.50 点；100 次独立审计的合并没有出现证据丢失，单侧 95% 损失率上界为 2.95%，活动存储还缩小了 1.42%。
[原文](https://arxiv.org/abs/2611.07631)

## 开源与工程

**4. vLLM 0.32 在 262,144 token 边界崩溃，维护者只能建议先钉回 0.31.1**
Issue #62383 记录了一次生产事故：8 张 H200、TP=2 加专家并行，Qwen3.8-Flash-Next 的 NVFP4 检查点处理 262,397 token 的单个请求时，分块预填直接触发 CUDA illegal memory access，EngineCore 死亡、/health 返回 503，重启约 4 分钟，三个实例在真实长文档流量下接连崩溃。同一台机器、同一组参数，0.31.1 在 24.2 秒内完成并在大海捞针中召回密钥；0.32.0 与 nightly 则稳定复现，边界正好落在原生 262,144 之外的 YaRN 区。由于 1M 上下文是产品硬需求，无法退回 262k，团队实际只能整体回滚。同期 DeepSeek-V4.1 的 block-size 处理在 0.32.0 上也被报出异常（#59203），升级前先看 issue 比看 release note 更可靠。
[原文](https://github.com/vllm-project/vllm/issues/62383)

**5. SGLang 静默丢 FP8 KV 缩放：96 个模型文件里 74 个没接重映射**
Issue #39942 拆开了一条完全无声的精度损失链：Qwen3-MoE、Qwen2 与 Mixtral 的权重加载器在处理融合投影时，把 `v_proj` 子串替换进 `qkv_proj`，得到并不存在的 `qkqkv_proj.k_scale`，查不到就跳过；随后 `process_weights_after_loading` 看到两个缩放仍是初始的 -1.0，便统一回落到 1.0。于是每层 FP8 KV 缓存都按单位缩放反量化，误差正好等于原本标定的缩放值，输出依然有限、也不会抛异常，张量级检查完全看不出来。作者用 3 个模型、3 种命名、3 种变体共 27 组组合复现，只有 Qwen3-MoE 路径连一条警告都不打；全仓 96 个使用 QKVParallelLinear 的模型文件里只有 22 个调用了现成的重映射助手，修复 PR 已按 vLLM 的正则方案补上融合命名分支。
[原文](https://github.com/sgl-project/sglang/issues/39942)

**6. LiteLLM 1.108.1 转入 LTS：镜像全量 cosign 签名，验签要按提交哈希**
网关层这周的重点是供应链而非新功能：1.108.1 作为 LTS 发布，所有 Docker 镜像都用 commit e82a3c6 引入的密钥做 cosign 签名，官方建议用不可变的提交哈希拉取公钥来验签，而不是依赖可被移动的 tag；该版本从 main 回移了 21 个修复与 2 个前置补丁。签名解决的是“镜像有没有被换”，却解决不了“语义对不对”：同期仍开放的 #49359 显示，MCP 工具执行失败时 `is_error` 标志在转成 Anthropic tool_result 时被丢掉，模型会把“Database connection refused”当成一次成功的工具返回，从而不再重试或纠错。对做 Agent 的团队，这意味着既要按摘要锁镜像，也要在自己的工具层重新判定失败。
[原文](https://github.com/BerriAI/litellm/releases/tag/v1.108.1)

**7. Ollama 0.40.15-rc2 在 Renoir 核显上加载即崩，380MB 小模型也不放过**
Issue #19558 报告，Ryzen 7 4700U 的 Radeon 核显（gfx90c）在 Windows 10 上启用 Vulkan 后端后，`load_tensors` 阶段进程直接以 0xe06d7363 退出，0.40.14 稳定版与 0.40.15-rc2 都能复现，qwen2.5:0.5b（374 MiB）和 qwen2.5-coder:7b 一视同仁；直接启动随包的 llama-server 同样崩，说明不是调度器的问题。纯 CPU 路径正常，生成约 49 tok/s、prompt 约 95 tok/s，但 ROCm 回退也不可行——随包的 rocm_v7_1 只带 gfx1030 以上内核。对把本地推理塞进旧笔记本或迷你主机的用户，这一版应当直接跳过，并显式关掉 iGPU 卸载。
[原文](https://github.com/ollama/ollama/issues/19558)

**8. Unsloth 的 RAG 入库一次只嵌一篇文档：99ms/块 对 2.07ms/块**
Issue #16715 指出，RAG 摄取按文档逐个调用 `_embed_all`，批处理只发生在单篇文档内部，于是笔记、文档、wiki 这类小文件语料全程跑在最慢的单条路径上。实测同一批文本，CPU 单条为 99.0 ms/块（约 10 块/秒），CPU 32 条批处理降到 11.7 ms/块（86 块/秒），GPU 32 条批处理为 2.07 ms/块（482 块/秒），相差约 48 倍。一个 676 个 Markdown、2.6 MB、约 1,786 块的 Obsidian 库，默认设置要 25 分钟以上才能建完索引，而跨文档攒批后仅嵌入环节在 CPU 上就只需约 0.4 分钟。问题不在模型大小，而在调用粒度。
[原文](https://github.com/unslothai/unsloth/issues/16715)

*本期由站点自动生成 · 信息以原文为准。*
