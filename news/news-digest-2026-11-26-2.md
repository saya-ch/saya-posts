---
title: 27B 三值模型压到 5.9GB，480B 视频模型登顶 AA 榜
date: 2026-11-26T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头放在模型怎样被压小、又怎样被验收上。PrismML 把 27B 模型的三值权重压到 5.93GB，五项工具调用均分 77.2，几乎追平 FP16 的 77.7；Sand.ai 的 MAGI-2 用 480B 参数一次生成 30 秒 1080P 视频与音频，登上 Artificial Analysis 榜首，定价每秒 0.5 元。研究侧，GreenDynaMix 用温度感知的混合精度把 GPU 老化速率压低 25% 到 33%，另一篇论文则把代理的策略输入砍掉七成。工程侧，谷歌开源 86 套评测的 gBench，Transformers 把缓存接口统一成 cache，pydantic-ai 一次修掉六个中危问题。想看每条的数字边界与原始出处，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Sand.ai 发布 MAGI-2：480B 参数，一次生成 30 秒 1080P 视频与音频**
11 月 25 日发布的 MAGI-2 总参数 480B，支持单次 4 至 30 秒、1080P 加音频的原生生成，在 Artificial Analysis 榜单登顶全球第一；定价每秒 0.5 元，一条 30 秒成片约 15 元，单次最多可混输 10 张图片、5 段视频和 5 段音频，既能一镜到底也能多镜头推进；相比预览版的 114B，这一代把总参数扩到 480B、激活约 28B，训练数据规模扩大约 3 倍，并已完成海光、摩尔线程等国产芯片适配，官方称将尽快开放权重。
[原文](https://www.cnstock.com/commonDetail/833387)

**2. Ternary Bonsai 2.5 27B：三值权重 5.93GB，工具调用追平 FP16**
PrismML 把全部语言模型权重压成 {−1, 0, +1}，每 128 个权重配一个 FP16 scale，整模只有 5.93GB、1.76 bit/权重，而 FP16 版是 53.8GB；在 262K 上下文下 SWE-bench Verified 拿到 75.6（FP16 80.6），五项工具调用均分 77.2（FP16 77.7），MTP 草稿头让 NVIDIA 显卡解码提速 1.5 至 1.8 倍，Apache 2.0 许可。
[原文](https://docs.prismml.com/models/bonsai-2-5-27b)

**3. GreenDynaMix：用温度感知的混合精度，把 GPU 老化速率压低三成**
固定频率缩放只能在吞吐与精度之间二选一，这正是论文要解的耦合。作者把基于 DVFS 的温度管理器与动态精度调节器接在同一套运行时里，用 MoE 路由器给出的专家重要性信号决定哪些专家保留更高位宽；在服务器、桌面与嵌入式平台上，推理期由温度导致的老化速率下降 25% 到 33%，8/4 bit 混合精度恢复吞吐的困惑度代价为 0.9% 到 1.6%，显存逼到 4/2 bit 时升到 9.4%，且不需要离线 Profiling 或重训。
[原文](https://arxiv.org/abs/2611.31737)

**4. 给代理状态做减法：策略输入最多减少 71%，成功率仍贴近参考**
论文把代理状态定义为交互历史的紧凑抽象，用 LLM 引导的遗传算法搜索确定性递归程序，并以任务成功率而非行为匹配来判断抽象是否充分，因为要求黑盒策略逐步复刻参考行为既苛刻又难验证；在文本家务环境 ALFWorld 与科学实验环境 ScienceWorld 上，学到的抽象保持接近参考代理的成功率，同时把平均策略输入分别减少最多 71% 与 57%，程序本身可直接检查，能看清哪些信息被保留、更新和丢弃。
[原文](https://arxiv.org/abs/2611.31800)

**5. VQACC：不改加速器，把码本查找塞进操作数寻址**
此前的 VQ 方案要么增加专用硬件，要么只覆盖有限推理场景。论文认为现有脉动阵列加速器已具备所需的存储与端口，缺的是复用机制，于是重写操作数缓冲的访问逻辑与数据映射，让反量化跟着正常寻址数据流走；相对 EVA 基线取得 2.63 倍加速，新增硬件只占计算阵列面积的 0.17%，不必再为 VQ 单独设计加速器。
[原文](https://arxiv.org/abs/2611.27561)

## 开源与工程

**6. 谷歌开源 gBench：86 套评测，外加代理质量与压力测试**
gBench 默认提供 86 个评测套件，分属通用知识、数学证明、代码与算法、长上下文、工具调用、多模态六个支柱，另有 16 项 Golden Set 冒烟检查和 60 条 GemmaClaw 代理质量场景，分成学术评测、冒烟检查与代理质量三层；模型发现会读 HuggingFace 的 config.json，按参数量给出 1 卡、2 卡或 8 卡的 GPU 档位建议；它可评测任意 OpenAI 兼容的 /v1 端点（已在 vLLM 与 Ollama 验证），压力测试用不少于 1099 个请求把 P99 上限标到 ±9%，Apache 2.0 许可。
[原文](https://github.com/google-deepmind/gbench)

**7. Transformers 统一缓存接口，导出模型比 torch.compile 快 1.4 倍**
混合模型此前要在 past_key_values 与 cache_params 两套命名之间来回切换，这是统一接口的直接动因。5.24.1 于 11 月 24 日发布，修掉 vLLM 等远程代码对旧缓存名的依赖；5.24.0 起 past_key_values 与 cache_params 统一成 cache（forward 的旧名保留到 5.40.0），导出统一返回 ExportedModel：ONNX Runtime 配 TensorRT EP 在 EmbeddingGemma2 上比 eager 快 7.6 倍、比 torch.compile 快 1.4 倍，OpenVINO 解码快 2.4 到 3.9 倍。
[原文](https://github.com/huggingface/transformers/releases)

**8. pydantic-ai 一次修掉六个中危问题，浏览器出口是重灾区**
2.74.0 于 11 月 24 日修复的问题里，四个出在 PlaywrightBrowser 的私网拦截：漏掉 IPv6 site-local、NAT64 内嵌的云元数据地址、带反斜杠的无主机 URL，以及 WebRTC、WebSocketStream、FedCM 旁路；另有一条 OpenTelemetry baggage 污染评测结果，一条 ElevenLabs 实时会话把 secret__ 动态变量写进 trace。六个问题都定为中危，影响面集中在会打开不可信页面的代理；修复随 harness 0.74.0 发布，次日的 2.75.0 再把低权限工作区文件移出系统指令，避免工作区文本被当成操作者指令。
[原文](https://github.com/pydantic/pydantic-ai/releases)

**9. Flower 1.45.0：AgentApp 复用压缩上下文，差分隐私只动浮点数组**
11 月 25 日发布的 1.45.0 由 9 位贡献者完成，让 Agent 运行更易追踪，AgentApp 可以跨运行复用压缩后的上下文，并更灵活地使用 Space；差分隐私的范数裁剪与加噪现在只作用于浮点数组，BatchNorm 的整数缓冲不会被误改，此前这类状态会触发类型转换错误。上一版 1.44.0 已把预热 worker 的任务交接中位数从 480ms 降到 101ms。
[原文](https://flower.ai/docs/framework/main/en/changelog/index.html)

*本期由站点自动生成 · 信息以原文为准。*
