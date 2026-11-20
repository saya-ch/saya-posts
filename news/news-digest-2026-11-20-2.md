---
title: 决策模型只收输入费，35B 扩散模型单卡跑到 680 token/s
date: 2026-11-20T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头从“模型多大”移到“运行时多贵”。Liquid AI 的 d1.2 决策模型只按输入计费，在 AA 决策指数 v2.1 拿到 75 分，校准误差压到 0.8 个百分点；DiffuSpace 的 Clover 用 35B 总参数、3B 激活的扩散架构，在单张 GB300 上报告 680 token/s。研究侧，一篇论文指出自改 harness 必须靠带闸门的模块化更新，另一篇实测发现前沿模型能判对 98% 的新颖性，却只有 8% 的答案把“没人做过”说清楚。工程侧，LangGraph 1.2.24 修掉消息 reducer 的边界，CISA 把 LiteLLM 的自定义护栏端点列入已知利用漏洞，Ollama 与 Together AI 则把前缀缓存的账算到了每一轮。想看清每个数字与升级代价，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Liquid AI 发布 d1.2 决策模型：只按输入计价，决策指数 75 分登顶**
11 月 16 日，Liquid AI 放出 d1.2 与 d1.2-ultra 两档决策模型，输入从纯文本扩到图像与音频，并新增“读出数值并给出来源字符”“找出所有同类片段”“框出目标”“带区间的数值估计”“多选全部适用项”五种答案类型。d1.2-ultra 在 Artificial Analysis 决策指数 v2.1 拿到 75 分，比此前最好成绩高 2 分，d1.2 为 69 分；在 13 个基准、22,505 道题上，d1.2-ultra 的校准误差是 0.8 个百分点，对比 Perplexity Decider v1.3 的 3.7 与 GPT-6 Luna 决策接口的 7.2。计费只算输入，d1.2 每百万 token 0.02 美元、ultra 0.042 美元，不产生输出 token，一张 1024×1024 图片按 1,536 token 计。目前只有托管接口，权重“会在某个时候开放”。
[原文](https://www.liquid.ai/blog/d1-2)

**2. Clover-35B-A3B 用扩散架构做智能体：单张 GB300 跑到 680 token/s**
11 月 18 日的技术报告把 Gated Delta Net 接到块扩散上：已生成的上下文用循环记忆保存，当前块内用双向注意力，两者在同一个 softmax 里竞争，另加一条零初始化的右上下文卷积。35B 总参数只激活 3B，支持思考模式，SWE-bench Verified 73.6、Terminal-Bench 2.1 51.7，在单张 GB300、BF16、batch size 为 1 时报告生成速度 680 token/s。配套的决策变体能在一次去噪里同时回答一组相关问题，内部 500 组样本上单题准确率从 90.37% 升到 93.88%，全组答对率从 65.4% 升到 77.0%；权重、SFT 代码与推理引擎均已放出。
[原文](https://arxiv.org/abs/2611.24636)

**3. 《Harnesses That Improve Themselves》：自改运行时要靠带闸门的模块化更新**
11 月 17 日的论文把智能体 harness 拆成四块可编辑组件——上下文与状态、控制流、动作接口、验证与恢复，再配一条“诊断失败、提出补丁、验证候选、部署更新”的四阶段回路。作者在 SWE-bench Lite 上对比提示反思、整份 harness 重写与带验证闸门的模块化演化三种做法，结论是第三种跨模型、跨仓库都更稳，token 成本只有前两者的零头，因为整份重写很容易让模块之间互相拖累而过拟合。文章还把提示膨胀、刷榜与评估器篡改列为自修改必须设防的三类失败模式。
[原文](https://arxiv.org/abs/2611.22299)

**4. 前沿模型能判对 98% 的新颖性，却只有 8% 的答案说清楚**
11 月 17 日的论文从新晋录用的论文里取了 40 个研究问题，让模型判断“是否已被人做过”。GPT-6 Astra 的判断正确率是 98%，但在普通回答里把“没人做过”明确讲出来的比例只有 8%；六个前沿系统的“判断—表述”缺口在 30 到 90 个百分点之间，能力越强缺口越大。反过来，当论文原文被直接提供时，模型识别“已研究过”的准确率有 88% 到 100%。作者用“先判断、再单独核查所引论文”的方式拼出固定答案，把 Astra 的明确表述率从 8% 拉到 95%，并据此复现了审稿人的新颖性评分区间。
[原文](https://arxiv.org/abs/2611.23546)

## 开源与工程

**5. LangGraph 1.2.24 修消息 reducer 边界，顺带补上一批状态读取开销**
11 月 20 日发布的 langgraph 1.2.24 只做两件事：让 `_messages_delta_reducer` 正确处理 `REMOVE_ALL_MESSAGES`，以及在 `DeltaChannel` 里容忍空 checkpoint。前几个版本的重点在状态读写：1.2.23 让 `Send` 派发的工具调用拿到与普通边一致的状态，1.2.22 则跳过纯文本消息的转换、在读取任务状态前先过滤待处理写入、把更新键的匹配改成集合查找，并把未跟踪通道与 delta 通道的发现收敛到每次运行一次；配套的 checkpoint-postgres 3.2.1 也去掉了连接池上多余的锁。
[原文](https://github.com/langchain-ai/langgraph/releases)

**6. OpenAI Agents SDK 0.25.0：把沙箱工作区做成一等公民**
11 月 18 日上传的 0.25.0 仍标着 pre-1.0，但已经把 `SandboxAgent` 写进核心概念：用 `Manifest` 声明工作区条目，用 `GitRepo` 拉仓库，再用 `UnixLocalSandboxClient` 或 Docker 客户端把文件读写、命令执行与补丁应用关进沙箱。SDK 同时保留 handoffs、guardrails、sessions、tracing 与 realtime 语音智能体，官方口径是通过适配层支持 100 多个模型；语音、Redis、SQLAlchemy、Temporal 等能力拆成 extras，按需安装，避免把一个网关塞进最小依赖。
[原文](https://pypi.org/project/openai-agents/0.25.0/)

**7. CISA 把 LiteLLM 列入已知利用漏洞：自定义护栏端点可执行任意代码**
11 月 18 日，CISA 的 KEV 目录新增 CVE-2026-40217：BerriAI LiteLLM 在 `/guardrails/test_custom_code` 上允许攻击者通过字节码改写执行任意代码，CVSS 3.1 评分 8.8，联邦机构的修复期限是 12 月 2 日，NVD 记录的受影响版本截至 2026-04-08。同批还有 Apache Axis、REDCap 与 Synology DSM 三个漏洞，其中三个的期限是 11 月 21 日；安全媒体把这枚 CVE 与 LLMjacking 活动联系起来，建议不要把代理与管理端点暴露到公网，并在升级后轮换它持有的上游密钥与虚拟密钥。
[原文](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)

**8. Ollama 与 Together AI 算清前缀缓存的账：一百轮是 5,050 比 100**
11 月 16 日的联合长文给出一个可验算的例子：每轮固定新增内容时，十轮对话不做缓存要处理 1 加到 10 共 55 份 prompt，做缓存只需 10 份；一百轮则是 5,050 对 100。缓存从第一个不同的 token 处截断，所以把时间戳、git 状态或工作目录写进系统提示，等于每轮重建整段历史；GLM-5.3-Flash 的聊天模板还会把推理强度写进第一个系统块，中途调高就要重算全部前缀。文章同时给了子智能体该继承历史还是只给简报的取舍，以及引擎侧按前缀哈希路由、把空闲会话的缓存挪到内存、把预填充与解码分到不同集群的做法。
[原文](https://ollama.com/blog/open-models-long-horizon-agents)

**9. NVIDIA Dynamo 收掉 rollout 的重复计算：权重更新快 15 倍**
11 月 19 日的技术博客给出 RL 训练里的三项优化：KV 感知路由在缓存复用与 worker 负载之间取平衡，避免同一提示被反复预填充；ModelExpress 复用集群里已有的权重，只传压缩后的增量，CoreWeave 据此把权重更新提速 15 倍；worker 掉线时把请求导向健康节点，替换节点用 checkpoint 加增量追平。官方给出的生产数据是 H Company 的训练步时间最多缩短 20%、GPU 小时减少 5%，框架侧已接入 verl、NeMo RL、Slime 与 prime-rl，微软 AI 的 post-training 也在用这套 rollout 推理。
[原文](https://developer.nvidia.com/blog/reducing-rollout-bottlenecks-in-reinforcement-learning-with-nvidia-dynamo)

*本期由站点自动生成 · 信息以原文为准。*
