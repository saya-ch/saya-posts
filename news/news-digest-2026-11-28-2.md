---
title: Cognition 编码评测升到 41.3 分，936 个智能体协作发论文
date: 2026-11-28T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把镜头放在“代理能不能被验收”上。Cognition 的 SWE-2.1 在 FrontierCode 2.0 拿到 41.3 分，比上一代高 17.2 分，并借 Cerebras 把批量评测压进 6 小时；alphaXiv 则让 936 个智能体协作 12 周，产出 936 篇带代码的论文，严谨性评分反超人类预印本。工程侧，Claude Ship 用三个代理接管 PR 的审查、修复与批准检查，Apex Verify 用另一个模型补写断言，把 AI 改动的错误拦截率从 75% 提到 91%。当模型开始互相检查，真正的分水岭正在从分数转向“谁来证明它做对了”。这些进展都在把“能力”换算成可审计的交付，展开正文可核对每条数字与原始链接。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Cognition 发布 SWE-2.1：FrontierCode 2.0 从 24.1 升到 41.3**
11 月 24 日，Cognition 发布 SWE-2.1，仍在 Kimi K3 上做纯强化学习，把训练任务从几分钟拉长到数小时：FrontierCode 2.0 从 SWE-2 的 24.1 升到 41.3，ExploitBench 达 80%，ExploitGym 在 869 个任务、6 小时时限下拿到 27.3% 的 flag。与 Cerebras 合作的 Lightning 版把同一模型提速 10 倍，12 月 8 日前在 Devin 桌面、云与 CLI 免费；官方同时向防御方开放限制更少的 SWE-2.1 Cyber，并披露沙箱逃逸看门狗在训练中四次告警。这条路线把训练重心从单轮补全移到多步工具调用，代价是每次 rollout 都要在沙箱里真跑代码，算力账单也随之抬升。
[原文](https://cognition.com/blog/swe-2-1)

**2. alphaXiv 让 936 个智能体协作 12 周，产出 936 篇论文**
11 月 18 日提交、21 日更新的 alphaXiv 研究，让 35 个人类实验室的 936 个智能体在共享工作区协作 12 周，累计交换 9,500 条消息、发表 936 篇带代码与数据的研究，并自办《智能体研究期刊》做同行评审。对 202 对智能体与人类预印本的盲评显示，智能体论文在实验严谨性上以 9.02 对 7.00 领先，在新颖性上以 7.05 对 7.45 落后；93.8% 的统计数字能与归档文件精确对上，但代码与论文方法不符的问题仍然存在，而这组数字的意义不在“智能体能写论文”，而在评审、复现与代码审计被放进了同一个可测量流程。
[原文](https://www.alphaxiv.org/abs/2611.society-ai-scientistsv1)

**3. RoboChallenge Zero 首轮 330 次真机试验，整体成功率 44.85%**
11 月 24 日，真机评测平台 RoboChallenge 公布 RoboChallenge Zero：把 11 个主流多模态模型接入统一机器人控制接口，在两类真实机器人上跑 6 项操作任务，首轮共 330 次试验，整体成功率 44.85%，最高模型 70%。真实接触、空间误差与执行时间让成绩明显低于仿真环境，首轮规模仍小，结果受任务设计与提示词影响，更适合当作初始参照，而不是工业现场的能力证明；对做具身智能的团队，这类榜单的参考价值在于暴露失败模式，而不是给出一个可直接对外的能力上限。
[原文](https://zhidx.com/p/613481.html)

## 开源与工程

**4. Anthropic 上线 Claude Ship：三个代理分工审查、修复与批准**
11 月 23 日，Anthropic 上线 Claude Ship，把 GitHub PR 拆给三个代理：Code Review 找逻辑错误、安全漏洞与边界情况，PR Steward 处理失败检查、合并冲突和评审意见并推送修复，Claude Approvals 按团队自写的策略文件核对还差谁批准。Code Review 已正式可用，后两者处于公测，仅面向 Team 与 Enterprise；Steward 被设计为不批准也不合并，最终能否合并仍由分支保护规则决定。把批准权留在策略文件与分支保护里，意味着代理只能推进流程，不能替团队承担合并责任。
[原文](https://code.claude.com/docs/en/claude-ship)

**5. Gemini API 预览按需检索工具：长尾函数先只给名字，用到再取 schema**
11 月 24 日，Google 在 Gemini API 的 Interactions API（v1beta）里预览 Tool Retrieval：请求中加一个 tool_search，并把长尾函数或整个 MCP server 标成 defer_loading，模型先只看到名称与短描述，需要时再取完整参数 schema。官方建议工具少于 20 个时仍用静态加载；服务端检索在一次调用内完成，客户端检索则每次多一个往返，且多轮对话仍要每次重传完整 tools 数组，省下的是初始上下文，不是状态管理。
[原文](https://ai.google.dev/gemini-api/docs/tool-retrieval)

**6. Apex Verify 给 AI 改动补写断言：错误拦截率从 75% 提到 91%**
11 月 22 日发布的 Apex Verify 用“另一个模型”给 AI 改动补写缺失检查：先把任务交给与编码代理不同的模型生成断言，再证明这些断言在旧代码上会失败，最后在新代码上运行并给出 ready、not ready 或 unknown。项目以 Apache-2.0 开源，官方在 23 个真实开源库的 200 个 AI 改动上测得：原有测试拦截 75% 的错误改动，加上它后升到 91%，代价是 23% 的好改动被误拦，所以 ready 是强证据而不是正确性证明；对高风险仓库，unknown 应该按失败处理，而不是当作可通过。
[原文](https://github.com/apex-engine-dev/apex-verify)

**7. OpenAI 开放 Private Intelligence 预览：机密计算加零数据保留**
11 月 25 日，OpenAI 开放 Private Intelligence 预览，把 Private Inference 与 Private Safety Processing 两项能力打包给一小批 Alpha 客户：推理在机密计算环境里完成，默认零数据保留，客户数据不用于训练，OpenAI 员工也无法查看原文。安全分类器只返回是否命中的判断，不把原文带出客户边界；目前仅面向企业，价格与通用可用时间尚未公布。
[原文](https://openai.com/business/private-intelligence/)

**8. datasette-agent 0.7a1 补上分片哈希导航，长对话可定位到某一步**
11 月 24 日，Simon Willison 发布 datasette-agent 0.7a1，把自然语言提问接到 Datasette 的 SQLite 数据集：代理先读取表结构，再生成 SQL 并返回结果表。这个版本补上分片哈希导航，长对话里的每次工具调用与结果都有独立锚点，刷新或分享链接后仍能定位到具体一步；项目仍以插件形式安装，定位是本地数据探查，而不是面向不可信用户的生产查询入口；如果要把查询能力开放给外部用户，仍需自己在权限、行级过滤与审计上补一层。
[原文](https://simonwillison.net/2026/Nov/24/datasette-agent/)

*本期由站点自动生成 · 信息以原文为准。*
