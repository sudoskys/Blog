---
title: 叙事物理学：用守恒定律和熵建模交互式叙事的动力学
date: 2026-02-06 18:00:00
tags:
  - AI
  - Roleplay
  - 叙事设计
mermaid: true
original: true
---

**sudoskys**
GitHub: [github.com/sudoskys](https://github.com/sudoskys)

*Correspondence: sudoskys@github*

---

## 摘要

大语言模型（LLM）驱动的交互叙事在长会话中面临两个结构性问题：**世界状态漂移**（角色遗忘已知事实、空间位置矛盾、因果链断裂）与**叙事节奏退化**（张力单调递减或混乱振荡、伏笔无回收、情节碎片化）。现有工程方案——包括世界书（World Book）注入、摘要压缩、检索增强生成（RAG）——在经验上缓解了部分症状，但缺乏统一的理论框架来解释为什么某些方案有效、在什么条件下失效、以及最优策略的理论极限在哪里。

本文提出 **Narrative Physics**（叙事物理学）框架，将叙事状态形式化为有向属性图 $G=(V,E,\phi)$，在其上定义六种粒子类型和五类力作为本体论基础，建立五条守恒定律作为一致性不变量，引入压力函数族 $\mathcal{P}=\{P_\tau,P_\sigma,P_\gamma,P_\rho,P_\theta\}$ 刻画叙事演化的微观驱动力，并通过叙事熵 $H(G)$ 度量宏观叙事状态。我们进一步利用叙事图的 Laplacian 谱分析建立压力传导速率的理论上界（定理 3），证明契诃夫守恒的一般判定问题与停机问题等价（定理 2），并提出叙事相变假说（Conjecture 3）以解释长叙事中的一致性崩塌现象。全文共提出 6 条猜想，涵盖 Le Chatelier 稳定性、谱间隙-聚合度关联、相变临界指数、熵产率界、契诃夫阈值最优性与 Lyapunov 收敛性，为交互叙事的形式化理论提供初步框架。

**关键词**：交互叙事，叙事生成，图论，谱分析，耗散结构，大语言模型

## 1. 引言

### 1.1 问题动机

大语言模型本质上是条件概率分布 $p(x_t \mid x_{<t})$，不维护显式的世界状态表示。设上下文窗口容量为 $W$ tokens，叙事世界状态的 Kolmogorov 复杂度为 $K(S)$——直观地说，$K(S)$ 是完整描述当前世界状态所需的最短程序长度，包括所有角色的位置、关系、已知信息、悬而未决的伏笔、以及因果历史。

当 $K(S) > W$ 时，状态漂移不可避免——某些世界事实必须被截断或有损压缩。更严格地说，设世界状态可表示为 $m$ 个原子事实的集合 $S = \{s_1, \ldots, s_m\}$，压缩映射 $\pi: S \to \hat{S}$ 将其编码进容量为 $W$ 的上下文窗口，失真度 $d(S, \hat{S}) = |S \triangle \hat{S}|$（被错误添加或遗漏的事实数量）。由 rate-distortion 理论，当原子事实近似独立且等概时：

$$\mathbb{E}[d(S,\hat{S})] \geq m - W / \bar{l} \tag{1}$$

其中 $\bar{l}$ 为单个事实的平均编码长度（tokens）。直觉：$W / \bar{l}$ 是窗口能忠实容纳的事实数上限，剩余 $m - W/\bar{l}$ 个事实必然被有损处理。

这意味着状态漂移不是实现缺陷，而是**信息论必然**。任何基于有限窗口的方案——无论是摘要压缩、世界书注入还是 RAG 检索——都无法将 $d$ 压至零（当 $m > W/\bar{l}$ 时）。这一观察构成了本文的出发点：既然完美保真不可能，我们需要一种框架来判断**哪些事实最重要**、**什么时候遗忘是可接受的**、以及**系统应如何自动恢复一致性**。

### 1.2 现有方案的失败模式

在进入形式化之前，我们先观察 LLM 交互叙事中反复出现的结构性失败模式。这些模式不依赖于特定实现，而是源于缺乏显式状态管理的根本局限：

**失败模式 A：角色遗忘**（Character Amnesia）。在超过 30–50 个回合的长会话中，LLM 经常「遗忘」早期建立的角色特征或关系。例如，一个在第 5 回合被确立为「恐高」的角色，可能在第 40 回合毫无困难地攀爬悬崖。这不是 LLM 的「智力」问题，而是信息论问题：该角色特征在 30+ 回合前被提及，早已滑出有效注意力窗口。

**失败模式 B：空间传送**（Spatial Teleportation）。角色在未经过移动描写的情况下突然出现在不同地点。这违反了我们将在 §4 定义的存在守恒（定律 I）。

**失败模式 C：伏笔蒸发**（Thread Evaporation）。早期精心植入的叙事伏笔在后续发展中完全消失——不是被有意放弃，而是被无意遗忘。经典编剧理论中的契诃夫法则（「第一幕墙上的枪必须在第三幕开火」）在无状态管理的 LLM 叙事中频繁失效。

**失败模式 D：张力坍塌**（Tension Collapse）。叙事张力在长会话中呈现以下两种退化模式之一：(i) 单调递减——系统无法自行引入新的冲突或复杂化，叙事变得平淡；(ii) 混乱振荡——缺乏节奏控制，连续的高强度事件导致叙事疲劳。这两种模式都偏离了经典编剧理论所描述的健康张力曲线（参见 §6.3 三幕同构）。

**失败模式 E：因果断裂**（Causal Rupture）。事件之间的因果链断裂：角色突然做出与其既定动机不符的行为，或者结果与前因无逻辑关系。在形式化框架中，这对应因果守恒（定律 II）的违反。

这五种失败模式并非独立——它们共享一个共同的根源：LLM 缺乏对叙事状态的显式、持久、结构化表示。本文提出的叙事物理学框架正是为了提供这样一种表示，并在其上定义约束和动力学。

### 1.3 为什么需要形式化

读者可能会问：为什么需要用数学来描述故事？编剧们几千年来凭直觉就能写出好故事。

我们的回答是：**人类叙事者不需要形式化，但机器叙事者需要。** 人类编剧拥有对「好故事」的直觉——这种直觉来自多年的阅读、写作和文化浸润。LLM 虽然在训练中接触了大量叙事文本，但它缺乏两个关键能力：(1) 在无限长的交互中维护一致的世界状态；(2) 在没有全局规划的情况下维持良好的叙事节奏。

形式化的价值在于三点：

1. **可计算性**。一旦叙事结构被表示为图，所有的一致性检查都变为图上的算法问题。我们可以精确地说出哪些检查是多项式时间的（定理 1），哪些是不可判定的（定理 2）。
2. **可度量性**。叙事熵 $H(G)$ 提供了一个标量指标来量化「叙事有多紧张」。这使得节奏控制从「凭感觉」变为「看仪表盘」。
3. **可推导性**。一旦定义了压力函数，系统对叙事元素的优先级排序就是数学推导的结果，而非 ad hoc 的规则工程。

本文的方法论受物理学启发：我们不是直接规定「好故事应该怎样」（这相当于告诉鸟「你应该有翅膀」），而是建立叙事世界的「物理定律」（相当于定义重力和空气动力学），让好的叙事结构作为这些定律的自然后果涌现出来。

### 1.4 贡献与论文结构

本文的贡献分为三个层次：

1. **建模层**（§3）：将叙事状态形式化为带类型的有向属性图，定义六种粒子类型和五类力，建立叙事本体论。
2. **约束层**（§4）：在叙事图上建立五条守恒定律，分析其判定的计算复杂度，区分可判定约束与不可判定约束。
3. **动力学层**（§5–7）：定义压力函数族、叙事熵、Laplacian 谱分析、非平衡热力学类比，提出 6 条猜想。

论文组织如下。§2 回顾所需的数学预备知识。§3 建立叙事图模型。§4 定义守恒定律并分析其复杂度。§5 引入压力函数族和 Laplacian 分析。§6 发展叙事热力学理论。§7 提出猜想和开放问题。§8 通过一个完整案例演示框架的运作。§9 综述相关工作。§10 讨论局限性和假设。§11 总结并展望未来方向。

## 2. 预备知识

本节回顾后续章节所需的数学工具。熟悉这些概念的读者可直接跳至 §3。

### 2.1 有向属性图

**定义**（有向属性图）。一个有向属性图是三元组 $G = (V, E, \phi)$，其中 $V$ 是有限节点集，$E \subseteq V \times \mathcal{L} \times V$ 是带标签的有向边集（$\mathcal{L}$ 为标签集合），$\phi: E \to \mathcal{A}$ 是将每条边映射到属性空间 $\mathcal{A}$ 的函数。

与标准有向图不同，属性图的每条边携带两类附加信息：一个类型标签 $\ell \in \mathcal{L}$（如 TRUSTS、KNOWS）和一个属性值 $\phi(e) \in \mathcal{A}$（如信任等级 $= 8$）。属性图广泛用于知识图谱（如 Wikidata、Neo4j），本文将其引入叙事建模领域。

在叙事语境中：节点表示叙事实体（角色、地点、伏笔等），边表示实体间的关系（空间位置、认知状态、情感关系等），属性值携带关系的量化参数（信任度、紧迫度、确信度等）。

### 2.2 谱图论基础

给定图 $G = (V, E)$，定义以下矩阵：

- **邻接矩阵** $A \in \mathbb{R}^{N \times N}$：$A_{ij} = 1$ 若 $(v_i, v_j) \in E$，否则为 0。对于加权图，$A_{ij}$ 为边权。
- **度矩阵** $D = \text{diag}(d_1, \ldots, d_N)$：$d_i = \sum_j A_{ij}$。
- **图 Laplacian** $L = D - A$：半正定对称矩阵，特征值 $0 = \lambda_1 \leq \lambda_2 \leq \cdots \leq \lambda_N$。

Laplacian 矩阵在图论中扮演的角色类似于物理学中的拉普拉斯算子（$\nabla^2$）——它描述信号在图上的扩散行为。特别重要的是 **Fiedler 值** $\lambda_2$（第二小特征值），它度量图的**代数连通度**[^14]：

- $\lambda_2 = 0$ 当且仅当图不连通（存在孤立的子图）；
- $\lambda_2$ 越大，图越「紧密连接」——任意两个节点间的信息传递越高效。

直觉上，可以把 $\lambda_2$ 想象为图的「刚度」：$\lambda_2$ 大的图像一块钢板，推动一个角会带动整体；$\lambda_2$ 小的图像几块松散连接的拼图，推动一块其他的几乎不动。我们将在 §5.3 中利用这一性质分析叙事压力的传导速率。

**注意**。标准 Laplacian $L = D - A$ 定义于无向图（$A$ 对称）。本文的叙事图是有向图，因此在涉及谱分析时，我们对邻接矩阵做对称化处理 $A_{\text{sym}} = (A + A^\top)/2$，对应的 Laplacian $L_{\text{sym}} = D_{\text{sym}} - A_{\text{sym}}$ 是实对称半正定矩阵，其谱性质（包括 Fiedler 值）与无向情况一致。对称化的叙事含义是：若 Alice 信任 Bob（有向边），则在压力传导意义上，这条关系对双方都产生影响——这与社交网络中的常见假设一致。

### 2.3 线性时序逻辑

线性时序逻辑（LTL）[^15] 是一种用于描述序列性质的形式语言。在本文中，我们只使用一个算子：

- $\Diamond\, \varphi$（eventually $\varphi$）：表示「在未来某个时刻，$\varphi$ 成立」。

以及它的有界版本：

- $\Diamond_{\leq k}\, \varphi$：表示「在未来 $k$ 步内，$\varphi$ 成立」。

例如，「每个植入的伏笔最终都会被兑现或放弃」可以写为：

$$\forall\, t:\; \text{status}(t) = \texttt{planted} \implies \Diamond\; \text{status}(t) \in \{\texttt{paid}, \texttt{abandoned}\}$$

这比自然语言精确得多：它明确了量化范围（所有伏笔）和时间语义（最终，而非立即）。

### 2.4 信息论基础

**Kolmogorov 复杂度** $K(x)$ 定义为能够输出字符串 $x$ 的最短程序的长度。它度量了 $x$ 的「内在信息含量」——不可压缩的部分。虽然 $K(x)$ 是不可计算的，但它提供了压缩的理论极限。

**Shannon 熵** $H(X) = -\sum_x p(x) \log p(x)$ 度量随机变量 $X$ 的不确定性。本文将定义的叙事熵（§6.1）借用了「熵」的名称，但严格来说是一个确定性标量函数而非概率度量——它度量的是叙事图中未解决张力的总量，而非随机性。我们保留这一命名是因为它与热力学熵的类比在动力学层面（§6.4）是深刻的。

### 2.5 动力系统与稳定性

**Lyapunov 函数** 是分析动力系统稳定性的核心工具。设离散动力系统 $\mathbf{x}(n+1) = f(\mathbf{x}(n))$，若存在连续函数 $V: \mathbb{R}^d \to \mathbb{R}_{\geq 0}$ 满足：

1. $V(\mathbf{x}) = 0$ 当且仅当 $\mathbf{x} = \mathbf{x}^*$（平衡点）；
2. $V(f(\mathbf{x})) - V(\mathbf{x}) < 0$ 对所有 $\mathbf{x} \neq \mathbf{x}^*$；

则 $\mathbf{x}^*$ 是全局渐近稳定的，$V$ 称为 Lyapunov 函数。直觉上，$V$ 类似于物理系统的「势能」——如果每一步势能都在降低，系统必然趋向最低点。我们将在 Conjecture 6（§7.6）中为叙事系统构造 Lyapunov 候选函数。

## 3. 叙事图：形式化模型

### 3.1 符号约定

全文使用以下记号：

| 符号 | 含义 |
|------|------|
| $G = (V, E, \phi)$ | 有向属性图（叙事图） |
| $V$ | 节点集，$\|V\| = N$ |
| $E \subseteq V \times \mathcal{L} \times V$ | 带标签有向边集 |
| $\mathcal{L}$ | 边标签集（力类型） |
| $\phi: E \to \mathcal{A}$ | 属性函数 |
| $L(G) = D - A$ | 图 Laplacian |
| $0 = \lambda_1 \leq \lambda_2 \leq \cdots$ | $L$ 的特征值谱 |
| $\lambda_2(G)$ | Fiedler 值（代数连通度） |
| $n$ | 当前场景序号 |

### 3.2 叙事粒子

我们借用粒子物理学的隐喻：叙事世界由不同类型的「粒子」（节点）组成，它们之间通过「力」（边）相互作用。

**定义 1**（粒子类型系统）。节点集 $V$ 上定义类型函数 $\text{type}: V \to \Omega$，其中类型宇宙 $\Omega = \{\mathsf{C}, \mathsf{S}, \mathsf{T}, \mathsf{\Sigma}, \mathsf{G}, \mathsf{E}\}$：

$$V = \bigsqcup_{\omega \in \Omega} V_\omega \quad \text{（不相交并）}$$

即每个节点恰好属于一种类型。六种类型的定义如下。为避免与图 $G=(V,E,\phi)$ 的符号冲突，类型标签使用 $\mathsf{sans\text{-}serif}$ 字体，对应的节点子集使用花体（如 $\mathcal{C}, \mathcal{G}, \mathcal{E}$）：

| 类型 $\omega$ | 节点集 | 语义 | 签名 |
|---------------|--------|------|------|
| $\mathsf{C}$ | $\mathcal{C}$ | 角色（Character） | $(\text{id}, \text{traits}: 2^{\mathbb{S}}, \text{arc}: \mathbb{S} \rightharpoonup \mathbb{S} \times [0,1])$ |
| $\mathsf{S}$ | $\mathcal{S}$ | 场景（Setting） | $(\text{id}, \text{atmosphere}: \mathbb{S})$ |
| $\mathsf{T}$ | $\mathcal{T}$ | 伏笔（Thread） | $(\text{id}, \text{status}: \Xi, \text{weight}: \mathcal{W}, \text{age}: \mathbb{N})$ |
| $\mathsf{\Sigma}$ | $\Sigma$ | 秘密（Secret） | $(\text{id}, \text{tension}: [0,10], \text{revealed}: \mathbb{B})$ |
| $\mathsf{G}$ | $\mathcal{G}$ | 目标（Goal） | $(\text{id}, \text{intensity}: [0,10], \text{status}: \Psi)$ |
| $\mathsf{E}$ | $\mathcal{E}$ | 事件（Event） | $(\text{id}, \text{seq}: \mathbb{N}, \text{sig}: \{+,-\})$ |

其中 $\mathbb{S}$ 为字符串域，$\Xi = \{\texttt{planted}, \texttt{developing}, \texttt{paid}, \texttt{abandoned}\}$ 为 Thread 状态空间，$\mathcal{W} = \{\texttt{major}, \texttt{minor}, \texttt{subtle}\}$ 为权重域，$\Psi = \{\texttt{active}, \texttt{achieved}, \texttt{abandoned}, \texttt{blocked}\}$ 为 Goal 状态空间。$\rightharpoonup$ 表示偏函数（partial function），因为并非所有角色都有显式的成长弧线。

**Remark 1**（直觉解释）。用日常语言来说：

- **Character**（角色）是故事的行动者——他们做出决定、采取行动、承受后果。`traits` 是角色的稳定性格特征（如「好奇、固执」），`arc` 描述角色的成长轨迹（如「从怯懦到勇敢，进度 60%」）。
- **Setting**（场景）是叙事发生的空间——一间酒馆、一座城堡、一片战场。`atmosphere` 是该空间的情绪基调（如「阴森」「温馨」）。
- **Thread**（伏笔）是一条尚未完结的因果线——「墙上的枪」「神秘的信件」「角色的承诺」。它有四种生命周期状态：planted（刚植入）→ developing（正在发展）→ paid（已兑现）或 abandoned（被放弃）。`weight` 区分主线（major）、支线（minor）和暗线（subtle）。
- **Secret**（秘密）是角色间的信息不对称——有人知道而有人不知道的事实。`tension` 量化这个秘密一旦曝光会产生多大的冲击（0 = 无关紧要，10 = 毁天灭地）。
- **Goal**（目标）是角色追求的欲望——「找到凶手」「保护家人」「获得权力」。`intensity` 量化追求的强度。
- **Event**（事件）是已经发生的事实——不可撤销的叙事历史。`sig` 为 $+$ 或 $-$，表示对持有者而言是正面还是负面事件。

这些类型的选择并非任意。它们对应了叙事理论中反复出现的核心元素：Propp[^16] 的形态学中的 dramatis personae（角色），Aristotle《诗学》中的 mythos（情节/事件），Todorov 的叙事平衡理论中的 disruption（目标/冲突），以及 Barthes 的叙事代码中的 hermeneutic code（秘密/伏笔）。

**公理 1**（主动性不对称）。定义因果源集 $\text{Src}(\mathcal{F}_{\text{causal}}) = \{v \mid \exists\, (v, \ell, u) \in E,\, \ell \in \mathcal{F}_{\text{causal}}\}$。则：

$$\text{Src}(\mathcal{F}_{\text{causal}}) \subseteq \mathcal{C} \cup \mathcal{E}$$

**Remark 2**（主动性的含义）。公理 1 断言只有角色和事件可以作为因果链的源。场景不能「导致」事件（酒馆不会主动攻击人），目标不能「导致」事件（欲望本身不产生行动），秘密不能「导致」事件（信息本身是惰性的）。这将因果权限锁定在行动者层，防止了叙事中常见的「场景驱动的 deus ex machina」——即环境无缘由地产生事件来推动情节。

当然，在自然语言中我们经常说「暴风雨导致了沉船」，但在本框架中，这需要拆解为：暴风雨是一个 Event（由之前的 Event 或 Character 行动触发），沉船是另一个 Event，两者通过 CAUSED 边连接。

### 3.3 叙事力

如果粒子是叙事世界的「物质」，那么力就是「相互作用」。

**定义 2**（力分类）。边标签集 $\mathcal{L}$ 分为五大类：

$$\mathcal{L} = \mathcal{F}_{\text{sp}} \sqcup \mathcal{F}_{\text{cog}} \sqcup \mathcal{F}_{\text{rel}} \sqcup \mathcal{F}_{\text{cau}} \sqcup \mathcal{F}_{\text{nar}}$$

| 力族 | 含义 | 成员 | 类比 |
|------|------|------|------|
| $\mathcal{F}_{\text{sp}}$（空间力） | 实体在空间中的位置关系 | AT, IN, NEAR | 万有引力 |
| $\mathcal{F}_{\text{cog}}$（认知力） | 角色对信息的认知状态 | KNOWS, SUSPECTS, BELIEVES, HIDES | 电磁力 |
| $\mathcal{F}_{\text{rel}}$（关系力） | 角色之间的情感关系 | TRUSTS, LOVES, HATES, FEARS, RESPECTS, OWES | 弱核力 |
| $\mathcal{F}_{\text{cau}}$（因果力） | 事件之间的因果关系 | CAUSED, PREVENTED, ENABLED | 强核力 |
| $\mathcal{F}_{\text{nar}}$（叙事力） | 叙事结构的元关系 | INVOLVES, WANTS, CONFLICTS, BLOCKS | （元力） |

**Remark 3**（为什么是五类）。这五类力覆盖了叙事的五个维度：空间（角色在哪里）、认知（角色知道什么）、情感（角色对彼此感觉如何）、因果（事件如何关联）、结构（叙事元素如何组织）。任何叙事关系都可以归入其中之一。例如：「Alice 在酒馆」→ 空间力；「Alice 知道 Bob 的秘密」→ 认知力；「Alice 信任 Bob」→ 关系力；「Alice 的行动导致了爆炸」→ 因果力；「这条伏笔涉及 Alice」→ 叙事力。

**定义 3**（力的类型约束）。每种力 $\ell \in \mathcal{L}$ 附带定义域和值域的类型约束 $\text{dom}(\ell) \times \text{cod}(\ell) \subseteq \Omega \times \Omega$：

| 力 $\ell$ | $\text{dom}$ | $\text{cod}$ | 属性 $\phi(\ell)$ |
|-----------|-------------|-------------|-------------------|
| $\texttt{AT}$ | $\mathsf{C}$ | $\mathsf{S}$ | — |
| $\texttt{KNOWS}$ | $\mathsf{C}$ | $\mathsf{\Sigma}$ | $\kappa \in [0,1]$（确信度） |
| $\texttt{SUSPECTS}$ | $\mathsf{C}$ | $\mathsf{\Sigma}$ | $\kappa \in [0,1]$ |
| $\texttt{HIDES}$ | $\mathsf{C}$ | $\mathsf{\Sigma}$ | $\text{from} \subseteq \mathcal{C}$ |
| $\texttt{TRUSTS}$ | $\mathsf{C}$ | $\mathsf{C}$ | $\text{level} \in [0,10]$ |
| $\texttt{LOVES}$ | $\mathsf{C}$ | $\mathsf{C}$ | $\text{type} \in \{\text{rom}, \text{fam}, \text{pla}\}$ |
| $\texttt{HATES}$ | $\mathsf{C}$ | $\mathsf{C}$ | $\text{reason}: \mathbb{S}$ |
| $\texttt{CAUSED}$ | $\mathsf{C} \cup \mathsf{E}$ | $\mathsf{E}$ | — |
| $\texttt{INVOLVES}$ | $\mathsf{T}$ | $\mathsf{C}$ | $\text{role}: \mathbb{S}$ |
| $\texttt{WANTS}$ | $\mathsf{C}$ | $\mathsf{G}$ | — |
| $\texttt{CONFLICTS}$ | $\mathsf{G}$ | $\mathsf{G}$ | — |

**Remark 4**（类型约束的作用）。类型约束禁止了语义上荒谬的关系。例如：$\texttt{TRUSTS}$ 的 dom 和 cod 都是 $C$（角色），所以不可能出现「一个秘密信任一个场景」这样的边。类型约束在编程语言中的对应物是类型系统——它们通过静态检查排除了一大类运行时错误。

### 3.4 良类型性

**命题 1**（类型安全性）。若叙事图 $G$ 的每条边 $e = (u, \ell, v)$ 均满足 $\text{type}(u) \in \text{dom}(\ell) \land \text{type}(v) \in \text{cod}(\ell)$，则称 $G$ 是**良类型的**（well-typed）。

良类型是叙事图的基本卫生条件。它不保证故事「好」，但排除了一类结构性荒谬。这类似于编程中的类型检查：通过类型检查的程序不一定正确，但至少不会出现「对整数调用字符串方法」这样的低级错误。

**命题 1'**（良类型性的可验证性）。良类型性在 $O(|E|)$ 时间内可判定。

*证明*。遍历每条边 $(u, \ell, v)$，检查 $\text{type}(u) \in \text{dom}(\ell)$ 和 $\text{type}(v) \in \text{cod}(\ell)$。每次检查 $O(1)$，总共 $O(|E|)$。$\square$

## 4. 守恒定律

### 4.1 设计哲学：软约束与力

在物理学中，守恒定律（如能量守恒、动量守恒）描述的是封闭系统的不变量——它们不能被违反。但叙事系统不是封闭系统：它持续接受外部输入（玩家的行动），这些输入可能破坏任何约束。

因此，我们对守恒定律采取**软约束**（soft constraint）方案：守恒定律的违反不导致系统拒绝操作（这会破坏玩家的自主性），而是产生可量化的**违反压力**，驱动系统在后续场景中趋向合法状态。

这与物理学中的**最小作用量原理**精神一致：系统不「遵守规则」，而是「遵循最小能量路径」。一个球不知道牛顿定律，但它总是沿抛物线运动，因为抛物线是作用量最小的路径。类似地，一个 LLM 不需要被告知「你必须回收伏笔」——如果伏笔的兑现压力足够高，它自然会在生成中倾向于处理它（参见 §5 的压力函数族和 §7.1 的 Le Chatelier 猜想）。

### 4.2 五条定律

**定律 I**（存在守恒，Conservation of Existence）。

$$\forall\, c \in \mathcal{C}:\; \exists!\, s \in \mathcal{S}:\; (c, \texttt{AT}, s) \in E \tag{I}$$

注意此处使用 $\exists!$（唯一存在量词）：每个角色必须且只能存在于一个空间中。这比简单的 $\exists$ 更强——它排除了角色同时出现在多个位置的叠加态。

**叙事诠释**。存在守恒对应物理学中「物体不能同时在两个地方」的常识。违反示例：在某个场景中，Alice 在酒馆与 Bob 交谈；同一场景中，Alice 又出现在码头追踪线索。这是 §1.2 中「空间传送」失败模式的形式化表述。违反存在守恒产生的压力会驱动系统在下一场景中明确角色的位置。

**定律 II**（因果守恒，Conservation of Causality）。

$$\forall\, e \in \mathcal{E} \setminus \{e_0\}:\; \exists\, v \in \mathcal{C} \cup \mathcal{E}:\; (v, \texttt{CAUSED}, e) \in E \tag{II}$$

初始事件 $e_0$ 是唯一的无因元素（公理性存在）。所有后续事件必须可追溯至因果链——我们称此为**接地原则**（Grounding Principle）。

**叙事诠释**。因果守恒保证了叙事中不存在「无缘无故发生的事」。每个事件要么是某个角色的行动导致的，要么是某个先前事件的后果。违反示例：Bob 突然被捕，但叙事中从未交代谁报了警、为什么报警。因果断裂会让读者/玩家产生强烈的不合理感——这正是违反压力的主观体验。

**定律 III**（秘密守恒，Conservation of Secrets）。

$$\forall\, \sigma \in \Sigma:\; |\{c \in \mathcal{C} \mid (c, \texttt{KNOWS}, \sigma) \in E\}| \geq 1 \tag{III}$$

**叙事诠释**。一个没有任何人知道的「秘密」在叙事上不存在——如果没有角色持有某个信息，它就不可能在故事中被揭示、被暗示或产生任何效果。秘密守恒确保每个秘密至少有一个「载体」。违反示例：叙事中提到「一个可怕的秘密改变了一切」，但没有任何角色知道这个秘密。这是一个悬空指针——形式化为守恒律后，系统可以自动检测并标记这类问题。

**定律 IV**（契诃夫守恒，Chekhov Conservation）。

$$\forall\, t \in \mathcal{T}:\; \text{status}(t) = \texttt{planted} \implies \Diamond\;\text{status}(t) \in \{\texttt{paid}, \texttt{abandoned}\} \tag{IV}$$

$\Diamond$ 为线性时序逻辑（LTL）的 eventually 算子（参见 §2.3）。

**叙事诠释**。契诃夫守恒是对著名的「契诃夫之枪」原则的形式化：如果你在第一幕展示了墙上的一把枪（planted），那么到故事结束时它必须被开火（paid）或被明确移走（abandoned）。注意 $\Diamond$ 是一个时序算子——它不要求立即兑现，只要求最终兑现。这允许伏笔存在任意长的潜伏期，同时保证不存在永远被遗忘的伏笔。

这是五条定律中最微妙的一条：它涉及未来时间的承诺，而非当前状态的约束。这一时序性质导致了深刻的计算后果（参见 §4.3 定理 2）。

**定律 V**（目标守恒，Conservation of Goals）。

$$\forall\, g \in \mathcal{G}:\; \text{status}(g) = \texttt{active} \implies \exists\, c \in \mathcal{C}:\; (c, \texttt{WANTS}, g) \in E \tag{V}$$

**叙事诠释**。一个「活跃」的目标必须有追求者。如果一个目标处于 active 状态但没有角色 WANTS 它——比如原来追求它的角色已经死亡或退场——则该目标应自动转为 abandoned。违反此定律的叙事会出现「幽灵目标」：一个没有人在追求但系统仍在追踪的空头目标。

### 4.3 一致性判定的计算复杂度

我们现在分析：给定一个叙事图，检验它是否满足守恒定律的计算代价是多少？

**定义 4**（一致性谓词）。

$$\text{CON}(G) \iff \bigwedge_{i \in \{\text{I},\ldots,\text{V}\}} \text{Law}_i(G)$$

**定理 1**（静态一致性的可判定性）。对于定律 I, II, III, V，一致性判定在 $O(|V| + |E|)$ 内可完成（单遍图遍历）。

*证明*。各定律均可化归为对特定类型节点的邻接边检查。具体地：

- **定律 I**：遍历 $\mathcal{C}$ 中的每个角色 $c$，计数其 AT 类型出边数 $d_{\texttt{AT}}^+(c)$。若 $d_{\texttt{AT}}^+(c) \neq 1$，则违反。时间 $O(|\mathcal{C}| + |E_{\texttt{AT}}|)$。
- **定律 II**：遍历 $\mathcal{E} \setminus \{e_0\}$ 中的每个事件 $e$，检查是否存在 CAUSED 类型入边。时间 $O(|\mathcal{E}| + |E_{\texttt{CAUSED}}|)$。
- **定律 III**：遍历 $\Sigma$ 中的每个秘密 $\sigma$，检查是否存在 KNOWS 类型入边。时间 $O(|\Sigma| + |E_{\texttt{KNOWS}}|)$。
- **定律 V**：遍历 $\mathcal{G}$ 中状态为 active 的每个目标 $g$，检查是否存在 WANTS 类型入边。时间 $O(|\mathcal{G}| + |E_{\texttt{WANTS}}|)$。

四者之和为 $O(|V| + |E|)$。$\square$

这一结果意味着，检验叙事图的当前快照是否一致是**廉价**的——代价与图的规模成线性关系。对于典型的交互叙事（几十个角色、几百条关系），这在毫秒级内可完成。

**定理 2**（契诃夫完备性的不可判定性）。设 $\mathcal{N}$ 为叙事图的无穷演化序列 $(G_0, G_1, G_2, \ldots)$，其中 Thread 集合 $\mathcal{T}$ 在 $G_0$ 中完全确定（不再新增 Thread）。判定「序列 $\mathcal{N}$ 是否满足定律 IV」是 $\Sigma_1^0$-完备的。

$\Sigma_1^0$ 是算术层级的第一层，包含所有形如「$\exists n: P(n)$」的可递归枚举谓词。$\Sigma_1^0$-完备意味着该问题与停机问题等价：不存在通用算法能判定任意叙事序列是否满足契诃夫守恒。

*证明*。

**$\Sigma_1^0$-难**。我们构造从停机问题到契诃夫判定的归约。给定图灵机 $M$ 和输入 $x$，构造叙事图序列 $\mathcal{N}_M$：在 $G_0$ 中植入唯一的 Thread $t_M$（$\text{status} = \texttt{planted}$），演化规则模拟 $M(x)$：步骤 $n$ 执行 $M(x)$ 的第 $n$ 步计算（将 TM 配置编码为 Event 节点的属性），当且仅当 $M(x)$ 停机时令 $\text{status}(t_M) = \texttt{paid}$。则 $\mathcal{N}_M \models \text{Law IV}$ 当且仅当 $M(x)$ 停机。

**$\Sigma_1^0$ 成员**。当 $\mathcal{T}$ 有限且固定时，定律 IV 等价于有限合取 $\bigwedge_{t \in \mathcal{T}} \exists n_t: \text{status}(t, n_t) \in \{\texttt{paid}, \texttt{abandoned}\}$。每个合取项是 $\Sigma_1^0$ 谓词，有限个 $\Sigma_1^0$ 谓词的合取仍是 $\Sigma_1^0$。$\square$

**Remark**。若允许在演化过程中动态植入新 Thread（即 $\mathcal{T}$ 随时间增长），定律 IV 的形式变为 $\forall n: \forall t$ planted at $n$: $\exists m > n: \text{resolved}(t, m)$，这是 $\Pi_2^0$ 谓词（$\forall\text{-}\exists$ 形式），严格强于 $\Sigma_1^0$。实际系统中，Thread 的动态植入是常态；定理 2 的 $\Sigma_1^0$ 结果适用于「给定一组已知伏笔，判定它们是否全部被回收」这一子问题。

**推论 1**。在实际系统中，定律 IV 必须弱化为有界时间版本：

$$\forall\, t \in \mathcal{T}:\; \text{status}(t) = \texttt{planted} \implies \Diamond_{\leq \alpha}\;\text{status}(t) \in \{\texttt{paid}, \texttt{abandoned}\}$$

其中 $\Diamond_{\leq \alpha}$ 为有界 eventually 算子（在 $\alpha$ 个场景内），此时判定退化为 $O(|\mathcal{T}|)$。阈值 $\alpha$ 的最优选择本身是一个开放问题（§7.5, Conjecture 5）。

**Remark 5**（不可判定性的实际含义）。定理 2 的不可判定性结果不意味着契诃夫守恒在实践中无用。它意味着我们不能构造一个通用的「预言机」来判定任意伏笔是否最终会被回收。但在实际系统中，叙事序列不是任意的——它们受到 LLM 生成分布的约束。推论 1 的有界弱化版本在实践中是完全可行的，且阈值 $\alpha$ 可以根据伏笔的权重类型动态调整（参见 Conjecture 5）。

### 4.4 约束层级

五条守恒定律形成了一个层级结构：

```
硬度层级（从最易到最难验证）：

│ 定律 I   (存在)  ← O(|V|+|E|)，纯当前状态
│ 定律 III  (秘密)  ← O(|V|+|E|)，纯当前状态
│ 定律 V   (目标)  ← O(|V|+|E|)，纯当前状态
│ 定律 II  (因果)  ← O(|V|+|E|)，当前状态 + 历史
│ 定律 IV  (契诃夫) ← 不可判定（一般情况），O(|T|)（有界版）
▼
```

前三条定律只涉及当前快照的局部属性（「每个角色现在在哪」），可以在每个场景结束后即时检查。定律 II 需要因果历史（「这个事件是怎么来的」），但仍是当前图的可遍历属性。定律 IV 本质上不同——它涉及对未来的承诺，因此在一般情况下不可判定。

这一层级结构的实际意义是：系统应该以不同的频率和强度监控不同的定律。定律 I–III 可以在每个场景后作为硬性检查运行（$O(|V|+|E|)$ 代价微不足道）；定律 IV 只能以启发式方式监控（通过兑现压力 $P_\tau$ 来近似）。

## 5. 压力动力学

守恒定律告诉我们叙事图的合法状态应该满足什么条件，但没有告诉我们系统如何从一个状态演化到下一个状态。**压力函数**填补了这个空缺：它们量化了每个叙事元素「想要被处理」的程度，为叙事演化提供驱动力。

### 5.1 压力函数族

**定义 5**（压力函数族）。定义五元函数族 $\mathcal{P} = \{P_\tau, P_\sigma, P_\gamma, P_\rho, P_\theta\}$，每个 $P_i: G \times \mathbb{N} \to \mathbb{R}_{\geq 0}$。

**I. 兑现压力**（Thread Pressure）。

$$P_\tau(t, n) = \text{age}(t,n)^{\,\beta} \cdot w(t), \quad \beta > 1 \tag{P1}$$

$$\text{age}(t, n) = n - n_{\text{plant}}(t) + 1, \quad w: \mathcal{W} \to \mathbb{R}_{>0}$$

$+1$ 保证植入当场景即有 $\text{age} = 1$（伏笔一经植入即开始产生压力）。

**为什么选择超线性增长**？$\beta > 1$ 意味着压力随时间加速增长——第 10 场景时的压力远大于第 5 场景时的两倍。这不是任意选择，而是有认知心理学依据：Stevens 幂律[^24]指出，人对刺激强度的主观感知 $\psi$ 与物理强度 $I$ 之间满足 $\psi = k \cdot I^\beta$，其中 $\beta > 1$ 对应加速增长的刺激（如疼痛、电击——以及悬念的折磨）。观众对未兑现承诺的不耐烦是加速增长的，而非线性的——「等了 3 集」和「等了 10 集」的差距远大于 3 倍。

**物理类比**。兑现压力类似于弹簧的弹性势能 $U = \frac{1}{2}kx^2$（$\beta = 2$ 的特例）：伏笔被「拉伸」得越远（age 越大），恢复力（兑现驱动）越强。

**II. 揭示压力**（Secret Pressure）。

$$P_\sigma(\sigma, G) = \tau(\sigma) \cdot \frac{|\mathcal{K}(\sigma)| \cdot (|\mathcal{C}| - |\mathcal{K}(\sigma)|)}{|\mathcal{C}|} \cdot \Big(1 + \alpha_s \cdot |\mathcal{Q}(\sigma)|\Big) \cdot \mathbb{1}[\neg\,\text{revealed}(\sigma)] \tag{P2}$$

其中 $\mathcal{K}(\sigma) = \{c \mid (c, \texttt{KNOWS}, \sigma) \in E\}$（知晓者集合），$\mathcal{Q}(\sigma) = \{c \mid (c, \texttt{SUSPECTS}, \sigma) \in E\}$（怀疑者集合）。

**公式要点**。核心因子 $|\mathcal{K}| \cdot (|\mathcal{C}| - |\mathcal{K}|) / |\mathcal{C}|$ 度量**信息不对称程度**：当无人知晓（$|\mathcal{K}| = 0$）或全体知晓（$|\mathcal{K}| = |\mathcal{C}|$）时，该因子为零——前者无泄露可能，后者无信息差。当约半数知晓时达到最大值——这正是秘密最具叙事张力的状态。怀疑者项 $\alpha_s |\mathcal{Q}|$ 提供额外放大。指示函数 $\mathbb{1}[\neg\,\text{revealed}]$ 确保一旦秘密被公开揭示（$\text{revealed}(\sigma) = \texttt{true}$），压力立即归零。

**直觉**。一个秘密的揭示压力取决于三个因素：(1) 内在张力 $\tau(\sigma)$——「Bob 喜欢吃苹果」和「Bob 杀了 Alice 的父亲」天壤之别；(2) 信息不对称——知道的人和不知道的人共存时张力最大，所有人都知或都不知时张力消失；(3) 怀疑者放大——有人在主动追查时，秘密更容易爆发。

**信息论解释**。秘密可视为叙事中的信息不对称。设角色 $c$ 关于秘密 $\sigma$ 的认知状态为 $X_c^\sigma \in \{\text{know}, \text{suspect}, \text{ignorant}\}$。定义认知分散度（角色间认知状态的异质性）为各角色边缘熵之和：

$$S(\sigma, \mathcal{C}) = \sum_{c \in \mathcal{C}} H(X_c^\sigma) = -\sum_{c \in \mathcal{C}} \sum_{x} p(X_c^\sigma = x) \log p(X_c^\sigma = x)$$

当所有角色认知状态一致（全知或全不知）时 $S = 0$，当知晓者与无知者共存时 $S > 0$。$P_\sigma$ 中的 $|\mathcal{K}| \cdot (|\mathcal{C}| - |\mathcal{K}|)$ 因子正是 $S(\sigma, \mathcal{C})$ 在二元认知（know/not-know）下的一阶近似。

**III. 冲突压力**（Conflict Pressure）。

$$P_\gamma(g_1, g_2, s) = \frac{\iota(g_1) + \iota(g_2)}{2} \cdot \mathbb{1}\big[\text{holder}(g_1) \in \text{chars}(s) \,\land\, \text{holder}(g_2) \in \text{chars}(s)\big] \tag{P3}$$

其中 $\iota(g)$ 为目标的 intensity，$\mathbb{1}[\cdot]$ 为示性函数（条件为真时取 1，否则取 0）。

**关键性质：空间激活**（Spatial Activation）。冲突压力仅在持有者共享空间时从潜在态跃迁为激活态。两个角色各自怀有矛盾的目标，但如果他们从不见面，冲突就不会爆发——它只是一个潜在的定时炸弹。一旦他们出现在同一场景中，冲突压力突然激活。

**叙事意义**。这对应亚里士多德《诗学》中三一律的「地点统一」要求：戏剧冲突需要空间上的共在。这也解释了为什么经典戏剧经常设计「所有人聚集在一个房间」的场景（如阿加莎·克里斯蒂的《无人生还》）——那是使所有潜在冲突同时激活的最优策略。

**IV. 关系压力**（Relationship Pressure）。

$$P_\rho(c_i, c_j, G) = \max\Big(0,\; |\mathcal{E}_{\cap}(c_i, c_j)| - |\mathcal{F}_{\text{rel}}(c_i, c_j)|\Big) \tag{P4}$$

其中 $\mathcal{E}_\cap(c_i, c_j)$ 为两人共同参与的事件集，$\mathcal{F}_{\text{rel}}(c_i, c_j)$ 为两人之间的关系边集。

**直觉**。两个角色如果频繁互动（共同事件多）但关系定义模糊（关系边少），就会产生关系压力——驱动叙事去明确他们到底是什么关系。这捕捉了一个常见的叙事直觉：当两个角色一起经历了很多事情后，观众期望看到他们的关系被定义或深化。如果系统一直不回应这种期望，就会产生「关系悬空」的不满感。

**V. 张力压力**（Tension Homeostasis）。

$$P_\theta(G, \Delta) = \delta_{\text{sign}} \cdot \Delta, \quad \delta_{\text{sign}} = \begin{cases} +\delta_+ & \text{if } \overline{H}_\Delta < H_{\text{low}} \\ -\delta_- & \text{if } \overline{H}_\Delta > H_{\text{high}} \\ 0 & \text{otherwise} \end{cases} \tag{P5}$$

其中 $\overline{H}_\Delta$ 为最近 $\Delta$ 个场景的滑动平均熵（定义见 §6.1）。

**直觉**。张力压力是一个**恒温器**（thermostat）：当叙事太平淡（$\overline{H}_\Delta < H_{\text{low}}$），它产生升温压力，鼓励引入新的 Thread 或 Conflict；当叙事太混乱（$\overline{H}_\Delta > H_{\text{high}}$），它产生释放压力，鼓励解决一些悬而未决的问题。这是 §1.2 中「张力坍塌」失败模式的直接对策。

**物理类比**。张力压力类似于恒温动物的体温调节：无论外界温度如何变化，身体都会通过出汗（降温）或颤抖（升温）将核心温度维持在 37°C 左右。叙事系统也需要类似的自稳定机制。

### 5.2 压力的物理类比总结

为了帮助读者建立直觉，我们总结五种压力与物理学的对应关系：

| 压力 | 物理类比 | 驱动行为 | 释放条件 |
|------|---------|---------|---------|
| $P_\tau$（兑现） | 弹性势能 | 趋向兑现伏笔 | Thread → paid/abandoned |
| $P_\sigma$（揭示） | 渗透压 | 趋向揭示秘密 | Secret → revealed |
| $P_\gamma$（冲突） | 静电力 | 趋向冲突爆发 | Conflict → confrontation |
| $P_\rho$（关系） | 成键能 | 趋向关系定义 | 添加/深化关系边 |
| $P_\theta$（张力） | 体温调节 | 趋向节奏平衡 | 熵回归目标区间 |

**Remark 6**（这些类比的严格性）。上述物理类比是启发性的，不是严格的同构。例如，弹性势能 $U \propto x^2$ 是精确的二次函数，而兑现压力 $P_\tau \propto \text{age}^\beta$ 中的 $\beta$ 是待定参数。类比的价值在于：它暗示了叙事动力学可能共享物理动力学的某些结构性质（如能量最小化、稳定平衡、振荡）——这些暗示构成了 §7 中猜想的动机。

### 5.3 压力传导与 Laplacian

到目前为止，每种压力都是**局部**的——它只作用于直接相关的节点。但在实际叙事中，一个局部事件会通过关系网络传导影响。例如：Alice 发现了 Bob 的秘密（$P_\sigma$ 变化）→ Alice 对 Bob 的信任降低（$\mathcal{F}_{\text{rel}}$ 变化）→ Alice 与 Charlie 讨论此事（$P_\sigma$ 进一步扩散）→ Charlie 开始怀疑 Bob（$P_\sigma$ 再次变化）。

我们用图 Laplacian 来形式化这种传导。

**定义 6**（叙事 Laplacian）。定义叙事图 $G$ 的加权邻接矩阵 $A_\omega$，其中：

$$[A_\omega]_{ij} = \sum_{\ell:\,(v_i, \ell, v_j) \in E} \omega(\ell)$$

$\omega: \mathcal{L} \to \mathbb{R}_{>0}$ 为力的传导权重函数——不同类型的力传导压力的效率不同。例如，TRUSTS 边可能比 NEAR 边更有效地传导揭示压力（你更可能通过信任的人知道秘密，而不是通过地理邻近）。由于叙事图为有向图，我们按 §2.2 的约定对称化：$A_\omega^{\text{sym}} = (A_\omega + A_\omega^\top)/2$，对应的 Laplacian $L_\omega = D_\omega^{\text{sym}} - A_\omega^{\text{sym}}$，特征值谱 $0 = \lambda_1 \leq \lambda_2 \leq \cdots \leq \lambda_N$。

**定义 7**（有效压力）。节点 $v$ 的有效压力定义为直接压力加衰减传导项：

$$P_{\text{eff}}(v) = P_{\text{local}}(v) + \mu \sum_{(u, \ell, v) \in E} P_{\text{local}}(u) \cdot \omega(\ell) \tag{8}$$

其中 $\mu \in (0,1)$ 为全局衰减因子。$\mu$ 控制传导的「衰减速度」：$\mu$ 接近 1 时压力传导几乎无损（紧密耦合的叙事），$\mu$ 接近 0 时压力几乎不传导（松散的叙事）。

**命题 2**（矩阵形式）。设 $\mathbf{p} = [P_{\text{local}}(v_1), \ldots, P_{\text{local}}(v_N)]^\top$，则有效压力向量为：

$$\mathbf{p}_{\text{eff}} = (I + \mu A_\omega^\top)\,\mathbf{p} \tag{9}$$

*证明*。将定义 7 的求和展开为矩阵乘法。$(I + \mu A_\omega^\top)\mathbf{p}$ 的第 $i$ 个分量为 $p_i + \mu \sum_j [A_\omega]_{ji} \cdot p_j$，恰好等于 $P_{\text{local}}(v_i)$ 加上所有指向 $v_i$ 的边传导的压力。$\square$

### 5.4 多跳扩散与全局均衡

方程 (9) 只考虑了 1-跳传导（直接邻居的影响）。在实际叙事中，压力会沿路径多跳传导。当 $\mu < 1/\rho(A_\omega)$（$\rho$ 为谱半径）时，Neumann 级数收敛，可扩展为 $k$-跳传导：

$$\mathbf{p}_{\text{eff}}^{(k)} = \left(\sum_{j=0}^{k} \mu^j (A_\omega^\top)^j\right) \mathbf{p} \tag{10}$$

直觉上，$(A_\omega^\top)^j \mathbf{p}$ 计算的是通过长度为 $j$ 的路径传导的压力，乘以 $\mu^j$ 的衰减。极限 $k \to \infty$ 给出**全局均衡压力**：

$$\mathbf{p}^* = (I - \mu A_\omega^\top)^{-1}\,\mathbf{p} \tag{11}$$

$\mathbf{p}^*$ 告诉我们：当压力完全扩散后，每个节点承受的有效压力是多少。这类似于物理学中热传导达到稳态后的温度分布。

**定理 3**（压力均衡速率）。叙事图上的压力扩散过程收敛到均衡分布的速率由 Fiedler 值 $\lambda_2(L_\omega)$ 控制。具体地，设 $\mathbf{p}(n)$ 为第 $n$ 场景后的压力分布，$\mathbf{p}^*$ 为均衡分布，则：

$$\|\mathbf{p}(n) - \mathbf{p}^*\|_2 \leq \left(1 - \frac{\lambda_2}{d_{\max}}\right)^n \cdot \|\mathbf{p}(0) - \mathbf{p}^*\|_2 \tag{12}$$

其中 $d_{\max} = \max_i d_i$ 为最大加权度。

*证明*。将压力扩散建模为离散扩散过程 $\mathbf{p}(n+1) = W\mathbf{p}(n) + \mathbf{f}(n)$，其中 $W = I - \frac{1}{d_{\max}}L_\omega$ 是 lazy random walk 矩阵，$\mathbf{f}(n)$ 是外部注入。$W$ 的特征值为 $1 - \lambda_i/d_{\max}$，故 $W$ 的谱间隙为 $\lambda_2/d_{\max}$。在齐次情况（$\mathbf{f} = 0$）下，标准的谱分析给出几何收敛速率 $(1 - \lambda_2/d_{\max})^n$。$\square$

**物理解释**。定理 3 的实际含义是：

- $\lambda_2$ 大（叙事网络紧密）→ 压力传导快 → 一个局部事件迅速影响全局 → 叙事感知为「紧凑、环环相扣」。例如：在阿加莎·克里斯蒂的密室谋杀中，所有角色紧密关联，一个线索的发现立刻改变所有人的嫌疑程度。
- $\lambda_2$ 小（叙事网络松散）→ 压力传导慢 → 不同子情节线之间近乎独立演化 → 叙事感知为「散漫、缺乏主线」。例如：一部有太多互不关联支线的 RPG，玩家在不同任务间切换时感受不到整体推进。

这一观察将在 §7.2 中被提升为正式猜想（Conjecture 2）。

## 6. 叙事热力学

前一节在微观层面定义了五种压力函数。本节在宏观层面引入**叙事熵**——一个将所有局部压力聚合为单一标量指标的度量，并利用它发展叙事的热力学理论。

### 6.1 叙事熵

**定义 8**（叙事熵）。叙事图 $G$ 在场景 $n$ 处的熵定义为所有活跃压力之和：

$$H(G, n) = \sum_{t \in \mathcal{T}_{\text{open}}} P_\tau(t,n) + \sum_{\sigma \in \Sigma} P_\sigma(\sigma, G) + \sum_{(g_i,g_j) \in \mathcal{G}_\times} P_\gamma(g_i,g_j,s_n) + \sum_{\substack{c_i, c_j \in \mathcal{C} \\ i < j}} P_\rho(c_i,c_j,G) \tag{13}$$

其中 $\mathcal{T}_{\text{open}} = \{t \in \mathcal{T} \mid \text{status}(t) \notin \{\texttt{paid}, \texttt{abandoned}\}\}$，$\mathcal{G}_\times = \{(g_i, g_j) \mid (g_i, \texttt{CONFLICTS}, g_j) \in E\}$。

**Remark 7**（为什么叫「熵」）。严格地说，$H(G, n)$ 不是 Shannon 熵——它不是概率分布的函数，而是叙事图的确定性标量函数。我们使用「熵」这一命名有两个理由：(1) 它度量叙事系统的「无序度」或「未解决张力总量」，这与热力学熵的直觉一致；(2) 更重要的是，$H(G,n)$ 在动力学层面表现出与热力学熵类似的行为——它在无外部干预时单调增长（引理 1），在特定条件下可以被主动降低（$P_\theta$ 的调控），并且健康的系统维持在一个非零的稳态附近。这些动力学性质的对应关系将在 §6.4 中详细展开。

**直觉**。可以把叙事熵想象为一个「紧张度仪表盘」：$H$ 低意味着故事平静（所有伏笔都已回收，没有未解决的冲突），$H$ 高意味着故事紧张（大量伏笔悬而未决，多条冲突线同时激活，秘密随时可能爆发）。

### 6.2 熵产率

**定义 9**（熵产率）。定义单场景熵产率：

$$h(n) = H(G, n) - H(G, n-1) = \Delta H_{\text{plant}}(n) - \Delta H_{\text{resolve}}(n) + \Delta H_{\text{aging}}(n) \tag{14}$$

其中三项分别对应：

- $\Delta H_{\text{plant}}$：新元素植入带来的熵增（新的 Thread、Secret、Goal、Conflict 被引入）。
- $\Delta H_{\text{resolve}}$：旧元素解决带来的熵减（Thread 被 paid/abandoned，Secret 被 revealed，Goal 被 achieved）。
- $\Delta H_{\text{aging}}$：现存元素因 age 增长带来的自然熵增。

**引理 1**（自然熵增）。设 $\mathcal{T}_{\text{open}} \neq \emptyset$。在无任何植入或解决操作的「空转」场景中（即 $\Delta H_{\text{plant}} = \Delta H_{\text{resolve}} = 0$），叙事熵的 Thread 分量严格增长：

$$h_{\text{idle}}(n) = \sum_{t \in \mathcal{T}_{\text{open}}} \left[\text{age}(t,n)^\beta - \text{age}(t,n-1)^\beta\right] \cdot w(t) > 0$$

*证明*。因 $\beta > 1$ 且 $\text{age} \geq 1$（由 $+1$ 项保证），函数 $f(x) = x^\beta$ 严格递增且严格凸（$f'' > 0$）。对任意 $x \geq 1$，$f(x+1) - f(x) > 0$ 且差分递增。$\mathcal{T}_{\text{open}} \neq \emptyset$ 保证至少有一项正贡献。$\square$

**注**。空转场景中其他压力分量（$P_\sigma, P_\gamma, P_\rho$）不随时间自动变化（它们不含 age 因子），故 $h_{\text{idle}}$ 恰好等于 Thread 压力的增量。当 $\mathcal{T}_{\text{open}} = \emptyset$ 时，$h_{\text{idle}} = 0$——但此时总熵 $H$ 未必为零，可能仍有秘密压力或冲突压力贡献。

**Remark 8**（引理 1 的叙事含义）。引理 1 的直觉非常深刻：**不作为本身就在增加叙事压力**。如果一个 GM（游戏主持人）或 LLM 连续多个场景「什么都不做」——不引入新元素，也不解决旧问题——叙事熵仍然在增长，因为所有悬而未决的伏笔每多等一个场景，观众的不耐烦就加深一分。这是契诃夫守恒在动力学层面的直接体现：时间流逝本身就是一种压力。

更进一步，$h_{\text{idle}}$ 是**递增的**（因为 $f(x) = x^\beta$ 严格凸），这意味着拖延的代价是加速增长的——等第 10 个场景比等第 5 个场景的每场景代价更高。

### 6.3 三幕同构

经典编剧理论（Field, Snyder, McKee 等）描述了一个几乎普遍的叙事节奏模式——三幕结构。我们现在证明，叙事熵 $H(G, n)$ 在三幕结构中的行为与经典编剧理论的经验描述精确对应。

**命题 3**（熵-节拍对应）。设植入率 $r_+(n)$ 和解决率 $r_-(n)$ 满足以下定性模式（总长度 $n_{\max}$ 场景）：

| 阶段 | 场景区间 | 植入率 $r_+$ | 解决率 $r_-$ | 熵行为 |
|------|---------|------------|------------|--------|
| Act I | $[0, 0.25n_{\max}]$ | 高 | 低 | $H$ 单调增 |
| Act II-a | $(0.25, 0.50]n_{\max}$ | 中 | 低 | $H$ 加速增（含 aging） |
| Act II-b | $(0.50, 0.75]n_{\max}$ | 低 | 低 | $H$ 因 aging 持续增至极大 |
| Act III | $(0.75, 1.0]n_{\max}$ | 低 | 高 | $H$ 急剧下降 |

```
       H(G,n)
        │
        │               ╭─╮  ← "All Is Lost" = argmax H
        │             ╱     ╲
        │           ╱         ╲
        │         ╱             ╲
        │       ╱                 ╲
        │     ╱     "Midpoint"      ╲
        │   ╱                         ╲___
        │ ╱                                ╲
        └────────────────────────────────────→ n
       0     0.25    0.50     0.75    1.0
            Act I    Act II          Act III
```

*证明*。

**Act I**（开场/设定）：此阶段以世界建设和角色引入为主，$r_+(n)$ 高（大量新 Thread、Secret、Goal 被植入），$r_-(n) \approx 0$（几乎不解决任何问题——问题才刚被提出）。因此 $h(n) = \Delta H_{\text{plant}} + h_{\text{idle}} > 0$，$H$ 单调增。

**Act II-a**（上升）：植入率降低至中等（世界已经建立，但仍有新的复杂化引入），解决率仍然低。$h(n)$ 主要由 $h_{\text{idle}}$ 驱动——Act I 植入的大量 Thread 的 age 在增长，且因 $\beta > 1$，增速加快。$H$ 加速增长。

**Act II-b**（危机积累）：植入率降至低水平（所有棋子已经就位），解决率仍然低（最大的问题被推迟到 Act III）。此时 $h(n) \approx h_{\text{idle}}(n)$，由引理 1，$h_{\text{idle}}$ 本身递增，$H$ 持续攀升至极大值。极大值对应 Snyder 模型中的 "All Is Lost" 节拍——所有问题汇聚但尚未解决。

**Act III**（解决）：大规模集中解决。$r_-(n) \gg r_+(n) + h_{\text{idle}}(n)$——Thread 被 paid，Secret 被 revealed，Conflict 被 resolved。$H$ 急剧下降，在故事结束时趋向低值（但通常不归零——优秀的故事会留一个小小的开放结局）。$\square$

**Remark 9**。命题 3 的意义不在于「证明」三幕结构——它的前提（植入率/解决率的分布模式）本质上已经编码了三幕节奏。其价值在于建立**翻译关系**：编剧理论中的经验性描述（「第一幕建立世界，第二幕制造危机，第三幕解决冲突」）可以精确映射为熵动力学的定量语言。这使得原本模糊的节奏直觉变得可计算、可监控。

一个更深的问题是：给定超线性兑现压力（$\beta > 1$）和观众参与度最大化目标（Conjecture 4），能否**推导出**最优的 $r_+(n), r_-(n)$ 策略恰好是三幕模式？这将是非循环的证明，但需要 Conjecture 4 的实验验证作为前提，留作未来工作。

### 6.4 非平衡热力学类比

叙事系统更接近**非平衡热力学**中的耗散结构（Prigogine[^3]）而非平衡态系统。我们详细展开这一类比：

**封闭 vs 开放系统**。经典热力学研究封闭系统——无物质交换，能量最终均匀分布（热寂）。叙事系统是**开放系统**：它持续有外部输入（玩家动作、新 Thread 植入、LLM 生成的事件）和输出（Thread 兑现、角色退场、冲突解决），因此不趋向热寂（$H = 0$）。

**远离平衡态的有序涌现**。Prigogine 的核心洞察是：远离平衡态的开放系统可以自发形成有序结构——所谓的**耗散结构**（dissipative structures）。经典例子包括 Bénard 对流格（液体在温差下自发形成六边形对流胞元）和 Belousov-Zhabotinsky 反应（化学反应自发形成时空振荡模式）。

在叙事语境中，类似的涌现现象对应**涌现情节**（Emergent Plot）：在压力函数的驱动下，LLM 可能自发生成出设计者未预期但叙事上合理的情节发展。例如，当两个具有冲突目标的角色被空间力聚集在同一场景中（$P_\gamma$ 激活），而同时有一个高张力的秘密即将被揭示（$P_\sigma$ 高），LLM 可能将两者结合，生成一个戏剧性的「在对峙中揭露真相」的场景——这不是任何规则直接规定的，而是多个压力交汇的自然结果。

**稳态非零熵产**。平衡态系统的熵产为零。但健康的叙事维持非零的稳态熵产率 $h(n) \approx h^*$——这对应持续的悬念与张力。一部完全没有悬念的故事（$h = 0$）和一部完全混乱的故事（$h \to \infty$）都不是好故事。最优的 $h^*$ 值对应 Conjecture 4 中的参与度高原区。

## 7. 猜想与开放问题

基于前述理论框架，我们提出六条猜想。每条猜想都伴随着动机分析、与已知理论的联系、以及潜在的检验方案。

### 7.1 Conjecture 1：叙事 Le Chatelier 原理

**陈述**。设叙事图 $G$ 处于压力准平衡态 $\mathbf{p}^*$。若在场景 $n_0$ 施加外部扰动 $\delta \mathbf{p}$（例如玩家的意外行动引入新的 Thread 或破坏一段关系），则系统在后续 $k$ 个场景内的自主演化方向将**部分抵消**该扰动：

$$\langle \mathbf{p}(n_0 + k) - \mathbf{p}^*, \;\delta\mathbf{p} \rangle < 0 \quad \text{for sufficiently large } k \tag{C1}$$

**动机**。热力学中的 Le Chatelier 原理指出：处于平衡态的系统受扰动后，其自发演化方向倾向于抵消扰动。在叙事语境中，这意味着：

- 突然引入大量新 Thread（熵激增）→ 张力压力 $P_\theta$ 切换到释放模式 → 系统加速解决一些旧问题 → 熵回落。
- 突然解决所有冲突（熵骤降）→ 张力压力 $P_\theta$ 切换到升温模式 → 系统倾向于引入新的 complication → 熵回升。

这保证了叙事系统天然抗拒极端状态。

**与控制论的联系**。Le Chatelier 原理在控制论中对应**负反馈**（negative feedback）。$P_\theta$ 的设计本质上是一个 bang-bang 控制器[^17]——当偏差超过阈值时切换方向。Conjecture 1 断言此控制器在更一般的扰动下仍然有效。

即系统的演化方向与扰动方向的内积为负——系统**反向**响应扰动，趋向恢复均衡。注意此处要求严格小于零，不仅仅是小于 $\|\delta\mathbf{p}\|^2$（后者对任何渐近稳定系统平凡成立）。

**可检验性**。在受控实验中（固定 LLM、固定叙事人格参数），记录扰动前后的 $H(G, n)$ 轨迹，验证 $\text{Corr}(\Delta H_{\text{post}}, \Delta H_{\text{perturb}}) < 0$。

### 7.2 Conjecture 2：谱间隙与叙事聚合度

**陈述**。叙事图 $G$ 的 Fiedler 值 $\lambda_2(L_\omega)$ 与主观叙事聚合度评分 $Q(G) \in [0, 1]$（由人类评估者给出的「故事是否感觉紧凑/聚焦」评分）之间存在正相关：

$$Q(G) = f(\lambda_2) + \varepsilon, \quad f \text{ 单调递增}, \quad \varepsilon \sim \mathcal{N}(0, \sigma^2) \tag{C2}$$

**动机**。谱图论中，$\lambda_2$ 衡量图的连通强度（参见 §2.2）。具有小 $\lambda_2$ 的叙事图对应「散装叙事」——多条情节线互不关联，缺乏叙事统一感。具有大 $\lambda_2$ 的叙事图对应「织网叙事」——角色、秘密、伏笔相互牵连，牵一发而动全身。

**已有间接证据**。在网络科学中，社交网络的 $\lambda_2$ 与社区凝聚力正相关[^18]。如果叙事图的 $\lambda_2$ 与叙事聚合度的相关性得到验证，它将提供一个完全客观的、可自动计算的叙事质量指标——无需人工评估。

**推论**。$\lambda_2$ 可作为叙事聚合度的实时监控指标。当 $\lambda_2$ 降至阈值以下时，系统应建议引入跨线连接（如让不同子线的角色相遇、或让一个秘密关联到多条伏笔线）。

### 7.3 Conjecture 3：叙事相变

**陈述**。定义**叙事密度** $\rho(G) = |E|\,/\,|V|^2$。存在临界密度 $\rho_c$ 和临界指数 $\nu > 0$，使得叙事聚合度 $Q$ 在 $\rho_c$ 附近发生相变：

$$Q(\rho) \sim \begin{cases} 0 & \text{if } \rho < \rho_c \\ (\rho - \rho_c)^\nu & \text{if } \rho \geq \rho_c \end{cases} \tag{C3}$$

**动机**。这直接类比渗流理论（Percolation Theory）中的相变。渗流理论研究的是：在一个网格中随机占据格点，当占据概率 $p$ 超过临界值 $p_c$ 时，突然出现一个贯穿整个网格的连通路径——这被称为**渗流阈值**，是一个二阶相变[^19]。

在叙事语境中的对应：

- **亚临界相** $(\rho < \rho_c)$：叙事呈碎片化状态。多个独立子图对应互不关联的情节碎片。玩家感受为「发生了很多事但没有主线」。这是长会话 AI RP 的常见退化模式——随着会话进行，新引入的角色和事件没有与既有的叙事网络建立足够的连接。
- **超临界相** $(\rho > \rho_c)$：出现跨越大部分节点的连通分量（巨组分）。任何局部事件都通过边传导影响全局。叙事感知为「紧凑、有主线」。
- **超密相** $(\rho \gg \rho_c)$：过度连接导致每个动作触发连锁反应，叙事失控。可能对应另一个相变点 $\rho_c'$，超过后聚合度反而下降（信息过载）。

临界指数 $\nu$ 的值取决于叙事图的拓扑类，这与底层力的类型分布有关。Erdős–Rényi 随机图的渗流相变给出 $\nu = 1$（均场近似）；叙事图由于力的类型约束（§3.3），可能属于不同的普适类（universality class）。确定 $\nu$ 的值是一个开放的理论问题。

### 7.4 Conjecture 4：熵产率的上下界

**陈述**。存在叙事参与度函数 $\text{Eng}(n) \in [0, 1]$（Engagement，可由用户行为指标估计——如回复长度、响应延迟的倒数、情感词汇密度），使得：

$$\text{Eng}(n) \approx \exp\!\left(-\frac{(h(n) - h^*)^2}{2\sigma_h^2}\right) \tag{C4}$$

其中 $h^* = (h_{\min} + h_{\max})/2$ 为最优熵产率，$\sigma_h$ 控制容忍宽度。这是一个**倒 U 形**函数，具有以下行为：

- $h(n) \ll h^*$（熵产率过低）：叙事停滞，参与度趋零。对应「什么都没发生」。
- $h(n) \gg h^*$（熵产率过高）：信息过载，参与度趋零。对应「发生了太多事、跟不上」。
- $h(n) \approx h^*$（最优区间 $h^* \pm \sigma_h$）：参与度处于峰值区。

**动机**。认知心理学中的 Yerkes-Dodson 定律[^4]描述了刺激强度与表现/参与度之间的**倒 U 形**关系：太低的刺激导致无聊（under-arousal），太高的刺激导致焦虑（over-arousal），最优区间对应心流状态（Csikszentmihalyi[^5]）。

高斯函数的选择直接对应倒 U 形：$h = h^*$ 时参与度最大，向两端对称衰减。$\sigma_h$ 的物理含义是受众对节奏偏差的容忍度——悬疑类受众可能有较小的 $\sigma_h$（对节奏敏感），日常类受众可能有较大的 $\sigma_h$（对节奏宽容）。

**推论**。最优叙事策略是维持 $h(n)$ 在 $h^* \pm \sigma_h$ 附近振荡。张力压力 $P_\theta$（§5.1-V）正是为此设计的自稳定器。$h^*$ 和 $\sigma_h$ 的具体数值需要通过实验确定，可能因叙事类型（冒险、悬疑、日常）和受众偏好而异。

### 7.5 Conjecture 5：契诃夫阈值的最优性

**陈述**。定律 IV 的有界弱化版本中，阈值 $\alpha^*$ 的最优取值满足：

$$\alpha^* = \underset{\alpha}{\arg\min}\; \Big[\mathbb{E}_{\mathcal{N}}\big[\text{FP}(\alpha)\big] + \gamma \cdot \mathbb{E}_{\mathcal{N}}\big[\text{FN}(\alpha)\big]\Big] \tag{C5}$$

其中 $\text{FP}(\alpha)$ 为假阳性率（正在发展中的伏笔被误判为陈旧——过早触发兑现压力，导致仓促回收），$\text{FN}(\alpha)$ 为假阴性率（真正被遗忘的伏笔未被检出——兑现压力不够，伏笔蒸发），$\gamma$ 为相对代价权重，$\mathcal{N}$ 为叙事分布。

**进一步猜测**。$\alpha^*$ 与 Thread 的权重类型 $\mathcal{W}$ 有关：

$$\alpha^*(w) \propto w^{-1/\beta}$$

即高权重伏笔（major）的容忍阈值更低（更早触发兑现压力），低权重伏笔（subtle）可以潜伏更久。这与观众期望一致：主线伏笔的回收应比支线更及时。一部剧集可以在第一集埋一个角落的暗示，直到最后一季才回收（subtle, $\alpha$ 大）；但主角的核心冲突不能拖太久（major, $\alpha$ 小）。

### 7.6 Conjecture 6：Lyapunov 收敛性

**陈述**。定义目标熵 $H^*$ 和 Lyapunov 候选函数：

$$V(G, n) = \frac{1}{2}\big(H(G,n) - H^*\big)^2 \tag{C6}$$

若张力压力 $P_\theta$ 的参数满足 $\delta_+, \delta_- > 0$ 且 $H_{\text{low}} < H^* < H_{\text{high}}$，则 $V$ 在 $P_\theta$ 的调控下是 Lyapunov 函数：

$$\Delta V(n) = V(G, n+1) - V(G, n) < 0 \quad \text{whenever } |H(G,n) - H^*| > \varepsilon \tag{C6'}$$

其中 $\varepsilon$ 为与 $\delta_\pm$ 和压力函数的 Lipschitz 常数相关的正常数。

**含义**。这保证了叙事熵的**轨道稳定性**——在张力压力的调控下，$H(G,n)$ 不会永久偏离目标区间，而是在 $H^* \pm \varepsilon$ 的邻域内振荡。叙事既不会永远平淡（$H \to 0$），也不会永远混乱（$H \to \infty$），而是被吸引到一个稳定的**极限环**（limit cycle）。

**证明思路**。当 $H > H^* + \varepsilon$ 时，$P_\theta < 0$ 产生释放压力，$h(n)$ 中出现负项使 $H$ 下降，故 $\Delta V < 0$。对称地，当 $H < H^* - \varepsilon$ 时，$P_\theta > 0$ 产生升温压力使 $H$ 上升。严格证明需要两个技术条件：

1. **Lipschitz 连续性**：压力函数的单步变化量有界，确保 $P_\theta$ 的调控不会因其他压力的剧烈变化而被淹没。
2. **Thread age 有界性**：需要假设系统存在某种 Thread 管理策略（如有界契诃夫守恒 $\Diamond_{\leq \alpha}$），使得 $\max_{t \in \mathcal{T}_{\text{open}}} \text{age}(t) \leq \alpha$。此条件保证 $h_{\text{idle}}$ 有界（$h_{\text{idle}} \leq |\mathcal{T}_{\text{open}}| \cdot [(\alpha+1)^\beta - \alpha^\beta] \cdot w_{\max}$），从而 $\delta_\pm$ 只需超过一个有限的下界。若缺少此条件，$h_{\text{idle}}$ 随 age 无界增长（引理 1），有限的 $\delta_\pm$ 最终无法抵消自然熵增——直觉上，一个无限拖延的伏笔会产生无限压力，任何恒温器都无法补偿。

**与 RimWorld Storyteller 的联系**。RimWorld[^9] 的 AI Storyteller 系统实现了一个类似的机制（尽管不是用 Lyapunov 理论设计的）：游戏根据殖民地的「财富值」和「威胁点数」动态调整事件的频率和强度，使游戏体验维持在一个「有挑战但不压倒」的区间。Conjecture 6 为这类系统提供了理论基础。

## 8. 案例研究

为了具体演示框架的运作，我们追踪一个完整的叙事片段，从初始化到压力释放，计算每一步的守恒验证、压力值和熵变化。

### 8.1 初始化 ($n = 0$)

```
CREATE  Character  alice   {traits: [curious, stubborn]}
CREATE  Character  bob     {traits: [secretive, protective]}
CREATE  Setting    tavern  {atmosphere: mysterious}

LINK    alice ──AT──> tavern
LINK    bob   ──AT──> tavern
LINK    alice ──TRUSTS[8]──> bob
LINK    bob   ──LOVES[rom]──> alice
```

**守恒验证**：
- 定律 I: ✓（alice AT tavern, bob AT tavern，各一条 AT 边）
- 定律 II: ✓（尚无事件需要验证）
- 定律 III: ✓（尚无秘密）
- 定律 V: ✓（尚无目标）

$H(G_0) = 0$。无开放 Thread、无 Secret、无冲突 Goal、关系边（TRUSTS, LOVES）虽存在但 $P_\rho = 0$（关系已定义，不产生关系压力）。

叙事图此时有 3 个节点（2 角色 + 1 场景）、4 条边。$\lambda_2 > 0$（图连通）。

### 8.2 植入 ($n = 1$)

```
CREATE  Secret  σ₁  {content: "Bob killed Alice's father", tension: 9}
CREATE  Thread  t₁  {name: "父亲之死", weight: major, planted_at: 1}
CREATE  Goal    g₁  {target: truth, intensity: 7, holder: alice}

LINK  bob  ──KNOWS[1.0]──> σ₁
LINK  bob  ──HIDES[from:{alice}]──> σ₁
LINK  t₁   ──INVOLVES[seeker]──> alice
LINK  t₁   ──INVOLVES[perpetrator]──> bob
LINK  alice ──WANTS──> g₁
```

**守恒验证**：定律 I–V 全部满足。

**压力计算**（设 $\beta = 1.5$，$w_{\text{major}} = 2$，$\alpha_s = 0.3$，$|\mathcal{C}| = 2$）：
- $P_\tau(t_1, 1) = \text{age}^{1.5} \cdot 2 = 1^{1.5} \cdot 2 = 2.0$（$\text{age} = 1 - 1 + 1 = 1$）
- $P_\sigma(\sigma_1, G_1) = 9 \cdot \frac{1 \cdot (2-1)}{2} \cdot (1 + 0) = 4.5$（1 个知晓者，0 个怀疑者，信息不对称因子 $= 1/2$）
- $P_\gamma = 0$（目前无冲突 Goal 对）
- $P_\rho = 0$

$$H(G_1) = 2.0 + 4.5 = 6.5, \quad h(1) = +6.5$$

从叙事状态 $H = 0$ 跃升至 $H = 6.5$——这是一个典型的 Act I 「Catalyst」节拍：一个重大的戏剧性问题被引入。

### 8.3 压力累积 ($n = 2 \to 5$)

| $n$ | 叙事事件 | $P_\tau(t_1)$ | $P_\sigma(\sigma_1)$ | $H(G_n)$ | $h(n)$ |
|-----|---------|:------------:|:-------------------:|:---------:|:------:|
| 2 | （空转） | $2^{1.5} \cdot 2 \approx 5.7$ | 4.5 | 10.2 | +3.7 |
| 3 | （空转） | $3^{1.5} \cdot 2 \approx 10.4$ | 4.5 | 14.9 | +4.7 |
| 4 | Alice SUSPECTS $\sigma_1$ | $4^{1.5} \cdot 2 = 16.0$ | $9 \cdot \frac{1}{2} \cdot 1.3 = 5.9$ | 21.9 | +7.0 |
| 5 | （空转） | $5^{1.5} \cdot 2 \approx 22.4$ | 5.9 | 28.3 | +6.4 |

**观察**：

1. $n = 2, 3$ 的空转场景中 $h > 0$（引理 1），且 $h$ 递增（$3.7 \to 4.7$）——这正是自然熵增的超线性效应。
2. $n = 4$ 时，Alice 开始怀疑秘密。SUSPECTS 边使怀疑者项激活（$\alpha_s = 0.3$），$P_\sigma$ 从 4.5 升至 5.9。同时 $h(4) = 7.0$ 是最大的单场景熵增——因为同时有 aging 效应和新的 SUSPECTS 边。注意 $P_\tau$ 仍是主要的熵贡献源——伏笔的时间压力增长速度远快于秘密压力。
3. 到 $n = 5$，$H \approx 28$。如果我们设 $H_{\text{high}} = 25$，则张力压力 $P_\theta$ 已经激活释放模式，建议系统考虑解决一些悬而未决的问题。

### 8.4 压力释放 ($n = 6$)

```
REVEAL  σ₁  to: {alice}    → UPDATE σ₁ {revealed: true}; LINK alice ──KNOWS──> σ₁
UPDATE  t₁  {status: paid}
DELETE  alice ──TRUSTS[8]──> bob
LINK    alice ──HATES[betrayal]──> bob
```

**压力变化**：
- $P_\tau(t_1) \to 0$（Thread 已 paid，移出 $\mathcal{T}_{\text{open}}$）
- $P_\sigma(\sigma_1) \to 0$（$\text{revealed} = \texttt{true}$，指示函数归零；且 $|\mathcal{K}| = |\mathcal{C}| = 2$，信息不对称因子 $= 0$）
- 新增关系压力：alice HATES bob 与 bob LOVES alice 形成情感矛盾

$$H(G_6) \approx P_\rho(\text{alice, bob}) > 0$$

$$\Delta H(6) \approx -(22.4 + 5.9) + P_\rho \ll 0$$

$H$ 从 $\approx 28$ 骤降至接近 $P_\rho$。这是一个典型的 Act III 特征：**主线压力的集中释放，伴随着新的、更深层的压力注入**——bob 的爱与 alice 的恨形成的情感张力成为新的叙事驱动力，为续篇埋下种子。

### 8.5 Laplacian 分析

追踪整个过程中 $\lambda_2$ 的变化：

| 阶段 | $\|V\|$ | $\|E\|$ | $\lambda_2$ 趋势 | 解释 |
|------|---------|---------|-----------------|------|
| $n = 0$ | 3 | 4 | 中等 | 小型紧密图 |
| $n = 1$ | 6 | 9 | 增大 | 新节点（$\sigma_1, t_1, g_1$）紧密连接到既有网络 |
| $n = 4$ | 6 | 10 | 略增 | 新增 SUSPECTS 边增强连通 |
| $n = 6$ | 6 | 11 | 变化 | +1 KNOWS（REVEAL），−1 TRUSTS +1 HATES |

这个案例展示了一个关键现象：$\lambda_2$ 在叙事早期（植入阶段）增大——因为新元素与既有网络建立了多重连接。这与 Conjecture 2 一致：高 $\lambda_2$ 对应「紧凑」的叙事感知。

## 9. 相关工作

### 9.1 编剧理论与叙事结构

古典叙事理论为本框架提供了经验基础。Aristotle《诗学》[^20] 提出了叙事的基本要素（mythos, ethos, dianoia, lexis, melos, opsis），其中 mythos（情节）的优先性对应我们将 Event 和 Thread 作为核心粒子类型的决策。Propp[^16] 对俄罗斯民间故事的形态学分析揭示了 31 种基本叙事功能——这可以视为我们类型系统的早期原型。

现代编剧理论提供了更具操作性的模型。Snyder[^2] 的三幕节拍结构为宏观叙事熵波形提供了经验基准（§6.3 命题 3）。Weiland[^6] 的角色弧线模型（Lie → Truth）可直接编码为 Character 粒子的 arc 属性。Swain[^1] 的 Scene-Sequel 模型——场景（目标-冲突-灾难）后接续场（反应-困境-决定）——启发了张力压力 $P_\theta$ 的交替自稳定设计。McKee[^7] 的冲突层级理论（内部冲突 → 人际冲突 → 外部冲突）对应了我们压力函数的多尺度结构。

### 9.2 涌现叙事系统

Dwarf Fortress[^8] 的世界模拟验证了「简单规则产生复杂涌现」的可行性——它通过模拟地质、气候、文明和个体行为，产生了极其丰富的叙事。但它缺乏对叙事节奏的主动控制。

RimWorld[^9] 的 AI Storyteller 系统与我们的张力压力 $P_\theta$ 具有相似的自稳定目标——它维护一个「威胁点数」系统来控制事件频率，但缺乏形式化的压力传导和熵度量。其三种预设 Storyteller 人格（Cassandra/Randy/Phoebe）可以视为 $P_\theta$ 的不同参数化实例。

Façade[^10] 的 Drama Manager 使用了显式的张力曲线管理——它在实时交互中追踪张力值并选择叙事节拍。这是最接近本文框架精神的先前系统，但其规则是硬编码的，缺乏本文提出的基于图结构的自适应性。

### 9.3 计算叙事学

Mani[^11] 的 Computational Narratology 提供了叙事的形式语言学框架，使用时间逻辑分析叙事的时间结构。Porteous & Cavazza[^12] 的 Plan-based Narrative Generation 使用 AI 规划器驱动叙事，将叙事生成视为规划问题。与本文的「压力驱动」范式相比，规划方法提供了更强的全局一致性保证，但灵活性较低——难以处理玩家的任意行动。两种范式可能互补：规划提供宏观骨架，压力提供微观驱动。

Riedl & Young[^21] 的 Narrative Planning 将叙事生成与角色行为的合理性统一在一个规划框架内。Murray[^22] 的《Hamlet on the Holodeck》从媒介理论角度讨论了交互叙事的设计原则。Ryan[^23] 的 Possible Worlds 理论为叙事的分支结构提供了模态逻辑基础。

### 9.4 图信号处理与网络科学

叙事 Laplacian 的引入连接了本框架与图信号处理（GSP）领域[^13]。Conjecture 2 中的谱间隙-聚合度假说可以在 GSP 的频域分析框架内进一步精化——高频分量对应局部冲突，低频分量对应全局叙事主题。

网络科学中的社区检测算法[^18] 可以直接应用于叙事图，识别相对独立的子情节——这为 Conjecture 3（叙事相变）提供了算法工具。

### 9.5 LLM 与交互叙事

近年来，LLM 在交互叙事中的应用爆发式增长（AI Dungeon, NovelAI, Character.AI 等），但学术研究相对滞后。大多数工作集中在 prompt engineering 层面（如 world book 设计、system prompt 优化），缺乏理论框架来分析这些工程决策的效果和极限。本文试图填补这一空白。

## 10. 讨论

### 10.1 局限性

本框架存在以下已知局限：

**粒子类型的完备性**。我们定义了六种粒子类型，但这一选择是否**完备**（即能否覆盖所有叙事元素）？例如，「象征」（Symbol）——一把不是用来开火而是用来象征暴力的枪——在当前框架中没有直接对应的粒子类型。它可以勉强编码为一个 Thread，但这并不自然。可能需要额外的类型来处理主题层面的叙事元素。

**压力函数的参数化**。$P_\tau$ 中的 $\beta$，$P_\sigma$ 中的 $\alpha_s$，$P_\theta$ 中的 $\delta_\pm, H_{\text{low}}, H_{\text{high}}$ 等参数在本文中是自由参数。它们的最优值可能因叙事类型（冒险 vs 日常 vs 悬疑）和受众偏好而异。建立参数选择的理论或经验指导是重要的未来工作。

**LLM 的压力响应函数**。本框架假设压力信号可以有效地引导 LLM 的生成方向，但未量化 LLM 对压力信号的实际响应函数 $g: \mathbb{R}_{\geq 0} \to [0, 1]$（压力 → 处理概率）。这个函数的形状——是线性的？Sigmoid 的？有阈值的？——对整个动力学系统的行为至关重要。

### 10.2 假设与适用范围

本框架的核心假设包括：

1. **可图化假设**：叙事状态可以被有意义地表示为有向属性图。这对结构化的叙事（有明确角色、地点、情节线的故事）成立，但对高度抽象或意识流的叙事可能不适用。
2. **可加性假设**：叙事熵 $H$ 是各压力分量的简单加和。实际上，不同压力之间可能存在非线性交互——例如，一个高张力秘密在两个冲突角色共同在场时可能产生超过线性和的效果。将 $H$ 推广为包含交叉项的非线性函数是一个自然的扩展方向。
3. **Markov 假设**：压力传导只依赖当前图状态，不依赖历史路径。这对短期分析是合理的近似，但长期叙事可能存在路径依赖——一段曲折的关系发展历程与一段平淡的历程，即使当前状态相同，叙事意义也不同。

### 10.3 与 CRPG 叙事系统的关系

计算机角色扮演游戏（CRPG）的叙事系统是本框架最直接的应用场景之一。经典 CRPG 使用分支叙事图（branching narrative graph）——设计者预先定义所有可能的分支和汇合点。这种方法保证了叙事质量但限制了玩家自由度，且内容创作成本随分支数指数增长。

本框架提出的压力驱动范式提供了一种替代方案：不预先定义分支，而是定义叙事的「物理环境」（守恒定律和压力函数），让具体情节在这个环境中涌现。这从根本上改变了叙事系统的设计范式——从「内容创作」到「规则设计」，类似于从手工动画到物理模拟的转变。

## 11. 结论与未来方向

本文提出的叙事物理学框架试图回答一个根本性问题：**交互式叙事的结构性质是否可以用物理系统的数学语言精确刻画？**

我们的初步回答是肯定的。守恒定律提供了一致性的不变量（§4），压力函数族提供了演化的驱动力（§5），叙事熵提供了宏观状态的度量（§6），Laplacian 谱分析提供了结构性质的代数刻画（§5.3–5.4）。六条猜想分别涉及稳定性（C1, C6）、结构性质（C2, C3）、信息论界（C4）和算法优化（C5），覆盖了从理论基础到工程实践的不同层次。

大量工作仍有待完成。以下列出最优先的方向：

1. **实验验证**：在受控的 LLM 交互叙事环境中验证六条猜想，特别是 Conjecture 2（谱间隙与聚合度的相关性，可通过人类评估实验检验）和 Conjecture 4（熵产率与参与度的关系，可通过用户行为数据分析）。
2. **参数理论**：建立压力函数参数（$\beta, \alpha_s, \delta_\pm$ 等）的选择理论——是否存在普适的最优参数，还是必须针对叙事类型和受众定制？
3. **非线性扩展**：将叙事熵从线性加和推广为包含交叉项的非线性函数，建模压力之间的协同和抵消效应。
4. **临界指数分类**：确定 Conjecture 3 中叙事相变的临界指数 $\nu$，以及叙事图所属的普适类。
5. **LLM 响应函数**：量化 LLM 对压力信号的响应函数，建立从「叙事物理量」到「生成概率分布」的映射。

框架的核心直觉——**好故事不是规定出来的，是在正确的物理环境中自然生长出来的**——至少提供了一个值得严肃对待的研究方向。如果叙事确实服从某种「物理学」，那么理解这些定律将不仅有助于构建更好的 AI 叙事系统，还可能加深我们对叙事本身——这一人类最古老的认知技术——的理解。

---

## 参考文献

[^1]: Swain, D. *Techniques of the Selling Writer*. University of Oklahoma Press, 1981.
[^2]: Snyder, B. *Save the Cat! The Last Book on Screenwriting You'll Ever Need*. Michael Wiese Productions, 2005.
[^3]: Prigogine, I. & Stengers, I. *Order Out of Chaos: Man's New Dialogue with Nature*. Bantam Books, 1984.
[^4]: Yerkes, R.M. & Dodson, J.D. "The Relation of Strength of Stimulus to Rapidity of Habit-Formation." *Journal of Comparative Neurology and Psychology*, 18(5), 459–482, 1908.
[^5]: Csikszentmihalyi, M. *Flow: The Psychology of Optimal Experience*. Harper & Row, 1990.
[^6]: Weiland, K.M. *Creating Character Arcs*. PenForASword Publishing, 2016.
[^7]: McKee, R. *Story: Substance, Structure, Style and the Principles of Screenwriting*. ReganBooks, 1997.
[^8]: Adams, T. *Dwarf Fortress*. Bay 12 Games, 2006–.
[^9]: Sylvester, T. *Designing Games: A Guide to Engineering Experiences*. O'Reilly Media, 2013.
[^10]: Mateas, M. & Stern, A. "Façade: An Experiment in Building a Fully-Realized Interactive Drama." *Game Developers Conference*, 2003.
[^11]: Mani, I. *Computational Narratology*. In: Hühn, P. et al. (eds) *The Living Handbook of Narratology*. Hamburg University, 2013.
[^12]: Porteous, J. & Cavazza, M. "Controlling Narrative Generation with Planning Trajectories." *Proceedings of ICIDS*, 2009.
[^13]: Shuman, D.I. et al. "The Emerging Field of Signal Processing on Graphs." *IEEE Signal Processing Magazine*, 30(3), 83–98, 2013.
[^14]: Fiedler, M. "Algebraic Connectivity of Graphs." *Czechoslovak Mathematical Journal*, 23(2), 298–305, 1973.
[^15]: Pnueli, A. "The Temporal Logic of Programs." *Proceedings of the 18th Annual Symposium on Foundations of Computer Science*, 46–57, 1977.
[^16]: Propp, V. *Morphology of the Folktale*. University of Texas Press, 1968. (Original Russian edition, 1928.)
[^17]: Athans, M. & Falb, P.L. *Optimal Control: An Introduction to the Theory and Its Applications*. McGraw-Hill, 1966.
[^18]: Newman, M.E.J. "Modularity and Community Structure in Networks." *Proceedings of the National Academy of Sciences*, 103(23), 8577–8582, 2006.
[^19]: Stauffer, D. & Aharony, A. *Introduction to Percolation Theory*. Taylor & Francis, 1994.
[^20]: Aristotle. *Poetics*. Translated by S.H. Butcher. Dover Publications, 1997. (Original circa 335 BCE.)
[^21]: Riedl, M.O. & Young, R.M. "Narrative Planning: Balancing Plot and Character." *Journal of Artificial Intelligence Research*, 39, 217–268, 2010.
[^22]: Murray, J.H. *Hamlet on the Holodeck: The Future of Narrative in Cyberspace*. MIT Press, 1997.
[^23]: Ryan, M.-L. *Possible Worlds, Artificial Intelligence, and Narrative Theory*. Indiana University Press, 1991.
[^24]: Stevens, S.S. "On the Psychophysical Law." *Psychological Review*, 64(3), 153–181, 1957.
