---
title: 文心 5.5 Flash 开源 280B，NInfer 让 27B 跑进 8GB 显卡
date: 2026-11-24T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头从“发了什么模型”移到“模型怎样被交付和验收”。百度在 11 月 24 日把 ERNIE 5.5 Flash Preview 的权重以 Apache 2.0 开放，280B 总参数只激活 14B，上下文 100 万 token；AWS 的 Strands Decider v2 则用五个 2B 到 31B 的决策模型，把“选工具、选模型、判断答案够不够好”压成一次前向。研究侧，ChiProgramBench 让代理从引脚实验重建电路，432 次试验只有 12.0% 完全通过；NInfer 同一天把 27B 三元模型塞进 8GB 显卡，并给出 397 token/s 的实测。工程与评测正在取代参数规模，成为这一周更值得看的变量。想核对每个数字的边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 文心 5.5 Flash Preview 开放权重：280B 总参数、14B 激活、100 万上下文**
11 月 24 日，百度把 ERNIE 5.5 Flash Preview 的权重放上 Hugging Face 与 ModelScope，采用 Apache 2.0，提供 FP4 专家、FP8 与 BF16 三种版本；模型为 43 层 MoE，其中 42 层为混合专家层，每层 512 个路由专家、每 token 激活 10 个另加 1 个共享专家，靠 6 层 MLA 与 37 层压缩注意力把 100 万 token 的 KV 缓存压住。官方自报 Terminal-Bench 4.0 46.5、MCP-Atlas 84.6、AutomationBench 61.9、BrowseComp 87.4，均需独立复现；这是继 Base 之后百度首次把后训练版本的权重完整放出，1M 上下文与 14B 激活的组合瞄准长文档与代理循环，而不是单纯刷榜。
[原文](https://huggingface.co/baidu/ERNIE-5.5-Flash-Preview)

**2. Strands Decider v2 把判断做成一次前向：31B 六项均分 82.1，校准误差 0.027**
11 月 23 日，AWS 开源 Strands Decider v2：基于 Gemma 4 的 E2B、E4B、12B、26B-A4B、31B 五个模型，Apache 2.0，训练集从 13.6 万条扩到 1748 万条决策样本；它不生成文本，只给每个选项打分并返回概率，31B 在六项文本基准上的均分从 v1 的 70.0 升到 82.1，期望校准误差从 0.062 降到 0.027。官方称亚马逊内部工单系统用 v1 替换 Sonnet 4.6 后成本降 98%、单次调用快 78%，这类数字比均分更值得看；决策模型把输出空间限制在选项上，省掉解码与解析，适合放在代理每一步之前做闸门，难点在于校准而不是分数。
[原文](https://strandsagents.com/blog/introducing-strands-decider-v2/)

**3. ChiProgramBench：432 次试验，代理从引脚重建电路只有 12% 全通过**
11 月 23 日的论文把“只能测引脚、不能看内部”的硬件逆向做成 18 道题：代理可以驱动并读取隐藏的有状态参考电路，最后提交门级网表，隐藏用例逐周期比对，只有全部通过才计算功耗、延迟与面积。6 种代理配置共跑 432 次，黑盒设置只有 12.0% 完全通过，给出功能提示后升到 14.8%，功能得分从 24.6% 升到 30.9%；58 个全通过设计里只有 3 个在三项 PPA 上优于参考，题目覆盖处理器、加速器、编解码器、互连与控制电路，更像真实的硬件调试而不是代码生成。
[原文](https://arxiv.org/abs/2611.31082)

**4. 让 LLM 自己写启发式：157 次提案，分布外路径问题多降 0.24–0.62 个百分点**
同一天的另一篇论文把“学习委派”里的子问题选择交给 LLM：模型提出并修改一段程序，决定把哪些整条路线组成子问题交给固定求解器重优化；一次发现过程在 200 次额度内做了 157 次提案、149 次筛选与 65 次完整评估，选出的策略在验证集平均降本 5.24%。在分布外、500 至 2000 客户的 CVRP-X 上，它用 LKH 修复时比两个 L2D 神经基线多降 0.24 至 0.62 个百分点，且优势在发现时用过的 500 客户规模就已出现；值得注意的是，验证集上排名二三的两个程序在测试中反而更好，说明用固定验证集挑程序仍有运气成分。
[原文](https://arxiv.org/abs/2611.31090)

## 开源与工程

**5. NInfer 11 月 24 日更新：27B 三元模型进 8GB 显卡，978,944 token 上下文**
NInfer-all 在 11 月 24 日的提交里继续压显存：Ternary Bonsai 2 27B 以 2.125 bit 权重在 RTX 5090 上用 DFlash2 跑到 397 token/s，并撑起 978,944 token 的上下文；同一份 8.87GiB 工件在 8GB RTX 3060 Ti 上也能开 24,576 token。Inkling 2 Mini 在 RTX 5090 上解码 182.1 token/s，对照 llama.cpp 的 96.9，官方称在 25 张卡上快 1.4 到 2.8 倍；项目以 Apache 2.0 开放，同时提供 OpenAI、Anthropic 与 llama.cpp 三套 API，对消费级显卡来说，真正的变化是 8GB 卡能跑 27B 的三元量化版，代价是权重格式与内核都绑在这套引擎上。
[原文](https://github.com/iamwavecut/ninfer-all)

**6. NanoClaw 2026.11.0 转稳定：网关升到 1.45，Webhook 默认只听本机**
11 月 20 日，NanoClaw 把 2026.11.0 推上稳定通道：此前跑了 9 个预发布版，这次把 OneCLI 网关升到 1.45.0、CLI 升到 2.11.0，新代理组默认只拿到自己的模型密钥，其余密钥要显式授权；Webhook 服务默认改为只监听本机回环地址，跨机反代需要手动设 WEBHOOK_HOST。它还新增 Cursor 作为代理提供方、支持代理从模板创建子代理，并把本地镜像推到 Claude Code 2.1.321 与 Node 22.23.5，八项需要按清单处理的变更之外，升级风险主要不在模型而在密钥授权与回调地址：默认收紧能减少暴露面，却会让旧部署在升级后静默断线。
[原文](https://github.com/nanocoai/nanoclaw/blob/3307c3e033ef3489705dfa5075553b5452baa7d9/CHANGELOG.md)

**7. GPUWorks v26.11.4：SGLang 长预填充不再被误判死亡，健康检查连败 3 分钟才下线**
11 月 18 日的 v26.11.4 修了一个 PD 分离部署里的典型故障：SGLang 的 /health 过去会走调度器生成 token，长分块预填充时超时，代理在约 13 秒内三次失败就把实例判死，解码端随即报 “Lost connection with prefill instance”。新版本给 SGLang 实例设 SGLANG_ENABLE_HEALTH_ENDPOINT_GENERATION=0，让健康检查只反映进程是否在服务，并把下线条件改成连续失败 3 分钟；容器退出仍然立即处理，升级要先升服务端再升代理，健康检查本该回答“进程还活着吗”，一旦它顺手做业务推理，就会把正常慢路径当成故障。
[原文](https://github.com/flitai-io/gpuworks/blob/8b228a3b0edc3985acbfd1083b54475ef0e14c87/docs-en/releases/v26.11.4.md)

**8. Vardock v0.1.0：把代理的权限交给独立“操作权威”，MIT 开源**
11 月 24 日，VirtualHatch 公开 Vardock v0.1.0：入口只负责对话、不能跑脚本，worker 没有网络与服务令牌，所有外部动作都要由独立的操作权威按所有者预先写好的规则批准，而不是让模型自己判断。每次操作留下带回执的记录，由独立见证进程生成，可用 pnpm vardock verify 复核，需要逐次看脚本时可开 --review-scripts；项目以 MIT 开源，把“人类把多少权限交出去”做成可审计的配置，对需要把代理接进生产系统的团队，审计回执可能比模型能力更重要。
[原文](https://github.com/VirtualHatch-Inc/Vardock)

*本期由站点自动生成 · 信息以原文为准。*
