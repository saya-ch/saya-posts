---
title: 决策模型挤进工具调用链，35B MoE 压进 3.43 GiB 内存
date: 2026-11-12T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期的主线是模型开始从“生成文字”转向“直接给出判断”。Google 的 Gemini Decision 1 不再逐 token 解码，而是在一次前向里给候选打分，单请求最多 1000 个问题、端到端 150 至 250 毫秒；Kakao 同周开源的 Kanana-2-V-Decide 把单张 H200 上的响应压到 0.04 秒。研究侧，百度的 ERNIE Evolve 用自动实验把 NanoChat 验证集 BPB 压到 0.8763，并把昆仑芯 P800 的生成速度提到基线的 6.2 倍。工程侧，33.1B 的 H3-Image 六步出 16MP 图，Cloudflare 与 Jane Street 把“怎么审计、怎么教”写成 Agent Skill，Edge0 则让 35B MoE 在 3.43 GiB 内存里跑起来。这些条目共同说明，竞争焦点已落到调用成本、可验证性与端侧可行性上，具体数字和边界见正文。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Gemini Decision 1 把“回答”改成打分，千题单次前向 150 毫秒**
Google 11 月 11 日发布 Gemini Decision 1 与 Decisions API：模型不逐 token 生成文本，而是在一次前向中对候选选项打分，返回概率与置信度；多模态输入上限 256K token、单请求最多 1000 个独立问题，端到端延迟 150 至 250 毫秒，模型计算中位数 50 至 100 毫秒。定价为每百万输入 token 0.10 美元、输出不收费，约为 Gemini 3.5 Flash-Lite 走 Interactions API 成本的五分之一，Artificial Analysis Decision Index v2.1 得分 66，API 已在 Google AI Studio 开放预览并提供 Python、JavaScript 与 REST 接入，候选分数可直接用于设阈值或路由，适合闭集判断而非需要解释或长文本生成的场景。
[原文](https://gigazine.net/gsc_news/en/20261112-gemini-decisions-1)

**2. Kakao 开源 Kanana-2-V-Decide：单张 H200 响应 0.04 秒**
韩国 Kakao 11 月 6 日把多模态决策模型 Kanana-2-V-Decide 放到 Hugging Face，基于自家 3B 级 Kanana-2-v-3B，可同时读文本与图片，用问题与选项的向量相似度直接输出概率、选择或分数，相同选项的向量可缓存复用。官方称在单张 NVIDIA H200 上平均响应 0.04 秒以内，比 GPT-6 Luna 快约 43 倍，并在韩语 Kanana Decision Bench 与公开 Decision Index 上领先微软 Decision-1、TypeSafe AI 的 Zev；它省掉了从长回答里再抽取结论的步骤，面向工单分类、换货判定与工具选择等闭集任务。
[原文](https://www.yna.co.kr/view/AKR20261106052900017)

**3. 百度 ERNIE Evolve 用自动实验把 BPB 压到 0.8763**
文心团队 11 月 9 日公开自动研究系统 ERNIE Evolve，双层循环里 Director 选方向、Selector 选起点、Executor 改代码，外层可并行多条研究线、每条维护自己的候选树，Evaluator 与 Reviewer 独立把关。三项公开任务中，NanoChat AutoResearch 验证集 BPB 降到 0.8763（社区最佳 0.8878），NanoGPT Speedrun Track 1 用 38.9 秒（最佳 39.9 秒）、Track 3 用 2590 步（最佳 2690 步），SOL-ExecBench 平均 0.819（榜首 0.827），接入 ERNIE-5.5-Flash 后昆仑芯 P800 低时延生成速度提升至基线的 623.6%，GPU 确定性吞吐提升 33.7%；但这些成绩为团队自报，社区复现尚未完成。
[原文](https://ernie.baidu.com/zh/research/posts/ernie-evolve/)

**4. Ostinato 让本地 MoE 推理不再在 prefill 与 decode 之间二选一**
11 月 8 日上传的 arXiv 论文提出 Ostinato：短 prefill 会把 84% 的专家路由打到一组能放进 GPU 的稳定专家上，每个 token 的六个路由专家里只剩不到一个留给 CPU，长 prefill 对每个冷专家只上传一次，decode 则压成固定形状的 CUDA graph。用 DeepSeek-V4-Flash 在四张 RTX 4090D 上测，工具输出后的等待中位数比 llama.cpp、KTransformers 低 22% 至 55%，会话整体完成快 1.4 至 2.5 倍，在两个阶段都快过六个基线；它不改变模型计算结果，只改变专家驻留与调度，解决的是本地 Agent 的“prefill 快、decode 慢”跷跷板。
[原文](https://arxiv.org/abs/2611.11533)

## 开源与工程

**5. MiniMax 放开 33.1B 图像模型，六步出 16MP 图**
MiniMax 11 月 10 日深夜在 Hugging Face 放出 H3-Image 权重：33.1B 扩散 Transformer、50 层，BF16 检查点 68.65GB，蒸馏到六步且不用 CFG，单检查点同时做文生图与最多 16 张参考图的编辑，4096×4096 约 16MP 输出不到 10 秒，最高支持 6272×2688，文本编码器需单独下载。许可证不是标准开源协议，欧盟、英国、韩国、美国不在授权范围内，年营收超 100 万美元或对外提供托管服务还需书面授权，官方虽称指令遵循是开源图像模型中最强却暂无独立对比，本地部署建议 SGLang、vLLM-Omni、diffusers 与 ComfyUI。
[原文](https://pandaily.com/minimax-h3-image-open-weights-16mp-text-to-image-editing-six-step)

**6. Cloudflare 开源六阶段安全审计 Skill，让新 Agent 专门证伪**
Cloudflare 把内部漏洞发现流水线的起点整理成 security-audit-skill：侦察建信任边界与覆盖台账，按台账分派猎手，每个候选漏洞交给全新的验证 Agent 尝试推翻，最后输出 confirmed、needs_validation、rejected 三类结构化记录。仓库为 MIT，约 6.9 万星、一周新增约 1.15 万，官方说单次运行大约只能找到多次运行合计的一半漏洞，多次运行会按上次台账补覆盖缺口；它要求 OS 级沙箱才能执行目标代码，否则结论只能停在 needs_validation。
[原文](https://github.com/cloudflare/security-audit-skill)

**7. Jane Street 的第一个 Skill 不教写代码，九天 6061 星**
量化机构 Jane Street 11 月 3 日创建 agent_skills 仓库，至今只有一次推送、一个 SKILL.md，却拿到 6061 星：socratic-tutor 先做诊断，给出 0 到 5 级技能地图，一次只问一个问题，穿插间隔复习，最后做考核。它不绑定模型，任何支持 SKILL.md 的 Agent 把目录拷进去就能用，普通聊天机器人可直接粘贴正文，仓库采用 MIT 协议；对大机构来说，Skill 正在成为发布“做事方式”的载体，而不是又一个工具。
[原文](https://github.com/janestreet/agent_skills)

**8. Edge0 把 35B MoE 压进 3.43 GiB，苹果 M5 解码 24.8 tok/s**
Edge0 11 月 4 日发布统一推理引擎 edge0-infer，同一套代码覆盖 macOS、iOS、Android 与 Windows：专家权重放 SSD 按需流式加载，Recover-LoRA 补 4-bit 量化损失，预路由器提前一步预测专家路由，官方称解码吞吐最高提升 59%。实测 edge0-35b 在 16GB 苹果 M5 上 Metal 解码 24.8 tok/s、峰值内存 3.43GiB，3084 token 提示首字 5.6 至 5.87 秒，iPhone 17 Pro Max 上 13.2 tok/s、3.05GiB，代价是首字延迟以秒计，适合后台与离线任务而非实时对话。
[原文](https://github.com/Edge0-AI/Edge0)

**9. MuJoCo 4.0 用推测接触防穿模，薄物抓取快 6 至 9 倍**
Google DeepMind 11 月 6 日发布 MuJoCo 4.0.0：Studio 从 experimental 转正，新增 speculative contacts，提前扫掠高速几何体并最多生成四个接触点，官方测试中机械臂以 16 毫秒步长每次接住 0.7 毫米、下落超 2 米/秒的纸牌，而离散碰撞在 8 毫秒以上就会漏掉。人形机器人上下楼梯与薄物抓取因此快 6 至 9 倍，MuJoCo 每月 PyPI 下载约 360 万次、GitHub 星标超过 1.6 万，碰撞栈换成原生凸碰撞后整体约快 4 倍，Python 包拆出只依赖 numpy 的 mujoco-core，并停止支持 Python 3.10。
[原文](https://github.com/google-deepmind/mujoco/discussions/3773)

*本期由站点自动生成 · 信息以原文为准。*
