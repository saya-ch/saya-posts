---
title: 三元哥德巴赫证明进 Lean，生信模型盲测压过 Opus-5
date: 2026-11-15T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期的主线是“把前沿能力压进可验证、可部署的形态”。独立开发者 Charlie Reams 用 Claude Code 把三元哥德巴赫猜想形式化到 Lean 4，19.4 万行代码零 sorry，外部假设从 5 条压缩到 2 条；无界前沿开源的 Fx-Bio-1031 只激活 6B 参数，却在 BioMysteryBench 困难子集拿到 70.59%。产品侧，ElevenLabs 把 102 种语言的转写词错率再降 27%，OpenAI 则让 Dots agent 拿支付 token 代下单，每笔都要指纹确认。工程侧，星尘智能的多机协作把通信量砍掉 80%，Inkling 2 Small 以 594B 总参数放开 Apache 2.0 权重，Fireworks 的供应链泄露则提醒第三方 API 的边界。想看清每条的数字与取舍，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 三元哥德巴赫猜想进 Lean：19.4 万行代码零 sorry，外部假设压缩到 2 条**
独立开发者 Charlie Reams 借助 Claude Code，把 Helfgott 2013 年对三元哥德巴赫猜想的证明完整形式化到 Lean 4：仓库累计约 19.4 万行代码，主定理不含任何 sorry，只依赖 Lean 的三条标准公理。更关键的是把外部假设从 5 条压缩到 2 条——Platt 的 GRH 验证（黎曼假设到 10^8）与二元哥德巴赫到 4×10^18；n≥10^27 的大范围情形不再需要后者。证明里的素数梯子共 254,638,308 级，全部由 Lean 内核逐级检查，仓库还记录了原书稿中常数写错、不等式不成立等多处问题。它把“证明完成”的判定从同行评议推向机器可验证的闭环。
[原文](https://github.com/calr3/ternary-goldbach)

**2. 无界前沿开源 Fx-Bio-1031：125B MoE，生信盲测 70.59% 压过 Opus-5**
11 月 13 日，无界前沿开放生物信息大模型 Fx-Bio-1031 的完整权重。它基于 Qwen3.8-Flash-Next 做 RSI 后训练，总参数 125B、每 token 仅激活 6B，原生支持 512K 上下文，采用 Qwen 社区许可 1.0；官方称医院与课题组可在本地机房私有化部署，组学数据不出域。在 Anthropic 的 BioMysteryBench 困难子集上，它的 Pass@5 为 70.59%，高于同框架 GPT-6-Astra 的 47.06% 与 Claude Opus-5 的 52.94%；BiomniBench 的 Pass@3 达 89.50%。乳腺癌队列案例中，它锁定 762 名独立患者，算出 ESR1 与 RAS/MAPK 互斥的 OR=0.317、p=0.0011，又主动补了 14 项敏感性分析与阴性对照。
[原文](https://www.qbitai.com/2026/11/509663.html)

**3. ElevenLabs 发布 Scribe v3：102 种语言，词错率比上代低 27%**
11 月 10 日，ElevenLabs 推出语音转写模型 Scribe v3 与低延迟版 v3 Live。Scribe v3 在 102 种语言的 FLEURS 测试中把词错率较 v2 降低 27%，德语、西班牙语、英语分别改善 42%、35%、23%；三人以上会议的说话人分离错误减少 17%，实时版 v3 Live 的语义词错率较 v2 Realtime 降低 43%，并首次把说话人分离带进实时转写。两款模型已进入 ElevenAPI，切换只需改一行 model ID；关键术语提示最多支持 1,000 个词，适合会议记录、客服与临床文档。
[原文](https://elevenlabs.io/blog/scribe-v3)

**4. 火山引擎 Seedance 2.5 flash：720P 每秒 0.99 元，比旗舰便宜 34%**
11 月 12 日，火山引擎推出视频生成模型 Seedance 2.5 flash，官方称生成速度较 Seedance 2.5 翻倍，720P 文生/图生视频价格约 0.99 元/秒，较旗舰低约 34%。它单段最长支持 30 秒，覆盖 480P 与 720P，单次最多可输入 10 张图片、5 段视频和 5 段音频，适合批量短视频与电商素材，即日起陆续进入即梦、LibTV、Updream、Creao AI、TapNow 等平台，API 则于 11 月 15 日上线火山方舟。把“够用画质 + 可算成本”放在同一条产品线上，说明视频模型的竞争点正从演示效果转向单位内容成本。
[原文](https://www.cnstock.com/commonDetail/827612?commTag=true)

## 开源与工程

**5. OpenAI 上线 ChatGPT Wallet：Dots 代下单，每笔都要指纹确认**
11 月 13 日，OpenAI 向美国 18 岁以上、使用美国账单地址与合格 Amex、Chase、Mastercard、Visa 卡的 Pro 用户开放 ChatGPT Wallet。绑卡后 Dots agent 可按提示购物，但每次结账前都要用户确认，并通过 Face ID 或指纹做生物识别；真实卡号被替换为支付 token，agent 与商家都拿不到原始卡信息。OpenAI 计划后续允许用户用消费限额、购买品类与时间窗口预授权，把整笔采购交给 agent。落地阻力在渠道侧：部分零售商限制甚至封禁 AI agent，亚马逊此前封过 Meta 的 Muse，网站风控需要先学会识别“合法 agent 发起的交易”。
[原文](https://www.theverge.com/ai-artificial-intelligence/1033425/openai-chatgpt-wallet)

**6. 星尘智能 CoMind：隔墙协作成功率从 0 到 100%，通信量降 80%**
11 月 13 日，深圳星尘智能发布多机协同框架 CoMind（Be My Eye）。它不做中心化调度，也不互传完整摄像头画面，而是让每台机器人把本地观测压缩成任务相关的上下文，再双向共享、独立生成动作；训练时队友的动作误差会反传回发送方的信息编码器，迫使它“说重点”。六项真机测试覆盖隔墙拿笔、盲盒置物、纸箱组装、叠被收纳、冰箱补货与哑铃组装：隔墙拿笔成功率从 0% 提到 100%，盲盒置物从 38.9% 提到 88.9%，通信量减少约 80%。技术报告已公开。
[原文](https://www.astribot.com/AI/CoMind)

**7. 40 核老服务器不插显卡跑 Kimi K3：1-bit 量化 0.85 token/s**
11 月 13 日，一位玩家在社区分享：他用 40 核第三代至强处理器与已停产的英特尔傲腾持久内存 200 系列，在没有 GPU 的服务器上跑起月之暗面 2.8 万亿参数的 Kimi K3。Kimi K3 每 token 激活约 1040 亿参数，标称上下文 104 万 token；即便压到 1-bit 量化也要数百 GB 内存。方案借 llama.cpp 把傲腾设为 App Direct 模式直接当主存，1-bit 下实测 0.85 token/s，关闭 DAX 最高 1.5 token/s，3-bit 约 0.75 token/s，输出 5 万 token 要 18.5 小时。单条傲腾功耗上限 18W，介质 83℃ 起降频、85℃ 触发保护关机；它验证的是内存墙下的可行性边界，而非可用吞吐。
[原文](https://www.aibase.com/zh/news/32314)

**8. Thinking Machines 放开 Inkling 2 Small：594B 总参数，Apache 2.0**
11 月 12 日，Thinking Machines 按 Apache 2.0 放出 Inkling 2 Small 权重，模型总参数 594B、每 token 激活 17B，支持图像输入与可变推理力度。硬件门槛不低：模型卡给出的原生 NVFP4 检查点至少要 500GB 聚合显存，官方示例为 2 张 B300 或 4 张 H200；MXFP8 版本则要 800GB。vLLM 首日支持视觉输入、混合精度 MoE 与 DSpark 投机解码，TP8 可跑满 1,048,576 token 上下文，TP4 单用户吞吐最高约 630 token/s。Code Arena 的 WebDev 分数从 1,408 升到 1,647，在开放模型中排全球第四、美国开放模型第一；Agent Arena 的中位任务成本为 0.26 美元，约为 Kimi K3.1 Pro 1.22 美元的五分之一。
[原文](https://huggingface.co/thinkingmachines/Inkling-2-Small-MXFP8)

**9. Fireworks AI 被入侵：Notion 约 1% 用户的 AI 聊天记录泄露**
11 月 13 日，Fireworks AI 披露一起供应链攻击：攻击者先入侵其云服务提供商，再取得开发系统权限，窃取部分客户账单信息与数据集。Fireworks 服务数万家客户，包括 Harvey 与 Cursor；Notion 证实约 1% 用户受影响，泄露内容是使用 Notion AI“Insights”时输入输出的聊天记录，部分数据因缺少可追溯信息而无法定位到人。Notion 已于 11 月 3 日切断连接、删除相关存储并停止全部对接。Fireworks 认为攻击者意在窃取 token 转售，这是其继 2026 年 10 月之后数月内第二起安全事件。
[原文](https://www.aibase.com/zh/news/32288)

*本期由站点自动生成 · 信息以原文为准。*
