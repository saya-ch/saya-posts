---
title: GPT Image 2.6 六秒出图，Gemini 视频延到 15 秒
date: 2026-11-07T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 本期两条产品线都在压缩生成时间：OpenAI 的 GPT Image 2.6 把出图最快压到 6 秒，并新增 GIF、SVG 与标记编辑；Google 的 Gemini Omni 1.2 Flash 把单段视频从 10 秒拉到 15 秒，720p 价格降到每秒 0.07 美元。但同周的两项研究给出了另一面：Ai2 的 StateBench 显示四个前沿模型在最难的长文档状态追踪上跌到 13% 以下，斯坦福复测 50 条 LLM 结论则发现 44% 已经失效。两项结果都提醒，能力描述必须绑定具体模型与评测条件。工程侧还有 NVIDIA 四个 16B 开放世界模型、OpenClaw 免发版模型目录、ComfyUI 0.44 与 LiveNerf 的漂移监测。想看清这轮“更快”究竟换来了什么，展开正文逐条核对。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. GPT Image 2.6 双模型上线：出图快 50%，最快 6 秒**
OpenAI 11 月 4 日把 Images 2.6 推向 ChatGPT、Codex 与 API，拆成 gpt-image-2.6-flare 与 sunburst 两档：前者在低质量档最快 6 秒出图，后者主打多次编辑后的细节一致。官方称整体生成速度提升 50%，新增 animated GIF、SVG 导出、markup 局部编辑与 complexity 三档规划参数；图像输入/输出仍为每百万 token 8/30 美元，但同一张图的输出 token 最多可到 2.5 的 4.2 倍。这是 Images 系列两个月内的第二次迭代，2.5 于 9 月 8 日发布；GIF 与 SVG 把输出从静态图扩展到可循环动图与矢量素材，品牌和游戏资产是直接受益场景。
[原文](https://openai.com/index/introducing-images-2-6)

**2. Gemini Omni 1.2 Flash 转正：单段视频 15 秒，720p 每秒 0.07 美元**
Google 11 月 5 日将 gemini-omni-1.2-flash 开放 GA，接入 Gemini API、Gemini for business 与 Google Vids。原生片段上限从 10 秒提高到 15 秒，草稿支持 360p/720p，成片可升到 1080p 或 4K，720p 价格下调 30% 至每秒 0.07 美元；新增多轮对话式改片，官方称对人物、服装、包装文字与品牌色的参考一致性更好。付费层速率上限翻倍，1.1 版保留四周过渡，12 月 3 日起自动路由到 1.2。Google 同时把 Omni 推进 Workspace 的 Vids，生成、改片与剪辑被放进同一条时间线，视频生产从“出一段素材”变成“在时间线上反复改”。
[原文](https://cloud.google.com/blog/products/ai-machine-learning/gemini-omni-1-2-flash-generally-available)

**3. Ai2 StateBench：最难状态追踪，四个前沿模型跌到 13% 以下**
Ai2 11 月 3 日发布 StateBench，把长文档状态追踪拆成四级，最长序列含 64,000 次赋值，是其小模型训练长度的 250 倍。Gemini 3.8 Flash、Claude Opus 5.5、GPT-6 Sol 与 Luna 在 r-trivial 与 aperiodic 上表现正常，但在最难的 general finite 长序列上准确率跌到 13% 以下。直接训练的 275M Gated DeltaNet 与混合模型能外推到数万次赋值，纯 Transformer 始终学不会最难一级；同一问题换成表格编辑格式，部分模型又从接近满分掉到随机水平。Ai2 指出，长上下文评测常把“找到信息”当成“跟住状态”，两者并不是一回事，这也是 StateBench 要分四级、并同时测试多种呈现格式的原因。
[原文](https://allenai.org/blog/statebench)

**4. 斯坦福复测 50 条 LLM 结论：44% 已失效，平均寿命约 16 个月**
斯坦福团队 11 月 2 日发布追踪研究，用 20 个模型、8 家厂商复测 50 条被广泛引用的 LLM 结论。截至 2026 年 9 月，44% 已失效、34% 明显减弱、16% 仍成立、6% 反而增强；22 条失效结论从发表到失效平均 15.7 个月，6 个月、1 年、2 年的失效率分别为 4%、16%、39%。引用伪造率从原论文的 55% 降到 Sonnet 5.5 的 1.37% 与 GPT-6.1 Sol 的 0%，但 Qwen3.7-Flash 仍达 69%，说明“普遍失效”不等于每个模型都过关。研究者据此建议，论文应标注所用模型的版本与日期，并把提示词和分析代码一并公开，否则结论很快失去可复现性。
[原文](https://zou-group.github.io/llm-claims-shelf-life)

## 开源与工程

**5. NVIDIA 开放四个 Cosmos 3 检查点：16B 参数，面向交互式世界模拟**
NVIDIA 11 月 5 日再放出四个 Cosmos 3 Nano 后训练检查点，每个含 16B 可训练参数，覆盖闭环自动驾驶仿真、最多 11 路摄像头加 LiDAR 的合成数据、双臂机器人策略仿真，以及深度/分割/边缘/模糊等辅助模态迁移。权重以 OpenMDW 1.1 发布，允许商用与非商用，技术报告、下载地址与训练配方同步公开。对做具身智能的团队，这相当于把“可交互的世界”做成可下载的基础设施，而不是只给一段演示视频。四个检查点都从 Cosmos 3 Nano 后训练而来，不是另起炉灶的新模型，因此更接近“拿现有底座改成自己的仿真器”的工程路径。
[原文](https://huggingface.co/blog/nvidia/cosmos-interactive-world-simulation)

**6. OpenClaw 把模型目录搬出安装包：约 1,000 个模型免发版可见**
OpenClaw 11 月 6 日发文复盘一次重构：用约五周、近 80 个 PR，把模型清单从随包发布的 manifest 改成三条实时来源——每四小时重建一次的托管目录（约 1,000 个模型）、用户账号返回的可用模型、以及本机安装的 Codex。运行中的 Gateway 可热切换目录，OpenRouter 超时后按 30 秒到 30 分钟退避重试，默认模型切换从 68–142 秒降到 0.6–1.4 秒。团队还踩过一个坑：列表按 Codex 0.161.0 请求、实际用 0.160.0 跑，模型能选中却在首轮报 HTTP 400，最终改为“谁列模型谁跑模型”。对用户来说，多数新模型发布当天就能在旧版 OpenClaw 里选到，不必等下一次发版，前提是本机 Codex 足够新。
[原文](https://openclaw.ai/blog/use-new-models-in-openclaw-without-an-update)

**7. ComfyUI 0.44 一次接入 Wonder 4、GPT-6.1 与 PH3Image**
ComfyUI 11 月 4 日发布 v0.44.0，距离 10 月 27 日的 0.43.0 只有八天。更新把 Topaz Wonder 4 加进 Topaz Image Enhance 节点（PR #17491），新增 GPT-6.1 Terra/Sol 的 OpenAI 伙伴节点（#17493）、PH3Image 支持、Sonilo Music 1.5、可自动扩展图像输入的 TextGenerateV2，以及可单独设置 VAE 计算精度的节点；视频保存开始支持 ProRes 与不同色度抽样，同时弃用 Seedance 1.x 节点。对本地工作流用户，升级前要确认自定义节点跟上了新的前端与注意力配置。这类高频小版本也是本地生成工具目前的常态：模型侧几乎每周都有新版本，工作流侧只能按周跟进，否则新模型进不了节点。
[原文](https://github.com/Comfy-Org/ComfyUI/releases/tag/v0.44.0)

**8. LiveNerf 跑到第 41 天：Opus 5.5 没有测到“变笨”**
独立项目 LiveNerf 用 2,336 道 GPQA Diamond、MMLU-Pro、竞赛数学与 AIME 2025–26 题筛选出 78 道“有时对有时错”的题，组成固定面板，每天跑一遍 Claude Opus 5.5。11 月 3 日第 41 天的最新结果是第三个 10 天窗口相对发布周 +0.028±0.023，未达到预注册的“连续两个窗口至少偏移 3 个百分点”阈值，因此判定未检测到能力漂移。项目称一次全量运行可识别每 10 天约 7.5 个百分点的变化，成本约为每周套餐的 3.6%，代码以 MIT 开放。它把“模型是否被悄悄降级”从论坛猜测变成可复现的统计检验，也让“发布周基线”与“后续服务”被分开记录。
[原文](https://github.com/ninjahawk/livenerf)

*本期由站点自动生成 · 信息以原文为准。*
