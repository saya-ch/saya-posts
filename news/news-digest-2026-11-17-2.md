---
title: Inkling 2 Small 放开 594B 权重，vLLM 0.34 按实测内存定 KV 缓存
date: 2026-11-17T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这期的主线是“把实验室能力落到可部署的形态”。Thinking Machines 把 594B 总参数、17B 激活的 Inkling 2 Small 以 Apache 2.0 放开，1M 上下文与视觉输入一并开放；蚂蚁 inclusionAI 的 ZoomGain-9B 用 9,000 条 RL 样本，把 13 项未参与训练的基准平均分从 59.75 拉到 66.10；NVIDIA 则把 Nemotron 3.5 Super VL 与近 5 万道化学题一起开放。工程侧，vLLM 0.34 改为按预热后的真实空闲显存分配 KV 缓存，H200 上 MiniMax-M3 的缓存从 433k 涨到 672k token；Transformers v5 只留 PyTorch，Qwen Code 1.0 把 Qoder 的 harness 并进开源仓库。想看清每个版本的数字、代价与升级坑，展开正文即可。这些变化共同说明：能力、成本与升级代价正在被同时计价。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Inkling 2 Small 放开权重：594B 总参数、17B 激活，Apache 2.0**
Thinking Machines 于 11 月 12 日把 Inkling 2 Small 的完整权重放上 Hugging Face，594B 总参数、17B 激活的 MoE 同时接受图像输入并支持 1M token 上下文；官方给出的 Artificial Analysis 智能指数为 31，SWE-bench Pro v2 的 Hard/Full 成绩为 86.3/97.7%，MMMU Pro 为 81.6%。模型用 292M 参数的视觉编码器切 20×20 像素 patch，预训练在 GB300 NVL72 上以原生 NVFP4 跑了 8.4 万亿 token、耗时 68 小时，权重以 Apache 2.0 开放，vLLM、SGLang、Unsloth 的适配与 NVIDIA、Fireworks、Together 等多家 API 同日到位；官方另给出 DeepSWE v1.1 72.1%、DesignArena 1377 分，定位是开源阵营里最靠前的通用 agent 模型之一。
[原文](https://thinkingmachines.ai/news/inkling-2-small/)

**2. ZoomGain-9B 悄悄上线：9,000 条 RL 样本换来 6.35 分平均提升**
蚂蚁 inclusionAI 在 11 月 10 日上传 ZoomGain-9B 与 4B 两个视觉语言模型，代码以 Apache 2.0 开放，训练数据与评测集一并放出，训练集由 7,713 张源图挖出 9,000 条“全局看得到、放大才看清”的细粒度问答，用 GRPO 训练一个 epoch；9B 在 13 项未参与训练的基准上平均 66.10，高于基座 Qwen3.5-9B 的 59.75，高分辨率基准 HR-Bench 4K 从 82.13 升至 90.25。两个 BF16 检查点分别为 18.82GB 与 9.08GB，次日补上 Q4_K_M 等三档 GGUF，最小组合约 6.15GB，可在消费级显卡或笔记本上跑；但检查点仓库本身尚未打许可证标签，商用前需自行核对。论文把 55.48 的榜内成绩标为 Held-in，提醒读者别把训练分布内的分数当成泛化能力。
[原文](https://arxiv.org/abs/2611.14076)

**3. NVIDIA 推出 Nemotron for Science：近 5 万道化学题随模型开放**
NVIDIA 在 11 月 16 日的 SC26 上发布 Nemotron for Science，把开放的 Nemotron 3.5 Super VL、约 5 万条化学推理问答集 LitmusBench v0.2 与后训练配方打包，模型可在 128GB 的 DGX Spark 上本地运行；数据集覆盖分子性质与药物发现实验数据，用于训练和评测模型的化学推理。Terray 用它抽取反应式，在 RxnScribe 上恢复约 76% 的标注反应，Lila Sciences 把量子点合成准确率提到 61%，超过 GPT-6 Astra 的 54%，推理成本低 10–20 倍；Argonne 已把它接入 StructBioReasoner，KTH 的 agent 在一夜之间修好 81% 的 GROMACS 测试用例。NVIDIA 称这套组合面向生命科学与化学，目标是让研究者用自有数据后训练出领域 agent，而不是从通用聊天模型重新起步。
[原文](https://investor.nvidia.com/news/press-release-details/2026/NVIDIA-Introduces-Nemotron-for-Science-to-Accelerate-Discoveries-With-Agents/default.aspx)

## 开源与工程

**4. Transformers v5 发布首个 RC：只留 PyTorch，量化升为一等公民**
Hugging Face 于 11 月 12 日发布 v5.0.0rc-0，正式把 TensorFlow 与 Flax 移出主路径，tokenizer 只保留 tokenizers 后端，不再区分 Fast 与 Slow；新版权重加载把量化做成核心能力，并加入连续批处理、分页注意力与兼容 OpenAI 接口的 transformers serve。官方数字是每天 300 万次 pip 安装、累计 12 亿次安装、400 多个模型架构与 75 万个 Hub 检查点；GGUF 可直接读入微调、模型也能导出给 llama.cpp，vLLM 与 SGLang 则把 Transformers 当作模型定义源，升级前需预留一轮兼容性排查。这次 RC 也把模型定义收敛为模块化写法，减少新架构接入时要审的代码量。
[原文](https://github.com/huggingface/blog/blob/main/transformers-v5.md)

**5. Qwen Code 1.0 预览开源：把 Qoder 的 agent harness 并进终端**
阿里在 11 月 11 日推出 Qwen Code 1.0 Preview，以 Apache-2.0 在 QwenLM/qwen-code 的独立分支发布 v1.0.0-preview.1，原有 0.27 稳定线保留；核心变化是把 Qoder 产品里的 harness 抽成可嵌入的 agent 内核，事件总线从单进程内存模型扩展到多节点，推理编排与工具执行分离，MCP 与沙箱资源集中管理。官方列出四项收益：推理、记忆、工具等处理单元可围绕事件插件化，模型访问收敛到单一入口并集中记录授权与调用，沙箱在同一权限边界内复用，MCP 配置与连接统一管理；但尚未公布基准与稳定版日期，预览标签也意味着接口仍可能变。对想自建 coding agent 的团队，价值主要在 harness 与沙箱管理，而不是又一个新的终端界面。
[原文](https://pandaily.com/alibaba-qwen-code-1-0-preview-qoder-harness-open-source-agent-core)

**6. Agent Framework Python 1.26：MCP 目录上限 100 页、1,000 项**
微软 11 月 12 日发布 agent-framework 1.26.0，延续 1.25 的两项破坏性调整：安装门槛升到 Python 3.11，MCP 工具与提示发现默认最多 100 页、每个目录 1,000 项，框架自有请求超时默认 30 秒；1.26 新增 loop_return_final_only、Hyperlight 沙箱 30 秒执行预算，GitHub Copilot 集成要求 CLI 至少 1.0.85 并支持 Pydantic 结构化输出。发布说明还提到 FIDES 重放溯源、审批消息分组与 Redis 历史裁剪的修复；旧的内存上下文 API 已标记弃用，将在 2026 年 12 月 31 日后移除，迁移需改用 FileMemoryProvider 与 AgentFileStore。这些默认值把“一次拉取上万条 MCP 工具”的部署挡在门外，升级前要显式调大上限。
[原文](https://github.com/microsoft/agent-framework/blob/main/python/CHANGELOG.md)

**7. vLLM 0.34 删除旧 Runner：KV 缓存按实测显存分配，解码最高快 1.95 倍**
vLLM 0.34.0 合入 1,061 个提交、412 位贡献者（其中 120 位新面孔），新增 Mistral Large 4 等 10 个模型，其中 Mistral Large 4 为 1T 总参数、52B 激活并支持 FP8 与 NVFP4；可扩展 KV 缓存转为默认，按预热与 CUDA graph 捕获后的真实空闲显存定量，H200 上 MiniMax-M3 的缓存从 433k 增至 672k token，代价是部分模型在并发 128 时 TPOT 上升 6%–18%。同时 Model Runner V1 被移除、默认 block size 改为 64，Step5 在 4×GB200 上解码最高提升 1.95 倍，GLM-5.3-Flash 吞吐提升 1.6 倍；ROCm 只剩 10.0 构建，huggingface_hub 需 2.0.0 以上。缓存变大并不等于端到端一定更快，官方把选择权交回给 gpu-memory-utilization 与显式关闭开关。
[原文](https://github.com/vllm-project/vllm/releases/tag/v0.34.0)

**8. anthropics/skills 周增 6,077 星：Agent Skills 用 SKILL.md 定规范**
Anthropic 的 skills 仓库在 11 月 16 日更新 docx、pptx、xlsx 三项文档技能，按 GitHub 热榜统计其时星标为 195,376，本周净增 6,077；每个技能就是一个含 SKILL.md 的目录，YAML 只要求 name 与 description 两个字段，规范与模板单独放在 spec、template 目录。仓库里的文档技能正是 Claude 文档能力的生产实现，以 Apache 2.0 开放；它还能作为 Claude Code 插件市场源，用 /plugin marketplace add anthropics/skills 安装，适合当作团队自定义技能与验收脚本的写法参考。对技能作者来说，真正的门槛不在格式，而在把验收标准写成可执行的脚本与资源。
[原文](https://github.com/anthropics/skills)

*本期由站点自动生成 · 信息以原文为准。*
