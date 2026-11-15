---
title: DeepSeek 把五项对话能力拆成接口，阿里代码审查单日涨 897 星
date: 2026-11-15T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头对准“能力怎么被拆成可调用、可验证的零件”。DeepSeek 把 App 的搜索、文件、链接阅读与安全校验并入 API，五项能力在 deepseek-flash 上开参数即用，搜索抓取按千次 5 元计费；阿里则把内部代码审查助手开源，单日涨 897 星，靠确定性流水线把 token 压到通用 agent 的九分之一。研究侧，TREAT 用 737 个定理的等价变形测出最强模型仅 60.73% 的识别率，NeurIPS 的“代码解释”把解释缩短 6 到 100 倍。工程侧还有 vercel-labs/ui、pydantic-ai 2.67、n8n Nod 与 WebGPU 画布 gpucanvas。想看清每条的数字与边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. DeepSeek 把 App 的五项对话能力拆成接口，搜索抓取每千次 5 元**
11 月 13 日，DeepSeek 在 API 变更日志里把 App 的联网搜索、深度思考搜索、文件输入、链接阅读与安全校验并入现有接口；Messages、Responses、Chat Completions 三套协议下的 deepseek-flash 只需开参数即可获得，工具调用与上下文拼接由服务端完成。Search、Fetch、OCR、ASR、TTS 另列为独立接口，搜索与抓取每千次 5 元（海外 0.75 美元），文件解析与 OCR 每千页 0.225 美元；官方还开源了可导入历史对话的 chat-demo。对中小团队，复刻一个对话助手从“搭系统”变成“配参数”，但检索质量与拒答策略也随之上收，调试空间变小。
[原文](https://api-docs.deepseek.com/updates)

**2. TREAT 基准：737 个定理、29900 条等价变形，最强模型只认出 60.73%**
11 月 12 日公开的 TREAT 把定理条件改写成残差方程、见证陈述、优化恒等式等七类等价形式，用 737 个定理身份、29,900 条变形和 960 题固定评测面板，测模型能否从陌生写法找回标准定理。测试中最好的模型只命中 60.73%，其余系统还出现拒答、认错定理与结构化输出畸形。它说明模型对形式知识的掌握在等价改写面前很脆，适合用来检查数学 agent 的检索与工具链是否真的稳。
[原文](https://zenodo.org/records/24179071)

**3. NeurIPS 论文让模型用代码解释自己：比思维链短 6 到 100 倍**
11 月 10 日提交的 NeurIPS 2026 论文提出“代码解释”：让程序在一整个数据集上复现模型行为，用 Consistency 指标衡量程序与模型输出的一致比例，再借执行自动验证。作者在 3 个开源代码模型、8 项任务上测试，代码解释比思维链短 6 到 100 倍，在 24 个设置中有 9 个更贴合模型行为；但语言成分复杂的任务上，一致性会掉到与无信息基线相当。给 Claude Opus 5.5 加入一致性反馈与执行结果后，解释质量提升，复杂任务仍是短板。
[原文](https://arxiv.org/abs/2611.14704)

**4. NextTransformer 开放 Arena v2：同算力下比较 30 多种架构**
11 月 14 日，NextTransformer 的 Arena v2 按计划上线，允许任意 LLM 在统一协议下提交架构。所有方法都在 ClimbMix 的 49.152B token 上训练，每 token 固定 4.672 GFLOPs，跑 46,875 次 1,048,576 token 的更新，并在 189 个验证点与 6 个检查点做 CORE-22 评测；目前已有 31 次完整训练、36 张模型卡，MIT 许可。它把“谁的架构更好”从单点刷榜变成可复现的等算力对比，也给 agent 自动做架构研究提供了带计量算力的试验场。
[原文](https://github.com/Interactive-Active-AI/NextTransformer)

## 开源与工程

**5. 阿里开源代码审查工具：单日涨 897 星，只用通用 agent 九分之一 token**
11 月 14 日，阿里内部用了两年的 AI 代码审查助手 Open Code Review 在 GitHub 上单日新增 897 星，总数达到 51,009、分叉 3,804，Apache-2.0。它把文件选择、捆绑、规则匹配与评论定位交给确定性流水线，只让 agent 做动态检索与判断；官方在 50 个仓库、200 个真实 PR、10 种语言、1,505 条标注缺陷的 AACR-Bench 上称，同等模型下精度与 F1 高于通用 agent，token 消耗约为九分之一。代价是召回率更低，属于刻意用覆盖率换信噪比。
[原文](https://github.com/alibaba/open-code-review)

**6. Vercel 开源 agent UI 行为库：24 小时涨 929 星，适配五个框架**
11 月 15 日的 GitHub 快照里，vercel-labs/ui 以 24 小时 929 星升到 AI 仓库日增第一，累计 4,449 星，Apache-2.0。它把键盘导航、焦点、选择、弹层、拖拽与 ARIA 行为收进 @ui-sdk/core，再提供 React、Vue、Svelte、Solid 与 Web Components 适配；组件不预设样式，agent 可通过 MCP 与 WebMCP 绑定表单、开关和弹窗。对“让 agent 操作真实界面”这条路线，它补的是可被机器稳定调用的交互原语，而不是又一套皮肤。
[原文](https://github.com/vercel-labs/ui)

**7. pydantic-ai 2.67 补上三家新模型，嵌套指令默认封顶 32000 字符**
11 月 10 日发布的 pydantic-ai 2.67.0 把 Claude Fable 5.2、Claude Mythos 5.2 与 Gemini 4 Carbon 加进模型清单，并支持 OpenAI 的 fast/ultrafast 服务档。工程侧新增 RepoContext 按目录加载指令、每轮对话嵌套指令默认封顶 32,000 字符，Code Mode 的远程沙箱改走 gRPC（pydantic-monty 1.5.0），还把工具护栏判定记成可持久化操作。它反映的是 agent 框架的竞争点：不再比谁会调模型，而是比谁能把上下文、沙箱、重试与审计做成可回放的运行时。
[原文](https://github.com/pydantic/pydantic-ai/releases/tag/v2.67.0)

**8. n8n 开源本地 agent Nod：678 次跑完 98%，零危险动作**
11 月 10 日，n8n 团队发布本地个人 agent Nod，Apache-2.0，macOS 14+ 与 Windows 11 预览可用，模型可选 OpenAI、Anthropic、Gemini、Bedrock 等 20 多家或自建服务。它把信任检查、密码库、支付与限额放在服务端，浏览器与工作区各自跑在容器里；官方用 226 个任务、每个 3 次共 678 次运行测得 98% 完成、0 次危险动作，LongMemEval-S 记忆召回 93.2%，WhisperBench 216 次攻击中 0 条虚假记忆。所有会改变外部状态的动作都要先问用户。
[原文](https://github.com/n8n-io/nod)

**9. gpucanvas 把 5000 个实时 DOM 搬上 WebGPU 画布，帧率 117 对 23**
11 月 12 日创建的 gpucanvas 用 WebGPU 与 WGSL 把成千上万个真实 DOM 节点画进无限画布，节点仍是活的 React 或原生元素，点击、输入与状态照常工作。作者在 M 系列 Mac、Chrome 157 上测得 5,000 个界面平均 117 fps，纯 DOM 方案为 23 fps；10,000 个时是 108 对 9 fps。它还给 agent 留了 JSON 视图：一屏约 35 个条目、3.5 KB，读取约 3 毫秒，可返回元素 id、文本与坐标供自动点击。当前版本 0.4.4，API 仍可能变。
[原文](https://github.com/berkantay/gpucanvas)

*本期由站点自动生成 · 信息以原文为准。*
