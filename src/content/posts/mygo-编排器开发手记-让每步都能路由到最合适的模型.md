---
title: MyGO 编排器开发手记：让每步都能路由到最合适的模型
description: 让轻模型干轻活，让重模型干重活，让智能体干他最该干的活，让每一步路由到最合适的模型
published: 2026-08-22
category: 技术
tags:
  - 抽象
  - 创新
  - 计算机
  - 折腾
slug: dsh-my-go
zhihu-title: MyGO 编排器开发手记：让每步都能路由到最合适的模型
zhihu-topics:
  - deepseek-harness
  - 国产大模型DeepSeek
  - AI技术
  - 人工智能
  - Agent
  - AI-Agent
  - AI开源工具
  - AI开源项目
zhihu-link: https://zhuanlan.zhihu.com/p/2074523751576818726
---
<iframe src="//player.bilibili.com/player.html?isOutside=true&aid=117134998964032&bvid=BV1Jm866tEU7&cid=41141274335&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"></iframe>

DeepSeek 涨价太哈人了，现在一两天就能花以前半个月的钱。在涨价的前两天我买了 OpenCode Go，然后就被背刺了：

| 模型                                      | 输入    | 输出    | 缓存读取   | 缓存写入 | 使用额度 |
| --------------------------------------- | ----- | ----- | ------ | ---- | ---- |
| DeepSeek V4 Pro (Off-Peak)              | $0.66 | $1.98 | $0.022 | -    | $15  |
| DeepSeek V4 Pro (Peak)                  | $1.32 | $3.96 | $0.044 | -    | $15  |
| DeepSeek V4 Flash (Off-Peak)            | $0.22 | $0.66 | $0.007 | -    | $30  |
| DeepSeek V4 Flash (Peak)                | $0.44 | $1.32 | $0.014 | -    | $30  |
| DeepSeek V4 Flash Vision Exp (Off-Peak) | $0.22 | $0.66 | $0.007 | -    | $15  |
| DeepSeek V4 Flash Vision Exp (Peak)     | $0.44 | $1.32 | $0.014 | -    | $15  |

使用额度从 60 USD 直降到 15 USD

~~虽然我买完就开始和朋友一起用 Pro 蹬回本了~~

四天时间，花了这么多：

![每月用量 83%，大约相当于 50$](https://image-hk-1.oss-accelerate.aliyuncs.com/20260822134732702.png)

实话讲，这额度消耗还是很吓人，这下去谁敢用。

所以我参考 oh-my-openagent 开发了 dsh-my-go，意思是：

> **My** tasks, where to **GO** ?????

把合适的任务，派给合适的模型，以最大限度节省花费和上下文，并提高质量。

::github{repo="daizihan233/dsh-my-go"}

一行命令就能安装：

```bash
dsh plugin --profile web add dsh-my-go@latest --config.minimumReleaseAge=0
```

安装后需要配置模型，模型配置建议移步 README

![设置页截图](https://image-hk-1.oss-accelerate.aliyuncs.com/PixPin_2026-08-22_02-23-23.png)

保存设置后在新会话中选择 “MyGO!!!!! 模式” 即可使用：

![新会话截图](https://image-hk-1.oss-accelerate.aliyuncs.com/PixPin_2026-08-22_02-24-35.png)

## 闲聊

我很早之前就开始使用 opencode 了，后来知道了 oh-my-opencode，现在好像叫 oh-my-openagent，因为他们现在还支持了 Codex。

::github{repo="code-yeongyu/oh-my-openagent"}

我非常喜欢他们的愿景：

> [!QUOTE] OMO 项目 README 中文版本节选
> 你不需要为 2 小时的工作付 200 美元。 未来不是选一个赢家，而是把所有赢家编排到一起。模型每个月都在变便宜、变聪明。没有任何一个供应商能够独占。我们是在为那个开放的市场而构建，不是为他们的围墙花园。

从 DeepSeek 打响模型价格战第一枪开始，Xiaomi MiMo，到 GPT 5.6 Luna，模型一直在发展，而且发展得很快。然而写代码是一个复杂任务，生成模板代码、写 README、读代码结构 这些杂活让现在的大模型来干，结果和以前的小模型是一样的，不会有什么实质性的区别，甚至一些有着明确步骤和指令的活，哪怕是已经退出当下第一梯队的模型也有不错的表现。至于 诡异 Bug 定位与调试、跨模块分析、理解与规划 这类需要复杂推理的任务，小模型的能力显然是不够的。

**所以让一个模型干完所有活注定是不现实的。**

DeepSeek Harness 有他自己特色的 **“创造模式”**，能在 DSH 里开发插件并且热重载给自己用。所以这个插件其实也正是我自己三天 Vibe 出来的。当然我并不是说 DSH 已经强到普通人一句话就能开发一个复杂插件了，我学过软件开发，很多 Bug 也是我自己定位完然后让 AI 修的，**AI 自己还是修不明白 Bug。**

我是参考了 OMO 的设计，但 OMO 的设计确实复杂，感觉他想要一个插件顶一堆，但是我个人觉得步子一下迈太大不好，而且也不太符合 DSH 一切皆插件的设计哲学。一个插件理应就负责一件事，在程序设计里也是如此。

**所以在 MyGO 里我简化了设计，只保留了 Agent 编排这一核心功能。**

MyGO 的 Agent 编排设计是参考了 OMO 的 **自律军团 (Discipline Agents)**

> [!QUOTE] OMO 项目 README 中文版本节选
> Sisyphus 负责调度 Hephaestus、Oracle、Librarian 和 Explore。一支完整的 AI 开发团队并行工作。

MyGO 同样由 Sisyphus 负责调度，由其他智能体去完成具体的任务，整体上是一个**星形拓扑**。

很多的 Agent 编排器会选择 Workflow 流或者网状拓扑，就是让智能体们能够互相交互会话，其实我觉得这样并不好，因为会造成上下文混乱。把任务拆分交给 SubAgent 的目的就是为了让一个 Agent 只专注于一个任务，网状结构的话似乎背离了这个目标。

坦率地说，不论是 Workflow 流还是 OMO 并行执行，都能极大提升智能体工作的效率。但是 MyGO 项目里，我在设计时还是选择了开倒车，选择了**单线串行执行**。一方面，这样不会污染 Sisyphus 的上下文，不会在工作的时候突然打断插进来一个 Agent 的工作报告。更重要的是，我在使用 OMO 的时候经常遇到一个问题，就是我不知道 SubAgent 在干啥。固然，我可以点开看 SubAgent 在思考什么，但是我不能干预，而且我也盯不过来那么多的智能体。

得益于 DSH 自身的特性，你可以在子智能体里**插话发送**，这样你就可以和主智能体一起盯着子智能体干活。星形拓扑让主智能体可以掌控全局，最终验收；单线串行让你有精力盯着一个智能体干活和思考，早期修正。

这固然会一定程度上降低效率，但一方面在你没装编排插件之前模型基本也是 “一刀流”，从不创建 SubAgent。如果你能忍受没装编排插件之前 DSH 的干活效率，那单线串行对你来说其实没啥区别。何况，你不想 Review AI 写的代码，连 AI 思考过程都不愿意看，你敢上线吗？

除此之外，MyGO 对智能体这块做了一点优化，加了一个 Hermes 智能体。顺带一提的是，这里 Hermes 其实无意碰瓷一个叫 Hermes 的 Harness，只是为了迎合其他智能体的命名。MyGO 编排器的智能体命名也参考了 OMO，使用希腊神话人物标明作用和人设。

| Agent 命名          | 原型故事                                | 在 MyGO 中                                     |
| ----------------- | ----------------------------------- | -------------------------------------------- |
| Sisyphus（西西弗斯）    | 被诸神惩罚，永无止境地把巨石推上山顶，石头滚下来再推上去，循环往复。  | 分发任务、接收结果、驳回重做、再分发……直到任务最终通过。                |
| Prometheus（普罗米修斯） | 盗火者，把天火带给人类，赋予人类智慧和文明。              | 负责理解用户模糊的需求，把它 “点亮” 成清晰可执行的计划。是 “从无到有” 的创造者。 |
| Oracle（奥拉克尔）      | 德尔斐神庙的女祭司，能预言未来，给出的神谕晦涩但准确。         | 跨模块的循环依赖，修不好的诡异 Bug，Oracle 负责深度思考，精准判断。      |
| Hephaestus（赫菲斯托斯） | 火神与工匠之神，奥林匹斯山上最擅长锻造武器和工具的工匠。        | 不负责想，不负责看，只负责 “造东西”，也就是写代码。                  |
| Hermes（赫尔墨斯）      | 众神的信使，跑得最快，负责传递消息和执行宙斯的命令，从不质疑命令本身。 | 负责 “批量替换、格式化、修改文档排版” 这类不怎么需要动脑子的体力活          |

MyGO 的设计中把 “编写代码” 这个任务进一步细化，模板和简单代码由 Hermes 负责，稍难的用 Hephaestus，最难的用 Oracle，这样可以进一步节省成本。

就这样，你可以把不同难度的任务分配给不同的模型去完成。

值得一提的是，DeepSeek Harness 目前并不支持动态指定子代理模型。

> [!QUOTE] 《DeepSeek Harness 白皮书》9.2 子代理（subagent）：并行干活
> **已知限制（社区反馈）**：当前**不能动态指定子代理模型**——子代理模型跟随父 Agent 配置，社区已在讨论区反馈（[#118](https://github.com/deepseek-ai/deepseek-harness/discussions/118) 评论区，对比 Claude Code"可指定子代理模型"的体验）。需要不同模型能力时，目前只能整体切换父 Agent 的模型配置，或拆成独立会话分别指定。

dsh-my-go 使用的变通方法是使用 `agent/request` waterfall 这一扩展点，在每次发送请求前拦截并修改配置，以达到动态指定子智能体模型的目的。

最后你可能好奇为啥要起 MyGO 这么个名字，但全文和 MyGO 动画没啥关系。因为我最开始本来想参考 oh-my-openagent 叫 oh-my-dsh，但发现已经有叫这个的仓库了，是一些收集实用插件的项目，而且 DSH 的插件大多以 “dsh-” 开头。然后我本来随意一点叫 dsh-my-oh 的，然后可能打到 my 的时候 ~~触发关键词了~~ ，不小心打成了 dsh-my-go，索性就叫这个了。

总之，项目仍在积极开发与迭代，欢迎试用与 Issue，**希望能帮你省一些花费。**

## 致谢

Vibe 时使用的三位开发者：（排名不分先后）

 - DeepSeek V4 Flash 0731
 - DeepSeek V4 Pro 0813
 - Xiaomi MiMo V2.5

本项目开发时参考了由社区编写的《DeepSeek Harness 白皮书》

::github{repo="Electricitysheep/dsh-handbook"}