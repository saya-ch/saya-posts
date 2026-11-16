---
title: 奔驰产线跑出 96% 成功率，决策模型有了统一接口
date: 2026-11-17T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这期的主线是“把模型能力固化成可复用的接口”。Google DeepMind 的 Gemini Robotics 2.5 在奔驰匈牙利与德国产线完成真实试点，Apollo 2 自主分拣零部件的成功率达到 96%，200 项任务上的成功率比上一代高 30%–100%。研究侧，GDP.xlsx 用 70 个真实工作簿和 791 条评分标准测出最强模型严格通过率仅 38.3%，DOCKET 则发现 LLM 生成的立法意见只有 18%–30% 与真人重合。工程侧，Hugging Face 把“决策模型”收敛成 choice、score、noul 三种原语，MiniMax 以开放权重把 4K 图像生成压进 10 秒。这些研究共同说明：榜单分数之外，任务形态与内部处理才决定模型是否真的可用。想看清每条的数字与边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Gemini Robotics 2.5 进奔驰产线：Apollo 2 自主分拣，试点成功率 96%**
11 月 10 日，Google DeepMind 发布新一代 VLA 模型 Gemini Robotics 2.5，在 200 项涵盖工具使用、可变形物体与精细装配的任务上，成功率比上一代高 30%–100%，完成速度快 20%–30%。与 Apptronik、梅赛德斯-奔驰在匈牙利凯奇凯梅特和德国辛德尔芬根的产线试点中，Apollo 2 自主完成汽车零部件 kitting，成功率达 96%；推理时安全引导让障碍物旁与有人环境下的安全完成率分别提升 33% 和 43%。同步开源的 ASIMOV-Jailbreak 含 100 个场景、1,500 条伪装提示，Gemini 4 Argon 的防御率为 99.5%，上一代 ER 2 为 82%。该模型目前向 DeepMind 的机器人合作伙伴分批开放，尚未公布通用 API 价格。
[原文](https://blog.google/intl/ko-kr/company-news/technology/gemini-robotics-2-5-kr/)

**2. GDP.xlsx：70 个真实工作簿、791 条标准，最强模型严格通过率 38.3%**
11 月 13 日提交 arXiv 的 GDP_xlsx 用 70 个由 13 个行业专业人士编写的真实职场请求，配套含 218 万个填充单元格的工作簿与 791 条二元评分标准，要求 agent 在容器内用表格工具完成任务。20 个模型配置中，Gemini 4 Argon 的严格 pass@1 最高，仅 38.3%，中位数 19.3%；完整性与排他性标准要求答案“找全且不多给”。评测框架与 3 个公开任务已放出，其余 67 个留作隐藏集以防污染。
[原文](https://arxiv.org/abs/2611.17766)

**3. DOCKET：4,230 万条立法意见，LLM 反馈仅 18%–30% 与真人重合**
11 月 13 日的 DOCKET 汇集 2016–2026 年间 44 个联邦机构的 22,400 条法规、4,230 万条公众评论，首次系统评测 LLM 在美国规则制定中的多元观点表达能力。结果显示，模型生成意见只有 18%–30% 与真实人类评论对齐；它们倾向泛泛批评，缺少具体引证与亲历论证，且系统性认同政府立场（最高重合 68%），对工会意见重合最低仅 26%，人格化提示也无法消除偏斜。
[原文](https://arxiv.org/abs/2611.17914)

**4. 16 个模型、24 个基准：把“分数”拆成检索与推理两种内部过程**
11 月 12 日的《Tracing Knowledge and Reasoning in LLM》用注意力、激活与表征三类信号，对 16 个 LLM 在 24 个基准上的内部处理做任务级刻画，区分“检索主导”与“推理主导”两类任务，并暴露被总分掩盖的失败模式。论文已被 TACL 接收，代码以 trace-kr 开源；它的意义在于把选型问题从“谁分高”推进到“这个分数由哪种处理支撑”，为基准选择与模型匹配提供可检验的依据。作者也提醒，内部信号与任务分数并非一一对应，跨模型比较仍需控制提示与解码配置。
[原文](https://arxiv.org/abs/2611.17461)

**5. 扩散 LLM 长上下文免训练外推：8K 困惑度从 19.51 降到 3.12**
11 月 11 日的 MPPA 指出，扩散语言模型沿用自回归的静态 RoPE 缩放，忽略了部分解码序列不断变化的掩码模式。作者把索引线性插值回预训练窗口，再按累计掩码密度加入逐位置、逐步的偏移，且偏移随 token 落定而收缩。在 LLaDA-8B-Instruct 两倍窗口（8,192 token）上，困惑度从标准解码的 19.51 降到 3.12，低于窗口内水平，passkey 检索 100%，needle-in-a-haystack 精度不降；目前仅验证 2 倍外推。
[原文](https://arxiv.org/abs/2611.15029)

## 开源与工程

**6. MiniMax H3-Image 开放权重：10 秒内出 4096×4096，支持 16 张参考图**
11 月 10 日，MiniMax 以开放权重发布 H3-Image，支持文生图、参考图生成与图像编辑，可在 10 秒内生成 4096×4096（约 16MP）图像，速度最高为主流方案的 5 倍，并可理解最多 16 张参考图。Standard 模式比主流快 1.5–2 倍，Highspeed 快 3–4 倍，定价为同类产品的 50%–70%；发布周 4K 及以上请求五折。权重已上传 Hugging Face，官方称其提示遵循度在开放权重模型中领先；提速则归因于数据、强化学习、采样步数压缩与部署优化的组合，而非单一改动。
[原文](https://www.minimax.io/blog/minimax-h3-image)

**7. Hugging Face 把“决策”做成模型原语：choice、score、noul 三种输出**
11 月 10 日，Hugging Face 发布 Decision Models 支持，在 Transformers 中加入 DecisionPipeline、AutoModelForDecision 与 DecisionConfig，并统一到 systemone 端点。模型不再逐 token 生成文本，而是对 state 上的命名问题直接给出概率：choice 返回选项分布，score 返回有序档位的期望值，noul 返回伯努利概率，同时给出归一化 confidence。该接口也兼容普通自回归模型，llama.cpp 已实现同一端点，Hub 上线 decision-model 标签，TRL 的训练支持仍在 PR 中。
[原文](https://huggingface.co/blog/decision)

**8. NVIDIA 把 OpenShell 捐给 CNCF：给 agent 的工具与凭证访问加确定性闸门**
11 月 10 日，NVIDIA 将 agent 治理项目 OpenShell 捐入 CNCF Sandbox，用确定性策略约束 agent 对工具、推理服务、凭证存储与监控系统的访问；官方称已有数百家组织使用，不到一年收到数万条贡献。同步推进的 Agent Substrate 由 Google 主导，在任务间保留 agent 环境、空闲时释放算力。配套的 NVIDIA Snapshot 把 Crusoe 上 GLM-5.3 的就绪时间从约 8 分钟压到 31 秒，Saturn Cloud 的 vLLM 冷启动从 2–8.5 分钟降到 6–34 秒。
[原文](https://itbrief.in/story/nvidia-donates-openshell-to-cncf-as-it-backs-agent-scale)

**9. 华为云开源 CloudRobo-WM1.2：双榜第一的昇腾原生具身世界模型**
11 月 16 日，华为云发布并开源昇腾原生具身世界模型 CloudRobo-WM1.2，采用共享世界模型骨干与原生 DualAct 架构，把世界表征与动作生成耦合进同一模型，并给出 World/Expert Action、Joint/Action-only 四种可组合推理模式。官方称其在 RoboTwin 2.0 与 LiberoPro 2.0 两个评测中均位列第一，同时以多粒度监督数据自适应预训练和本体语义保真建模处理跨本体泛化；模型与工具链依托昇腾云开放，目标是覆盖仿真冷启动、云原生训练到真机部署的链路。
[原文](https://www.stdaily.com/web/gdxw/2026-11/16/content_614037.html)

*本期由站点自动生成 · 信息以原文为准。*
