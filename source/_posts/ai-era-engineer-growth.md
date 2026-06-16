---
title: AI 时代，软件工程师怎么成长？
date: 2026-06-14 20:00:00
tags:
  - AI
  - 软件工程
  - 职业成长
original: true
---

**sudoskys**
GitHub: [github.com/sudoskys](https://github.com/sudoskys)

有一天我翻出了三个月前自己写的一段代码，发现我完全不知道它为什么这样写。

不是因为代码复杂，是因为当时让 AI 生成了它，review 一遍觉得能跑就合并了。现在那段逻辑出了 bug，需要改，但脑子里没有任何关于它内部结构的 mental model。

这是我的代码，但它不是我的。

## 这不是第一次有人担心

每一代工具出现时，都有过同样的忧虑。

1957 年 FORTRAN 出现，汇编程序员认为那是「拐杖」。1998 年 Ellen Ullman 在 Salon 写《编程的愚蠢化》，痛批 Windows 时代的 click-and-drag 工程师「根本不会 debug」。2010 年代 JavaScript 框架大爆发，有人写了《前端失去的十年》，认为框架让工程师失去了对 HTML、CSS 和性能的直接理解。

每次担忧都有真实基础——抽象层提升确实会带来低层知识的流失。但每次也都伴随着整体产出的大幅提升，以及新一批真正掌握底层的人变得更稀缺、更值钱。

AI 这次有什么不同？Stack Overflow 是帮你找到别人写的答案，Copilot 是在你的上下文里直接生成代码。前者是「查字典」，后者是「让 AI 替你思考」。认知参与的程度不同，对技能发展的影响也就不同。

## 真实的数据，两个方向

生产力提升是真实的。MIT、微软、Accenture 联合做过目前规模最大的随机对照实验：4,867 名开发者、三家企业，GitHub Copilot 使 PR 完成量提升 26%。Google 内部 96 人的 RCT 是 +21%。这些数字有方法论支撑，不该被轻易否认。

但 2026 年 6 月 Science 上刚发表的一项研究（Daniotti et al.）给这个故事加了重要的脚注。研究者分析了 16 万开发者的 GitHub 提交记录，用神经网络分类器识别 AI 生成的代码，结论是：senior 工程师用 AI 较少，却拿走了几乎全部的生产力增益。junior 工程师用 AI 最多，生产力收益几乎为零。

AI 在拉大差距，不是缩小差距。

Anthropic 2026 年初的一项 RCT（N=52）在机制上做了解释：让参与者学习一个陌生 Python 库，无限制使用 AI 的那组，知识测验得分比对照组低 17%（50% vs 67%，p=0.01）。速度没有显著提升，但理解显著变差。

同一年，METR 的实验发现让有经验的开源开发者在真实大型项目上使用 AI，反而**慢了 19%**，但他们赛前预测自己会快 20%。那个 39 个百分点的感知偏差比速度数字本身更值得关注——检测退化的传感器本身出了问题。METR 后来坦承，他们设计第二轮实验时，有 30–50% 的开发者拒绝参加「不让用 AI 的对照组」，理由是那已经不是他们正常工作的方式了。

## 团队里正在发生什么

这不只是实验室里的结论。现实中的工程团队在面对同样的问题，有人开始公开记录。

Augment Code（AI 编程工具公司）的 VP of Engineering Vinay Perneti 写道：

> "Verification is the central problem. Every single group named this. Agents can write code — that's not the bottleneck anymore. The bottleneck is they can't verify that their code actually works."

他们真实遇到过的情况是：让 agent 写 100% 代码之后，积压了 **1,400 个 open PR**，中位等待人工评论的时间达到 **1,200 分钟（20 小时）**。代码产出提升了，但审查能力没有同步跟上，形成了一个新的卡口。

他们描述资深工程师的心态时写道：

> "Experienced engineers have spent years building deep expertise in writing code. That's their identity. And now someone is telling them that the thing they've spent their career getting great at is being automated. Even if you frame it as 'elevation,' it can feel like a demotion."

Linear 工程师 Matthijs Wolting 对 AI 工具的使用方式更像是「侦探助手」——先让 agent 帮他关联散落在 Slack、Sentry、日志里的上下文，再自己判断怎么处理。PM Sid Bhargava 描述了一个典型流程：在 Linear 里发现 bug，直接委托 agent 开 PR，一小时内人 review，上线，没有 Stage 2。但 Linear 创始人 Tuomas Artman 也说：「AI assists us in identifying and fixing 10% of our bugs. But let me be clear, I don't rely on it 100%.」

Cursor 工程师 Eric Zakariasson 在 AI Engineer 大会上的表述值得引用：**「对高风险代码区域，每一行都应该由人写，或者至少由一到两个人仔细 review。」** 他补充说：「很容易 vibe code 飞太高，然后就坠毁了。」

这些观察汇成了一个一致的轮廓：AI 工具加速了代码生成，但理解、验证、判断的责任没有转移，反而更重要了。

## 为什么 junior 受损最大

Anthropic 的实验把参与者的 AI 使用行为分成了六种模式。

让技能退化的三种：直接让 AI 写，review 一遍就合并；遇到不理解的地方继续让 AI 解释，不自己推导；反复 prompt 直到跑通，但从来不问「为什么」。

保留学习效果的三种：先自己想，再用 AI 验证或攻击自己的判断；自己完成核心逻辑，AI 处理样板；AI 生成后追问 why，真正理解了再提交。

追问型的测验得分是外包型的两倍左右。差距不在于用没用 AI，在于用 AI 的姿态。

Senior engineer Muhammad Haseeb Sohail 在做了 30 天的 vibe coding 实验后这样写：

> "Those gains required me to actually know what I was doing. Vibe coding for a senior engineer is different from vibe coding for a junior engineer. The senior has judgment that makes the process safe. The junior doesn't have that yet."

这把两种知识的差异说得很清楚。一种知识可以通过语言传递：语法、API 用法、设计模式的名字，AI 瞬间给你，成本接近零。另一种只能通过亲身承担后果获得：为什么这个索引在测试环境有用但上了生产就不行了？为什么那个在技术上更优雅的方案在这个团队里根本落地不了？这类知识无法被告知，你得自己走过那段路，踩那个坑。

Kent Beck 说：「AI 是一个无限耐心的导师，但它不太会主动创造学习机会。」它有问必答，但它不会故意让你卡住——而卡住才是学习真正发生的时候。

Engineering manager Brian Meeker 对 junior 的处境说得更直接：

> "Our industry is quickly pulling up the ladder on junior engineers. We are taking away the kind of work that you need to learn. You need reps in the kind of toil that AI excels at automating away."

## 成长路径

**筑基期**

代码自己写，AI 只回答概念性问题，不让它生成代码。这个阶段在建立判断力的地基。没有这个地基，后面的 AI 辅助只是在生产你看不懂的代码。判断标准只有一个：关掉 AI，能从空文件写出一个完整模块。

Vercel VP of Engineering Lindsey Simon 的面试政策是：「我们告诉候选人坦诚说明用了什么 AI 以及为什么。如果你让 AI vibe code 了整道题，你能向我解释它是怎么工作的吗？这才是真正重要的。」这个问题在筑基期之前回答不了。

**协作期**

AI 可以写样板代码、测试、文档，但涉及架构判断的东西——选型、抽象边界、数据流设计——必须人先做决定，AI 只负责填充。

一个简单但有效的约束：**不提交任何自己解释不了的代码**。你可以用 AI，但你得能在一句话里说清楚它为什么这样写。说不清楚，不合并。这是为自己保留代码所有权的最低门槛。

一个台湾 tech lead 把 PR 分为两类：leaf node（可以用 AI 自由改）和 core（必须人工审核或人工写）。Module Owner 对 AI 生成的 PR 有否决权。这个粒度的区分在实践上比「禁止/允许用 AI」更可操作。

**全杠杆期**

AI 做大部分实现，人的精力从「怎么写」转向「该不该这样写」——tradeoff 判断、失败模式预期、可维护性评估。

这是一次能力重心的转移：从写代码，变成读代码和判断代码。Daniotti et al. 的数据里 senior 工程师收益高、junior 收益低，根本原因就在这里——senior 的价值已经在判断层，AI 提升的是他们的执行速度；junior 的价值还在执行层，AI 直接替代了他们正在建立的东西。

**架构期**

不再 review 每一行代码，而是设计 linter、类型系统、schema 约束、自动化检查——让 AI 在不违反架构原则的前提下自由生成。微软 CTO Russinovich 把这个转变描述为：从 reviewer 变成 legislator（立法者）。

但这里有一个陷阱：如果没有走过前三个阶段，这个阶段的「判断」是空的。没有积累的架构师，指令质量不会高。

## 不靠自觉的机制

意识层面的原则在交付压力下很难长期执行。好的机制像 linter，流程本身强制执行，不依赖人记着要去做。

**PR 理解声明。** 在 PR 模板里加几个必填问题，未填就不能 merge：

```markdown
## Comprehension Gate
- [ ] 我能不借助 AI 解释此 PR 的每一行代码
- [ ] 这段代码为什么这样写（不是「做了什么」，是「为什么」）：___
- [ ] 如果这段代码 break，我知道去哪里找原因：___
```

Sankaranarayanan 2026 年的实验（N=78，arXiv:2602.20206）给这个做法提供了数据支撑：加入这个检查点后，「断开 AI 30 分钟后维护自己刚写的代码」的失败率从 77% 降到 39%，代价约 14 分钟/次。Augment Code 把类似的机制叫做「spec review」，认为它正在变得比 code review 更重要。

**无 AI 定期测试。** 每两周一次，从当前正在做的真实工作里挑一个 task，关掉 AI，独立完成。完成后写三行：做了什么、卡在哪里、什么东西以前靠 AI 现在自己不会了。用真实 task 而不是 LeetCode，因为要测的是对自己项目的掌控力。Brian Meeker 把这一点写进了团队政策：「You must be able to do your job if your AI tooling disappears.」

**决策日志。** 在仓库里建一个 `decisions/` 目录，每次做技术选型，写 3–5 行：选了什么、为什么、放弃了哪个替代方案、基于什么假设。AI 可以帮你理解代码写了什么，但无法告诉你当初为什么这样设计。Storey（ACM Queue 2026）把这个空缺叫做「意图债务」（Intent Debt）——代码和理解可以事后恢复，意图一旦没记录就消失了。

## 市场已经在说话

Stanford 数字经济实验室（Brynjolfsson、Chandar、Chen，2025）基于美国最大薪资软件 ADP 的行政数据，发现 22–25 岁软件开发者的就业相对更有经验的工作者出现了约 16% 的下降。职位市场出现极化：AI/ML 工程师岗位在涨（+59%），通才 SWE 岗位在跌（-49%）。拥有 2 项以上 AI 相关技能的工程师有 43% 的薪资溢价。

这些数据的因果关系还不清晰——科技业周期和利率也在其中——但方向一致：会用 AI 做判断的人在升值，只会用 AI 生成代码的人在贬值。

Shopify 的回应值得参考。他们把实习生规模从每期 25 人扩大到 1,000 人，理由是「这一代人是 AI-reflexive 的，是改变工程文化最好的媒介」。他们的招聘框架分三类：禁止 AI（测基础理解）、AI 可选（测判断力）、AI 必须（测工具约束下的完成力），三类并行，不是非此即彼。

中国公司的数字则展示了另一面：腾讯报告 94% 的代码评审已有 AI 参与，50% 的新增代码由 AI 辅助生成；字节跳动 80%+ 工程师在使用 TRAE。但腾讯云副总裁吴运声坦承：「对个人而言提效很明显，但组织上需不需要做配套工作，我们确实还在思考。」个人层面的提效和组织层面的协调，还没有完整对齐。

如果成为 senior 工程师需要走过那条路——从头写、搞砸、修好、真正理解——而 junior 岗位正在消失，那十年后的 senior 从哪来？

这是一个我没有答案的问题。Kelsey Hightower 担忧 AI 会「切断产生下一代工程师的 pipeline」。Kent Beck 说：「managing juniors for learning, not production」。这些更像是呼吁，而不是解法。

我只能说一件相对确定的事：**用 AI 辅助判断，和用 AI 替代判断，是两条不同的路，走到最后会到完全不同的地方。**

区别就在那个「为什么」，你有没有自己想过。

*参考资料：*

- *Daniotti et al. (2026). "Who is using AI to code?" Science. https://www.science.org/doi/10.1126/science.adz9311*
- *Shen & Tamkin (2026). "How AI Impacts Skill Formation." arXiv:2601.20245.*
- *Sankaranarayanan (2026). "Mitigating Epistemic Debt in AI-Scaffolded Programming." arXiv:2602.20206.*
- *METR (2025). "Measuring the Impact of Early-2025 AI on Experienced Developer Productivity." https://evals.alignment.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/*
- *Brynjolfsson et al. (2025). "Canaries in the Coal Mine." Stanford Digital Economy Lab.*
- *Storey, M-A (2026). "From Technical Debt to Cognitive and Intent Debt." ACM Queue. https://queue.acm.org/detail.cfm?id=3807966*
- *Perneti, V. (2026). "The Hardest Part About Going AI-Native." Augment Code. https://www.augmentcode.com/blog/hardest-part-about-going-ai-native*
- *Meeker, B. (2026). "Have a Coherent AI Policy." https://brianmeeker.me/2026/05/14/have-a-coherent-ai-policy/*
- *Beck, K. (2025). "The Bet on Juniors Just Got Better." https://tidyfirst.substack.com/p/the-bet-on-juniors-just-got-better*
- *Willison, S. (2025). "Not 10x." https://simonwillison.net/2025/Aug/6/not-10x/*
- *Linear (2026). "How We Use Linear Agent at Linear." https://linear.app/now/how-we-use-linear-agent-at-linear*
