---
title: Hy4 770B 权重全开，36B 机器人模型把动作接进语言
date: 2026-11-22T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 本期最值得看的是两条把大模型做成可部署资产的消息：腾讯混元把 770B 总参、49B 激活的 Hy4 权重全量开放，1M 上下文与多模态一并给出；Perceptron 的 Isaac 0.5 用 36B 稀疏骨干，把视频理解、空间定位与机器人动作放进同一个模型。评测侧，ADDIS 在 150 个无交互合成集上把误报从 66.1 个子群压到 0.09，Toolathlon 榜单前四名却只差 1.0 分、18 行无一行通过平台验证。工程侧，Hugging Face 的 harness 适配器让 Qwen3.8-27B 在 pi 的 Terminal-Bench 2.1 上从 52.9% 升到 57.6%，Transformers v5.24 则把 KV 缓存参数统一改名。想看清每个数字的边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 腾讯混元 Hy4 全量开放权重：770B 总参、49B 激活、1M 上下文**
11 月 20 日，腾讯混元放出 Hy4 的完整权重，官方给出的下载入口包括 Hugging Face、ModelScope 与 GitCode；模型为原生多模态 MoE，总参数 770B、每 token 激活 49B，上下文 1M，并同步发布 System Card 与 GGUF 版本，至此 Hy 系列从 Hy3 preview 到 Hy4 flash 全部开放。770B 的容量仍在，但推理时每个 token 只走 49B，算力按激活参数计，全部专家权重仍要占存储与显存；官方系统卡则补上了训练数据、能力边界与风险缓解的说明。
[原文](https://mp.weixin.qq.com/s/M0TIIvCzizkLw7_A3MCSjA)

**2. Perceptron 开源 Isaac 0.5：36B 稀疏骨干同时输出语言、坐标与动作**
Isaac 0.5 用 36B 参数的 Qwen 系视觉语言骨干加稀疏专家，读取图像、视频、语言、机器人状态与历史动作，输出文本、归一化坐标、任务进度或连续/离散动作；官方称训练覆盖 35 种机器人、10 万小时机器人经验、100 万小时通用视频与 3 万亿多模态 token，检查点已放到 Hugging Face，配套代码与 LeRobot 集成放在 Perceptron 仓库。相比 π0.5 的约 7 种机器人、SmolVLA 的单一本体，它的训练面明显更宽；代价是必须走 Perceptron 自己的仓库，Transformers 与原生 LeRobot 暂不能直接加载。
[原文](https://huggingface.co/PerceptronAI/Isaac-0.5)

**3. ADDIS 把交集偏差审计的误报从 66.1 压到 0.09**
针对“多个边缘身份叠加”的交集偏差审计，ADDIS 先用可加模型拟合各属性的独立效应，再用 Freedman-Lane 零假设检验子群的加性残差；在 150 个不含交互的合成集上，旧标准平均每个数据集误报 66.1 个子群，ADDIS 只报 0.09 个，并把已知交互的召回从 32% 提到 84%，对 13 个开源模型、738 个子群，被标记比例也从 7.1% 降到 3.0%。论文同时发布 IndiPersona：1 万条印度合成人物、6 个身份属性；性别认同与性取向的组合在每一个模型里都是最大的交集对，种姓与阶层的交互则在除一个之外的所有模型里都能被反事实人物检出。
[原文](https://arxiv.org/abs/2611.26538)

**4. 机器人基准的成功检查器有 61 处缺陷：40 次假成功、21 次真失败**
SimAudit 用 LLM 读任务指令与检查器代码，先在仿真里复现二者可能冲突的场景，再用多视角渲染核对行为；在 RoboTwin 2.0、RoboCasa365 与 RoboDojo 上，183 个测试用例全部复现了预测判定，最终确认 49 个任务里有 61 处缺陷，其中 40 处会把未完成执行判成成功，21 处会把合法执行判成失败。三类基准的缺陷方向并不一致：RoboCasa365 多数是“假成功”，RoboDojo 多数是“真失败”，RoboTwin 2.0 则两种都常见，说明只拿专家演示验收检查器远远不够。
[原文](https://arxiv.org/abs/2611.25179)

## 开源与工程

**5. 一个 rank-96 适配器让 Qwen3.8-27B 在三种编码代理里平均涨 8 分**
Hugging Face 把 pi、Hermes Agent、OpenCode 三个 harness 里用 GRPO 训出的 rank-32 LoRA 按秩拼接成 rank-96 的 super-adapter，冻结底座不改权重；开启 thinking 时，六个 harness–基准组合的平均 pass@1 从 55.5% 升到 63.5%，其中 OpenCode 在 SWE-bench Pro 的 hard-51 子集上从 47.1% 升到 66.7%，工具调用还少 19%–48%。代价也很清楚：super-adapter 在 pi 上只是追平 base，单 harness 且关闭 thinking 时专用适配器反而更高，AutomationBench 的域外结果则基本落在噪声内。
[原文](https://huggingface.co/harness-hub/Qwen3.8-27B-super-adapter)

**6. Transformers v5.24 统一 KV 缓存命名，旧参数保留到 v6.0**
v5.24.0 把 `past_key_values` 与 `cache_params` 统一改名为 `cache`，旧名在 forward 里保留到 v5.40.0、在 generate 里保留到 v6.0.0，同时要求 peft>=0.22.0 并移除 `modeling_attn_mask_utils.py`；新增模型包括覆盖 1,600 多种语言的 Omnilingual ASR（300M–7B）、7.2B 总参/0.6B 激活的 Lara Mini 与 105B 的 SarvamMLA。改名覆盖 forward 签名、generate、ModelOutput 与自定义 `Cache.update`，传旧名会触发一次弃用警告，同时传新旧名则直接报 ValueError，升级前最好先把远端代码里的自定义缓存类改掉。
[原文](https://github.com/huggingface/transformers/releases/tag/v5.24.0)

**7. Claude Code 2.1.329 把 /tasks 做成会话总账**
11 月 19 日发布的 2.1.329 让 `/tasks` 一次列出当前会话的定时任务、`/loop` 唤醒与 fork，`/code-review` 改为对整条分支与默认分支做差异审查；托管设置里的 hook 加载失败不再静默，会直接在会话里给出警告，Code Mode 也能同时开启 OpenTelemetry 事件与指标。对把 hook 下推到托管设置的团队，失败不再只留在日志里；对用 `/loop` 跑长任务的用户，`/tasks` 把定时、唤醒与分支会话放进了同一视图。
[原文](https://github.com/anthropics/claude-code/releases/tag/v2.1.329)

**8. Toolathlon 榜前四只差 1.0 分，18 行没有一行通过平台验证**
11 月 21 日抓取的 Hugging Face Toolathlon 榜单有 18 行，0 行带 verified 标记、15 行来自未合并 PR；前四名 76.9、76.5、76.0、75.9 只差 1.0 分，而该基准共 108 题，一题就值 0.93 分，所以榜首之争实际只相当于一道题。榜单的 task id 叫 `toolathlon_verified`，但这里的 verified 指题库经过校验，与平台是否核验过某一行是两件事；0 行通过平台验证，意味着这些分数仍以模型卡自报为主，读榜时更该看“同一 harness、同一预算”下的对比。
[原文](https://dev.to/ward_ed_6b5e6aa8ded94a987/toolathlon-on-hugging-face-ranks-1-to-4-sit-within-10-point-on-a-108-task-set-and-0-of-18-rows-3enj)

**9. 技能包比框架跑得快：mattpocock/skills 一周涨 5.6k 星**
11 月 21 日，mattpocock/skills 以 328,726 星超过 obra/superpowers 的 322,708 星，升到全球第 13；同一天 Claude-BugHunter 凭 83 个技能、15 个 slash 命令与 681 份公开报告整理的 24 类漏洞打法单日涨 1,349 星，Google Labs 的 stitch-skills 也有 10,954 星，说明“把经验打包成可安装技能”正在成为独立的分发单位。这类技能包以 Markdown 定义与轻量脚本为主，通常一条命令就能接入 Claude Code、Codex、Gemini CLI 等代理；分发门槛低，也解释了它为什么比底层框架涨得更快。
[原文](https://quidproquo.cc/posts/daily/2026-11-21-ai-agent-github-digest-en/)

*本期由站点自动生成 · 信息以原文为准。*
