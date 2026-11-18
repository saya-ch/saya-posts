---
title: 腾讯 Hy4 flash 总参 224B 只激活 12B，Cerebras 拆开预填充与解码
date: 2026-11-18T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头从模型发布移到“跑得动、管得住”。腾讯 Hy4 flash 以 224B 总参数、12B 激活把每百万 token 输入价压到 0.5 元，单任务成本只有 Hy4 的二十分之一，而 Cerebras 把预填充交给 AMD GPU、解码留给 CS-3，在 GPT-OSS-120B 生产端点上把每兆瓦吞吐提高 5 倍。研究侧，多智能体记忆实验显示委派让冲突证据的召回从 50.2% 升到 68.7%，冲突检出率却从 16.2% 降到 13.2%；本地医疗智能体用“选择题”把工具选择准确率做到 97.0%。工程侧还有 Open WebUI 的本地沙箱、OpenHands 1.41 的验收场景，以及 agent-framework 供应链投毒事件。想看清每条的数字与升级坑，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 腾讯 Hy4 flash 发布：224B 总参数只激活 12B，单任务成本压到 Hy4 的二十分之一**
11 月 16 日，腾讯混元发布 Hy4 flash，224B 总参数中含 64B Engram 参数，激活仅 12B，支持多模态与 1M 上下文；每百万 token 输入 0.5 元、输出 2 元，缓存命中 0.02 元，解码速度超过 150 tps。官方称其 KV cache 只有 Hy3 的 1/24、Hy4 的 1/6，内部 WorkIndex 上单任务成本仅为 Hy4 preview 的 1/13、Hy4 的 1/20，综合效果还略优于 770B 总参数、49B 激活的 Hy4 preview。
[原文](https://www.tencent.com/zh-cn/tencent-launches-hy4-flash-to-power-high-speed-cost-efficient-ai-deployment-for-everyday-workflows/)

**2. 多智能体委派反而更容易漏掉冲突：召回从 50.2% 升到 68.7%，检出率却降到 13.2%**
11 月 17 日提交的 arXiv 论文在 Harvey LAB 文档审查任务上做了受控事实冲突实验，汇总三个生成预算相近的模型后发现，要求委派让“同时检索到两条冲突主张”的比例从 50.2% 升到 68.7%，但冲突检出率从 16.2% 降到 13.2%；worker 只在 35.2% 的已检索冲突对上同时上报两条。作者指出，当两条主张落在同一个 worker 手里时检出更高，拆到两个 worker 再汇总时反而更低，worker 到 lead 的交接是主要改善点；问题不在单个 worker 的检索能力，而在证据被拆散后，lead 拿不到完整上下文。
[原文](https://arxiv.org/abs/2611.22245)

**3. 本地医疗智能体把工具调用改成选择题：1,952 次请求选对 97.0%，零编造内容**
11 月 16 日提交的 arXiv 论文提出 System 1 agents：用确定性编译器把工具 schema 和请求转成多选题，再由单次决策模型作答，不生成自由文本。在 157 个生物医学工具、1,952 次请求上，最佳 System 1 agent 选对工具或正确拒答的比例为 97.0%，高于 18 组本地 LLM agent 设置中最好的 95.2%；它还填对了 6,056 个声明参数值中的 97.0%，且没有一次调用出现编造内容，对照组的 LLM 调用有 5.5%–19.7% 含编造。
[原文](https://arxiv.org/abs/2611.21053)

**4. 扩散语言模型白拿 2 倍上下文：8K 困惑度从 19.51 降到 3.12**
11 月 11 日提交的 arXiv 论文提出 MPPA，不训练、不改权重，只按掩码密度给每个位置、每个去噪步加一个偏移，把 LLaDA-8B-Instruct 从 4,096 token 窗口外推到 8,192。标准解码在 8K 上困惑度 19.51，MPPA 降到 3.12，甚至低于模型窗内水平，同时保持 100% passkey 检索和不变的 needle-in-a-haystack 准确率。作者认为，扩散 LLM 的位置外推不能照抄自回归那套静态 NTK 缩放，掩码模式本身就是可用信号。
[原文](https://arxiv.org/abs/2611.15029)

## 开源与工程

**5. Cerebras 把预填充交给 AMD GPU：每兆瓦吞吐做到 5 倍，解码速度不降**
11 月 17 日，Cerebras 宣布分离式推理进入生产：AMD Instinct GPU 处理 prompt 预填充，CS-3 晶圆级引擎只负责逐 token 解码。公司在 11 月 14 日的共享 GPT-OSS-120B 端点上测得，相比纯 CS-3 配置，每兆瓦吞吐提升到 5 倍，token 生成速度没有损失；官方给出的原因是单芯片数十 GB SRAM 与数十 PB/s 带宽，分别是典型 GB300 的 350 倍和 5,400 倍。预填充是计算密集型、解码是带宽密集型，分开调度让两类硬件各做擅长的事，代价是多了一次 KV 状态传输。Cerebras 计划到 2027 年底把该方案铺到数百兆瓦的自有与伙伴机房。
[原文](https://www.cerebras.ai/blog/cerebras-launches-disaggregated-inference)

**6. Open WebUI 0.13.4 把本地电脑接进聊天：沙箱只开一个文件夹，MCP 走 stdio**
11 月 17 日发布的 Open WebUI 0.13.4 用 Tether 取代 Direct Connections：Ollama、LM Studio、任意 OpenAI 兼容服务、MCP 工具服务器和 Open Terminal 都能只对当前用户可见，地址、命令与密钥留在本机。Tether 0.1.13 起可在 macOS 与受支持 Windows 上把终端锁进用户选定的单个文件夹，默认离线、不接收入站连接，且不需要 Docker、虚拟机或管理员权限；沙箱无法强制时终端直接停住，而不是降级裸跑。该版本还允许通过 Tether 0.1.16 以 stdio 启动本地 MCP 服务器，不必再开端口或加 mcpo 代理；对自托管用户来说，这相当于把本地模型与工具接进统一聊天入口，同时不把内网地址和密钥交给远端。
[原文](https://github.com/open-webui/open-webui/releases/tag/v0.13.4)

**7. OpenHands 1.41.0 让 ACP Agent 自报推理强度，发布说明列出 48 个 PR**
11 月 16 日发布的 OpenHands 1.41.0 把 ACP 的推理强度改为按 Agent 自报能力选择，避免对不支持的模型硬塞参数；Hooks 对话框开始点名 workspace 与 user 两个 hooks 文件，automations 面板支持按 tag 分组，并在自动化设置页打开辅助模板、加上 CVE Remediation Factory 图标。发布说明共列出 48 个 PR，其中 verify-openhands 补上了 map 控件、浏览器时区等“非 happy path”场景，继续把“Agent 干完活有没有证据”做成可回归的验收项。
[原文](https://github.com/OpenHands/OpenHands/releases/tag/v1.41.0)

**8. TrnRT 把 GLM-5.3-Flash 塞进单台 Trainium2：每分钟 357 万 token，每百万 token 4 美分**
10 月 30 日创建、11 月 18 日仍在更新的 TrnRT 是 Apache 2.0 的 AWS Trainium 推理引擎，用 NKI megakernel 把解码压到每层半个 kernel，并保留跨设备与主存的 prefix cache。官方数据称单台 trn2.48xlarge 在 AgentX、batch 12 下达到 357 万 token/分钟，batch 8 单步解码从 32.2 毫秒降到 24 毫秒；按 spot 价折合每百万 token 0.040 美元，容量块价 0.167 美元。目前首个支持的模型是 GLM-5.3-Flash，仓库已有 1,629 星。
[原文](https://github.com/RunanywhereAI/trnrt)

**9. 微软 agent-framework 两个版本被投毒：.pth 启动钩子偷 Azure 与 CI 凭证**
11 月 17 日，PyPI 上的 agent-framework 1.26.1 与 1.26.2 被植入凭据窃取代码，分别在 20:35 UTC 与 21:08 UTC 发布，距上一个干净版本 1.26.0 只差一个补丁号。恶意包在 site-packages 根目录放下 agent_framework_meta.pth 与 af_runtime_diag.pth，借 Python 每次启动都会执行 .pth 的机制运行，目标包括 Azure、Azure DevOps、GitHub Actions 与 CI/CD 凭证；Chainguard 称在发布后 48 秒与 42 秒内完成拦截，两个版本存活均不足一小时。GHSA 建议把装过 1.26.1 与 1.26.2 的环境视为已泄露，轮换所有可达凭证；1.26.0 及更早版本不受影响。
[原文](https://github.com/advisories/GHSA-764m-3ppc-5m73)

*本期由站点自动生成 · 信息以原文为准。*
