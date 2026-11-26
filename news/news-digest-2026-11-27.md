---
title: 9B 全双工模型边聊边派活，24KB 汇编裸机跑起 Qwen
date: 2026-11-27T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期最值得看的是交互模型开始把“边聊边干活”压进同一次推理：上海 AI Lab 的 9B Sage-Interaction 在 FD-Bench v1.5 拿到 80.73 分，轮次切换延迟 1.29 秒，还能把长任务丢给后台代理异步执行。评测侧，PACE 复盘 9 项多智能体研究后发现，一旦补上采样与算力预算，123 组相对贪心基线的优势里有 53 组归零，提醒读者别把“更多代理”当成免费增益。工程侧，腾讯 DFly2 复用目标模型的稀疏 KV，把 64K 上下文解码提速 3.48 倍；llm.asm 则用 24KB 纯汇编把 Qwen3-0.6B 装进可启动的裸机系统。四条研究与五条工程消息都给出了可核对的数字和原始出处，想看清每个数字的适用边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 上海 AI Lab 开源 Sage-Interaction：9B 模型边聊边把任务派给后台**
11 月 23 日放出的 Sage-Interaction 基于 MiniCPM-o 4.5，参数量 9B，可同时接收流式音频与视频并输出语音，在 FD-Bench v1.5 上平均 80.73 分，FD-Bench v1 的轮次切换延迟为 1.29 秒；用户打断后的响应得分 83.00，背景声后恢复 96.94，日常委派任务拿到 80.47 F1。它的关键设计是把长任务提交给后端代理异步执行，前端继续维持对话，权重与代码以 Apache 2.0 开放，适合拿来验证“交互层”和“推理层”拆开后的系统边界。
[原文](https://github.com/InternLM/Sage-Interaction)

**2. PACE 复盘 9 项多智能体研究：补上采样预算后，53 组优势归零**
11 月 22 日更新的 PACE 把“多代理更强”拆成协调、集成与测试时算力三个来源，审计 9 项已发表研究、72 个同时报告贪心与采样基线的单元格，发现采样基线平均把多代理增益压低 0.036，其中 64 个单元格为负；123 组击败贪心基线的配对对比里，有 53 组在采样基线下不再为正。作者在 MMLU-Pro 的受控实验里还看到，辩论比等 token 的自一致性低 0.015，说明不少“协作收益”其实只是多花了采样与思考预算；论文同时放出七项检查清单与评测卡，要求把采样次数、token 预算和基线类型一并登记。
[原文](https://arxiv.org/abs/2611.17610)

**3. LLift 让一份检查点适配三档 SVD 压缩，不必逐档微调**
11 月 25 日的论文提出 LLift：先在单一压缩比上用原模型作教师微调低秩近似，再把学到的更新合并回原权重，同时保留被排除在前向之外的分量，使同一份检查点可以在不同截断秩上复用同一组分解。作者在 LLaMA-7B 上用一个准备模型覆盖三档压缩比，三档都取得精度增益，并称这一结论可跨模型家族、模型规模与不同 SVD 算子复现，为“一次准备、多档部署”提供了可操作的路径；相比每档都重训一个压缩模型，省下的是重复微调与多份检查点的存储开销。
[原文](https://arxiv.org/abs/2611.34667)

**4. 代理恢复流程会悄悄降级安全，ARGUS 用 864 次运行验证拦截**
11 月 25 日的论文把“工作流安全降级”定义为代理在操作失败后自主重规划、却丢掉原工作流防护要求的现象，并在可执行的 MCP/Skill 环境里对 7 个模型、12 个任务模板、3 个安全敏感领域做故障注入。ARGUS 把资源绑定的安全证据、隔离的恢复计划校验与执行期强制组合起来，在单独的 864 次运行中拦下全部已观测到的防护丢失事件；它提醒工程团队，恢复逻辑不能只以“任务是否完成”为验收标准。
[原文](https://arxiv.org/abs/2611.33605)

## 开源与工程

**5. 腾讯开源 DFly2：复用目标模型稀疏 KV，64K 解码提速 3.48 倍**
11 月 26 日，腾讯混元把 DFly2 的权重、训练代码与技术报告一并开放，草稿模型覆盖 Hy4 flash、DeepSeek-V4.1-Flash 与 GLM-5.3-Flash 三个目标。它直接读取目标层缓存的 KV 和 indexer 选出的固定位置，每个草稿 token 最多只看约 2K 个键，不额外维护缓存；在 Hy4 flash 的 CodeSpan 64K 上平均接受长度达到 4.12，解码提速 3.48 倍、端到端 2.51 倍；论文还给出 DeepSeek-V4.1-Flash 在高并发、同等单用户解码速度下的吞吐增幅，两组 64K 测试分别高出 62% 与 81%。
[原文](https://github.com/Tencent/AngelSpec)

**6. 24KB 纯汇编裸机系统跑起 Qwen3-0.6B，启动最快 74 毫秒**
11 月 25 日发布的 llm.asm 用 100% x86-64 NASM 写了三个项目：14,712 字节的 Linux 推理 ELF、24,252 字节的可启动裸机 OS、32,944 字节的 Mini-GPT 训练器，全部不依赖 libc 或任何运行时。裸机版在 QEMU KVM 上 74 至 94 毫秒启动，16 vCPU 解码 134.9 token/s、峰值 136.6 token/s；作者称在 1 到 40 线程区间比 llama.cpp 的 CPU Q8_0 快 18% 到 36%，并与 PyTorch FP32 对齐到 0.99965 余弦、64/64 贪心 token 一致。
[原文](https://github.com/alesha-pro/llm.asm)

**7. Graphvein 用确定性代码图谱给代理查调用链，正确次数从 53/63 升到 62/63**
11 月 25 日放出 0.1.1 预览的 Graphvein 会索引前端、后端与数据库代码，把路由、处理器和数据表连成一张可查询的图，索引过程不把代码发给模型，并提供本地 Web UI、CLI 与只读 MCP Server。官方用 21 个问题、3 个代理各跑 3 次做对照：接入 MCP 后 Claude Code 的正确次数从 53/63 升到 62/63，opencode 从 56 升到 61；Codex 在补充服务端指令后，主动调用工具的次数从 2/63 升到 26/63。
[原文](https://github.com/graphvein/graphvein)

**8. Google Workspace 八个 MCP Server 转正，2027 年起代理访问须走 MCP**
Google 在 11 月 24 日把 Workspace MCP Server 与 CLI 转为正式可用，覆盖 Gmail、Drive、Docs、Sheets、Slides、Calendar、Chat、Tasks 八个应用，Keep 在计划中，两周内完成铺开。配套规则要求把用于 MCP 的 OAuth 客户端单独标记为 agentic；2027 年 2 月 1 日起代理访问必须走 MCP，免费 Standard 档为每项目每产品每天 5,000 次请求，Scaled 档从 4 月 1 日起计费，现有直连 REST 的代理需要提前迁移。
[原文](https://www.techrepublic.com/fr/article/news-google-workspace-ai-agent-access/)

**9. Claude Artifacts 可装成桌面应用：Beta 向多数档位开放，权限分三级**
Anthropic 从 11 月 24 日起推送 Install as app Beta，Mac 与 Windows 版 Claude 6.28899.0 及以上可以把交互式 Artifact 装成带独立图标的桌面应用，底层 Artifact 被修改后应用会自动更新，Docs、Slides、Sheets 暂不支持这种方式。权限分为默认、应用权限、完整电脑访问三级；Team 默认开启前两级、关闭完整访问，Enterprise 三级默认全关，完整访问只限用户自建或组织内应用，且 Anthropic 不会在安装前审核应用。
[原文](https://www.techrepublic.com/fr/article/news-claude-artifacts-desktop-apps/)

*本期由站点自动生成 · 信息以原文为准。*
