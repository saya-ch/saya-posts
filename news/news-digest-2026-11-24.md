---
title: GLM-5.4 上线编程订阅，560B 医疗与 554B 记忆模型同周开源
date: 2026-11-24T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期的主角是三条不同路线的新旗舰：智谱 GLM-5.4 以约 7,000 亿参数上线编程订阅，上下文扩到 20 万 token，并新增视觉输入；蚂蚁百灵开源 560B MoE 医疗模型 LingSante-3.1-flash，支持 1M 上下文、采用 MIT 协议；商汤 SenseNova-Loop-9B 则用 32 层权重循环两次，把 9B 模型推到 27B 级评测区间。记忆侧，记忆张量与英伟达的 MemT-1.0-Pro 以 554B 总参数、59B 激活参数在 11 项记忆评测上拿到最高平均分。工程侧，Agent Client Protocol 把子代理会话转正，fast-anything 只把最慢的 5% 函数改写成 Rust。开放权重与订阅制在同周并行，说明厂商对“开放”的用法正在分化。想核对每个数字的边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. GLM-5.4 上线编程订阅：约 7,000 亿参数，新增视觉与 50% 额度**
11 月 23 日中午，智谱把 GLM-5.4 上线 GLM Coding Plan：约 7,000 亿参数，上下文从 12.8 万扩到 20 万 token，最大输出 6.4 万，并首次在编程订阅中支持截图与设计稿等视觉输入。据开发者社区与多家媒体报道，其底座为 Loop Transformer，若属实将是国内首个把该架构用于旗舰量产的模型；官方架构说明尚未随公告发布。订阅用户使用新模型的周额度比 GLM-5.3 多 50%，只需把模型名改为 glm-5.4，API Key 与接入地址不变。据摩根大通调研纪要，智谱还预告四季度推出约 3 万亿参数的 GLM-5.5，市场普遍预期 12 月接力。
[原文](https://news.qiniu.com/articles/glm-5-4-first-loop-transformer-release)

**2. 记忆张量开源 MemT-1.0-Pro：554B 总参数、59B 激活，11 项记忆评测平均分第一**
11 月 22 日，记忆张量发布并开源与英伟达联合研发的记忆原生模型 MemT-1.0-Pro：基于 Nemotron 3 Ultra，554B 总参数、59B 激活参数，用 Megatron-Core 与 Megatron Bridge 完成分布式训练。技术报告称它在 BEAM-1M 拿到 41.80 分、比 Flash 版高 4.76 分，MemoryAgentBench 为 23.87 分，并以 53.08 分在 11 项记忆评测中取得最高平均分。这些是厂商自报、尚未独立复现的结果，读分时要连同评测设置一起看。
[原文](https://www.geekpark.net/news/374331)

**3. 蚂蚁百灵开源 560B 医疗模型：MIT 协议、1M 上下文，InclusionMed 开源榜第一**
11 月 23 日，蚂蚁百灵开源医疗健康模型 LingSante-3.1-flash：560B 的 MoE 架构，MIT 协议，支持最长 1M token 上下文，可读整本病历、检查报告与医学文献，并通过工具调用完成多步骤诊疗任务。官方称其在推理、安全对齐与 Agent 评测上达到开源 SOTA，按 InclusionMed 综合指数位列全球开源模型首位；同日开源的还有面向多模态健康信息理解的 LingSante-3.1-air-VL。医疗场景对幻觉的容错更低，权重开放只是第一步，临床验证仍要单独做。
[原文](https://36kr.com/newsflashes/4081776064140163)

**4. SenseNova-Loop-9B 正式版：32 层权重跑两遍，9B 对上 27B**
11 月 17 日，商汤开源 SenseNova-Loop-9B 正式版：9.41B 参数、32 个解码器层，其中第 4 至 27 层共享权重并连续执行两次，一次前向完成 56 次层计算，上下文从 256K 扩到 512K，权重与代码以 Apache 2.0 开放，4-bit AWQ 量化版同日上线。官方自报 Terminal-Bench 2.1 为 59.55、GAIA 69.90、AutomationBench 36.67；与 Qwen3.8-27B 相比，AutomationBench 持平，AIDABench 以 65.31 对 65.01 微胜，NL2Repo 则以 41.36 落后。循环深度从两圈加到四圈还能涨分，但默认仍用两圈换推理效率。
[原文](https://huggingface.co/sensenova/SenseNova-Loop-9B)

## 开源与工程

**5. Agent Client Protocol 1.34 把子代理会话写进稳定 schema**
11 月 20 日至 21 日，Agent Client Protocol 连发 1.34.0 与 1.34.1：前者把子代理会话与计划操作从实验状态转为稳定 schema，编辑器、CLI 与远端代理之间因此能用同一套消息描述“谁在替谁干活、计划改到哪一步”；后者只做协议页面重排，并把 Cancellation 页改名。对一个被多家编辑器与代理客户端共同采用的协议来说，稳定项比新增项更值得关注，因为一旦写进 schema，后续只能做兼容式演进，接入方也终于敢把它放进生产路径。
[原文](https://github.com/agentclientprotocol/agent-client-protocol/blob/94824f11ec78f7a6699323a9342e47fa5edf2530/schema/v1/CHANGELOG.md)

**6. fast-anything 0.7.0：只把最慢的 5% 改写成 Rust，116 倍加速可一键回退**
11 月 22 日，fast-anything 发布 0.7.0：先采样找出应用里最耗时的函数，再让编码代理只把这 5% 改写成 Rust，随后回放最多 500 条真实调用，逐一比对返回值、浮点误差（1e-9）与报错行为，全部一致才允许启用；设置 FAST_ANYTHING_DISABLE=1 可随时退回原实现。README 给出的实测里，Python 模糊去重从 4.91 秒降到 42 毫秒（116 倍），TypeScript 版从 3.11 秒降到 155 毫秒（20 倍）。项目以 MIT 开源，并内置 8 个 MCP 工具，把“重写整个应用”换成“只换热点函数”。
[原文](https://github.com/EL4CTEO/fast-anything)

**7. ECC 2.2.3 把 328 项技能装成工程系统，单日再涨 701 星**
11 月 23 日的 GitHub 日榜上，ECC 以 308,998 星、单日新增 701 星再度上榜：它把自己定位成代理的工程系统，2.2.3 版打包 72 个代理、328 项技能与 99 个兼容命令，用 hook 与记忆把“计划、测试、实现、评审、验证、复盘”串成固定流程。随附的 AgentShield 用 102 条静态规则扫描提示词、MCP 配置、权限与密钥，官方称有 1,282 个测试、98% 覆盖率。项目 MIT 开源，Claude Code、Codex、Kimi Code 可原生安装，Cursor、Gemini、Copilot 等走适配层。
[原文](https://github.com/affaan-m/ECC)

**8. CLI-Anything 把任意软件包成命令行，单日新增 731 星**
同一份日榜上，港大团队的 CLI-Anything 以 62,745 星、单日新增 731 星排在第 18 位：它把 CAD、3D 建模、图表生成等原本只有图形界面的软件包一层命令行接口，让代理通过 CLI-Hub 注册、发现并复用这些能力，而不必为每个软件单独写工具封装。项目用 Python 实现，提供统一的安装与技能注册流程。它的价值不在某个具体封装，而在把“软件能不能被代理调用”变成一件可批量复制的事，代价则是每个封装都要跟着上游软件的版本走。
[原文](https://github.com/HKUDS/CLI-Anything)

**9. Hermes Agent v0.23.3 收进约 370 个 PR，补上提示缓存生命周期控制器**
11 月 19 日，Nous Research 发布 Hermes Agent v0.23.3：这个补丁版把 v0.23.2 以来合并的约 370 个 PR 收进稳定通道，窗口内共有 1,657 个非合并提交、3,948 个文件变更（+222,379/−96,154），关闭 60 个 issue。版本说明提到一条未展开的改动：新增提示缓存生命周期控制器，在长工具调用期间持续保温、在 1 小时缓存到期前压缩空闲会话，并默认开启工具结果主动裁剪，被压缩掉的结果会归档而不是直接丢弃。对每天跑大量代理会话的团队，这类缓存与上下文管理比多一个模型选项更直接影响账单。
[原文](https://github.com/NousResearch/hermes-agent/releases/tag/v0.23.3)

*本期由站点自动生成 · 信息以原文为准。*
