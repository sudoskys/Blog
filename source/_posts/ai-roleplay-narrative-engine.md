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

## 摘要

LLM 驱动的交互叙事在长会话中面临世界状态漂移与叙事节奏退化两个结构性问题。本文提出 **Narrative Physics** 框架，将叙事状态形式化为有向属性图 $G=(V,E,\phi)$，在其上定义五条守恒定律作为一致性不变量，引入压力函数族 $\mathcal{P}=\{P_\tau,P_\sigma,P_\gamma,P_\rho,P_\theta\}$ 刻画叙事演化的驱动力，并通过叙事熵 $H(G)$ 度量宏观叙事状态。我们进一步利用叙事图的 Laplacian 谱分析建立压力传导速率的上界，提出叙事相变假说（Conjecture 3）以解释长叙事中的一致性崩塌现象，并给出契诃夫守恒判定问题的计算复杂度下界。全文共提出 6 条猜想，涵盖谱间隙、熵产率、Le Chatelier 稳定性、相变临界指数与 Lyapunov 收敛性，为交互叙事的形式化理论提供初步框架。

## 1. 引言

### 1.1 问题动机

大语言模型本质上是条件概率分布 $p(x_t\mid x_{<t})$，不维护显式的世界状态表示。设上下文窗口容量为 $W$ tokens，叙事世界状态的 Kolmogorov 复杂度为 $K(S)$。当 $K(S) > W$ 时，状态漂移不可避免——某些世界事实必须被截断或有损压缩。更严格地说，设压缩映射为 $\pi: S \to \hat{S}$，其失真度 $d(S, \hat{S}) = |S \triangle \hat{S}|$（对称差的势），则：

$$\mathbb{E}[d(S,\hat{S})] \geq K(S) - W \quad \text{（信息论下界）} \tag{1}$$

这意味着状态漂移不是实现缺陷，而是**信息论必然**。任何基于有限窗口的方案——无论是摘要压缩、世界书注入还是 RAG 检索——都无法将 $d$ 压至零。

### 1.2 贡献概述

本文的贡献分为三个层次：

1. **建模层**（§2）：将叙事状态形式化为带类型的有向属性图，定义粒子-力本体论。
2. **约束层**（§3）：在叙事图上建立五条守恒定律，并分析其判定的计算复杂度。
3. **动力学层**（§4–6）：定义压力函数族、叙事熵、Laplacian 谱分析、相变假说与 Lyapunov 稳定性分析，提出 6 条猜想。

## 2. 叙事图：形式化模型

### 2.1 预备知识与符号

**符号约定**。全文使用以下记号：

| 符号 | 含义 |
|------|------|
| $G = (V, E, \phi)$ | 有向属性图（叙事图） |
| $V$ | 节点集，$\|V\| = N$ |
| $E \subseteq V \times \mathcal{L} \times V$ | 带标签有向边集 |
| $\mathcal{L}$ | 边标签集（力类型） |
| $\phi: E \to \mathcal{A}$ | 属性函数 |
| $A(G)$ | 邻接矩阵 |
| $D(G)$ | 度矩阵 |
| $L(G) = D - A$ | 图 Laplacian |
| $0 = \lambda_1 \leq \lambda_2 \leq \cdots \leq \lambda_N$ | $L$ 的特征值谱 |
| $\lambda_2(G)$ | Fiedler 值（代数连通度） |
| $n$ | 当前场景序号 |
| $\Delta n$ | 场景间隔 |

### 2.2 叙事粒子

**定义 1**（粒子类型系统）。节点集 $V$ 上定义类型函数 $\text{type}: V \to \Omega$，其中类型宇宙 $\Omega = \{C, S, T, \Sigma, G, E\}$：

$$V = \bigsqcup_{\omega \in \Omega} V_\omega \quad \text{（不相交并）}$$

| 类型 $\omega$ | 记为 | 语义 | 签名 |
|---------------|------|------|------|
| $C$ | $\mathcal{C}$ | 角色（Character） | $(\text{id}, \text{traits}: 2^{\mathbb{S}}, \text{arc}: \mathbb{S} \rightharpoonup \mathbb{S} \times [0,1])$ |
| $S$ | $\mathcal{S}$ | 场景（Setting） | $(\text{id}, \text{atmosphere}: \mathbb{S})$ |
| $T$ | $\mathcal{T}$ | 伏笔（Thread） | $(\text{id}, \text{status}: \Xi, \text{weight}: \mathcal{W}, \text{age}: \mathbb{N})$ |
| $\Sigma$ | $\Sigma$ | 秘密（Secret） | $(\text{id}, \text{tension}: [0,10])$ |
| $G$ | $\mathcal{G}$ | 目标（Goal） | $(\text{id}, \text{intensity}: [0,10], \text{status}: \Psi)$ |
| $E$ | $\mathcal{E}$ | 事件（Event） | $(\text{id}, \text{seq}: \mathbb{N}, \text{sig}: \{+,-\})$ |

其中 $\mathbb{S}$ 为字符串域，$\Xi = \{\texttt{planted}, \texttt{developing}, \texttt{paid}, \texttt{abandoned}\}$ 为 Thread 状态空间，$\mathcal{W} = \{\texttt{major}, \texttt{minor}, \texttt{subtle}\}$ 为权重域，$\Psi = \{\texttt{active}, \texttt{achieved}, \texttt{abandoned}, \texttt{blocked}\}$ 为 Goal 状态空间。

**公理 1**（主动性不对称）。定义因果源集 $\text{Src}(\mathcal{F}_{\text{causal}}) = \{v \mid \exists\, (v, \ell, u) \in E,\, \ell \in \mathcal{F}_{\text{causal}}\}$。则：

$$\text{Src}(\mathcal{F}_{\text{causal}}) \subseteq \mathcal{C} \cup \mathcal{E}$$

即只有角色和事件可以作为因果链的源。这排除了「场景导致事件」或「目标导致事件」等非法因果结构，将因果权限锁定在行动者层。

### 2.3 叙事力

**定义 2**（力分类）。边标签集 $\mathcal{L}$ 的五重分划：

$$\mathcal{L} = \mathcal{F}_{\text{sp}} \sqcup \mathcal{F}_{\text{cog}} \sqcup \mathcal{F}_{\text{rel}} \sqcup \mathcal{F}_{\text{cau}} \sqcup \mathcal{F}_{\text{nar}}$$

**定义 3**（力的类型约束）。每种力 $\ell \in \mathcal{L}$ 附带定义域和值域的类型约束 $\text{dom}(\ell) \times \text{cod}(\ell) \subseteq \Omega \times \Omega$：

| 力 $\ell$ | $\text{dom}$ | $\text{cod}$ | 属性 $\phi(\ell)$ |
|-----------|-------------|-------------|-------------------|
| $\texttt{AT}$ | $C$ | $S$ | — |
| $\texttt{KNOWS}$ | $C$ | $\Sigma$ | $\kappa \in [0,1]$（确信度） |
| $\texttt{SUSPECTS}$ | $C$ | $\Sigma$ | $\kappa \in [0,1]$ |
| $\texttt{HIDES}$ | $C$ | $\Sigma$ | $\text{from} \subseteq \mathcal{C}$ |
| $\texttt{TRUSTS}$ | $C$ | $C$ | $\text{level} \in [0,10]$ |
| $\texttt{LOVES}$ | $C$ | $C$ | $\text{type} \in \{\text{rom}, \text{fam}, \text{pla}\}$ |
| $\texttt{HATES}$ | $C$ | $C$ | $\text{reason}: \mathbb{S}$ |
| $\texttt{CAUSED}$ | $C \cup E$ | $E$ | — |
| $\texttt{INVOLVES}$ | $T$ | $C$ | $\text{role}: \mathbb{S}$ |
| $\texttt{WANTS}$ | $C$ | $G$ | — |
| $\texttt{CONFLICTS}$ | $G$ | $G$ | — |

**命题 1**（类型安全性）。若叙事图 $G$ 的每条边 $e = (u, \ell, v)$ 均满足 $\text{type}(u) \in \text{dom}(\ell) \land \text{type}(v) \in \text{cod}(\ell)$，则称 $G$ 是**良类型的**（well-typed）。良类型的叙事图不会产生语义非法的关系（如「秘密信任场景」）。

## 3. 守恒定律

### 3.1 五条定律

在叙事图上定义五条守恒定律，作为合法状态的不变量。关键设计决策：守恒定律的违反不导致系统拒绝（硬约束），而是产生可量化的违反压力（软约束），驱动系统趋向合法状态——这与物理学中「最小作用量原理」的精神一致。

**定律 I**（存在守恒，Conservation of Existence）。

$$\forall\, c \in \mathcal{C}:\; \exists!\, s \in \mathcal{S}:\; (c, \texttt{AT}, s) \in E \tag{I}$$

注意此处使用 $\exists!$（唯一存在量词）：每个角色必须且只能存在于一个空间中。这比简单的 $\exists$ 更强——它排除了角色同时出现在多个位置的叠加态。在数据库层面对应 UNIQUE 约束。

**定律 II**（因果守恒，Conservation of Causality）。

$$\forall\, e \in \mathcal{E} \setminus \{e_0\}:\; \exists\, v \in \mathcal{C} \cup \mathcal{E}:\; (v, \texttt{CAUSED}, e) \in E \tag{II}$$

初始事件 $e_0$ 是唯一的无因元素（公理性存在）。所有后续事件必须可追溯至因果链——我们称此为**接地原则**。

**定律 III**（秘密守恒，Conservation of Secrets）。

$$\forall\, \sigma \in \Sigma:\; |\{c \in \mathcal{C} \mid (c, \texttt{KNOWS}, \sigma) \in E\}| \geq 1 \tag{III}$$

**定律 IV**（契诃夫守恒，Chekhov Conservation）。

$$\forall\, t \in \mathcal{T}:\; \text{status}(t) = \texttt{planted} \implies \Diamond\;\text{status}(t) \in \{\texttt{paid}, \texttt{abandoned}\} \tag{IV}$$

$\Diamond$ 为 LTL（线性时序逻辑）的 eventually 算子。

**定律 V**（目标守恒，Conservation of Goals）。

$$\forall\, g \in \mathcal{G}:\; \text{status}(g) = \texttt{active} \implies \exists\, c \in \mathcal{C}:\; (c, \texttt{WANTS}, g) \in E \tag{V}$$

### 3.2 一致性判定的计算复杂度

**定义 4**（一致性谓词）。

$$\text{CON}(G) \iff \bigwedge_{i \in \{\text{I},\ldots,\text{V}\}} \text{Law}_i(G)$$

**定理 1**（静态一致性的可判定性）。对于定律 I, II, III, V，一致性判定在 $O(|V| + |E|)$ 内可完成（单遍图遍历）。

*证明概要*。各定律均可化归为对特定类型节点的邻接边检查：定律 I 需验证每个 $c \in \mathcal{C}$ 恰好有一条 AT 出边；定律 II 需验证每个 $e \in \mathcal{E} \setminus \{e_0\}$ 至少有一条 CAUSED 入边。这些均为线性时间的图遍历。$\square$

**定理 2**（契诃夫完备性的不可判定性）。设 $\mathcal{N}$ 为叙事图的无穷演化序列 $(G_0, G_1, G_2, \ldots)$，判定「序列 $\mathcal{N}$ 是否满足定律 IV」是 $\Sigma_1^0$-完备的（即与停机问题等价）。

*证明概要*。构造规约：给定图灵机 $M$ 和输入 $x$，构造叙事图序列，其中植入一个 Thread $t_M$，当且仅当 $M(x)$ 停机时 $t_M$ 被标记为 $\texttt{paid}$。则判定 $\text{Law}_\text{IV}(\mathcal{N})$ 需要判定 $M(x)$ 是否停机。$\square$

**推论 1**。在实际系统中，定律 IV 必须弱化为有界时间版本：

$$\forall\, t \in \mathcal{T}:\; \text{status}(t) = \texttt{planted} \implies \Diamond_{\leq \alpha}\;\text{status}(t) \in \{\texttt{paid}, \texttt{abandoned}\}$$

其中 $\Diamond_{\leq \alpha}$ 为有界 eventually 算子（在 $\alpha$ 步内），此时判定退化为 $O(|\mathcal{T}|)$。阈值 $\alpha$ 的选择本身是一个开放问题（§6, Conjecture 5）。

## 4. 压力动力学

### 4.1 压力函数族

**定义 5**（压力函数族）。定义五元函数族 $\mathcal{P} = \{P_\tau, P_\sigma, P_\gamma, P_\rho, P_\theta\}$，每个 $P_i: G \times \mathbb{N} \to \mathbb{R}_{\geq 0}$。

**I. 兑现压力**（Thread Pressure）。

$$P_\tau(t, n) = \text{age}(t,n)^{\,\beta} \cdot w(t), \quad \beta > 1 \tag{P1}$$

$$\text{age}(t, n) = n - n_{\text{plant}}(t), \quad w: \mathcal{W} \to \mathbb{R}_{>0}$$

选择超线性增长 $\beta > 1$ 的理由来自认知心理学的期望违背理论（Expectation Violation Theory）：受众对未兑现承诺的不满随时间加速积累，近似 Stevens 幂律[^1] $\psi = k \cdot I^\beta$。

**II. 揭示压力**（Secret Pressure）。

$$P_\sigma(\sigma, G) = \tau(\sigma) \cdot \Big(1 + \alpha_k \cdot |\mathcal{K}(\sigma)| + \alpha_s \cdot |\mathcal{S}_?(\sigma)|\Big) \tag{P2}$$

其中 $\mathcal{K}(\sigma) = \{c \mid (c, \texttt{KNOWS}, \sigma) \in E\}$，$\mathcal{S}_?(\sigma) = \{c \mid (c, \texttt{SUSPECTS}, \sigma) \in E\}$。

**信息论解释**：秘密可视为叙事中的信息不对称。设全体角色集为 $\mathcal{C}$，定义角色 $c$ 关于秘密 $\sigma$ 的认知状态为随机变量 $X_c^\sigma \in \{\text{know}, \text{suspect}, \text{ignorant}\}$。则秘密的「社会张力」可用条件熵刻画：

$$H(\sigma \mid \mathcal{C}) = -\sum_{c \in \mathcal{C}} \sum_{x} p(X_c^\sigma = x) \log p(X_c^\sigma = x)$$

当知晓者与无知者共存时，条件熵达到最大——这正是秘密最具叙事张力的状态。$P_\sigma$ 可视为此条件熵的一阶近似。

**III. 冲突压力**（Conflict Pressure）。

$$P_\gamma(g_1, g_2, s) = \frac{\iota(g_1) + \iota(g_2)}{2} \cdot \mathbb{1}\big[\text{holder}(g_1) \in \text{chars}(s) \,\land\, \text{holder}(g_2) \in \text{chars}(s)\big] \tag{P3}$$

关键性质：**空间激活**（Spatial Activation）。冲突压力仅在持有者共享空间时从潜在态跃迁为激活态。这对应亚里士多德的三一律中「地点统一」的要求——戏剧冲突需要空间上的共在。

**IV. 关系压力**（Relationship Pressure）。

$$P_\rho(c_i, c_j, G) = \max\Big(0,\; |\mathcal{E}_{\cap}(c_i, c_j)| - |\mathcal{F}_{\text{rel}}(c_i, c_j)|\Big) \tag{P4}$$

$\mathcal{E}_\cap$ 为共同事件集，$\mathcal{F}_{\text{rel}}$ 为关系边集。互动频繁但关系未定义时产生压力——驱动关系的显式化。

**V. 张力压力**（Tension Homeostasis）。

$$P_\theta(G, \Delta) = \delta_{\text{sign}} \cdot \Delta, \quad \delta_{\text{sign}} = \begin{cases} +\delta_+ & \text{if } \overline{H}_\Delta < H_{\text{low}} \\ -\delta_- & \text{if } \overline{H}_\Delta > H_{\text{high}} \\ 0 & \text{otherwise} \end{cases} \tag{P5}$$

其中 $\overline{H}_\Delta$ 为最近 $\Delta$ 个场景的滑动平均熵。这是一个**自稳定器**（homeostatic regulator），维持叙事张力在目标区间 $[H_{\text{low}}, H_{\text{high}}]$ 内振荡。

### 4.2 压力传导与 Laplacian

**定义 6**（叙事 Laplacian）。定义叙事图 $G$ 的加权邻接矩阵 $A_\omega$，其中：

$$[A_\omega]_{ij} = \sum_{\ell:\,(v_i, \ell, v_j) \in E} \omega(\ell)$$

$\omega: \mathcal{L} \to \mathbb{R}_{>0}$ 为力的传导权重函数。对应的 Laplacian $L_\omega = D_\omega - A_\omega$，特征值谱 $0 = \lambda_1 \leq \lambda_2 \leq \cdots \leq \lambda_N$。

**定义 7**（有效压力）。节点 $v$ 的有效压力定义为直接压力加衰减传导项：

$$P_{\text{eff}}(v) = P_{\text{local}}(v) + \mu \sum_{(u, \ell, v) \in E} P_{\text{local}}(u) \cdot \omega(\ell) \tag{8}$$

其中 $\mu \in (0,1)$ 为全局衰减因子。

**命题 2**（矩阵形式）。设 $\mathbf{p} = [P_{\text{local}}(v_1), \ldots, P_{\text{local}}(v_N)]^\top$，则有效压力向量为：

$$\mathbf{p}_{\text{eff}} = (I + \mu A_\omega^\top)\,\mathbf{p} \tag{9}$$

当 $\mu < 1/\rho(A_\omega)$（$\rho$ 为谱半径）时，Neumann 级数收敛，可扩展为多跳传导：

$$\mathbf{p}_{\text{eff}}^{(k)} = \left(\sum_{j=0}^{k} \mu^j (A_\omega^\top)^j\right) \mathbf{p} \tag{10}$$

极限 $k \to \infty$ 给出**全局均衡压力** $\mathbf{p}^* = (I - \mu A_\omega^\top)^{-1}\,\mathbf{p}$。

**定理 3**（压力均衡速率）。叙事图上的压力扩散过程收敛到均衡分布的速率由 Fiedler 值 $\lambda_2(L_\omega)$ 控制。具体地，设 $\mathbf{p}(n)$ 为第 $n$ 场景后的压力分布，$\mathbf{p}^*$ 为均衡分布，则：

$$\|\mathbf{p}(n) - \mathbf{p}^*\|_2 \leq \left(1 - \frac{\lambda_2}{d_{\max}}\right)^n \cdot \|\mathbf{p}(0) - \mathbf{p}^*\|_2 \tag{11}$$

*证明概要*。标准的 Laplacian 扩散分析。将压力扩散建模为 $\mathbf{p}(n+1) = (I - \frac{1}{d_{\max}}L_\omega)\mathbf{p}(n) + \mathbf{f}(n)$（$\mathbf{f}$ 为外部注入），利用 $L_\omega$ 的谱分解和 Cheeger 不等式得到几何收敛速率。$\square$

**物理解释**：$\lambda_2$ 大的叙事图（各节点间连接紧密）压力传导快——一个局部事件迅速影响全局。$\lambda_2$ 小的叙事图（包含松散连接的子图）压力传导慢——不同子情节线之间近乎独立演化。

## 5. 叙事热力学

### 5.1 叙事熵

**定义 8**（叙事熵）。叙事图 $G$ 在场景 $n$ 处的熵：

$$H(G, n) = \sum_{t \in \mathcal{T}_{\text{open}}} P_\tau(t,n) + \sum_{\sigma \in \Sigma} P_\sigma(\sigma, G) + \sum_{(g_i,g_j) \in \mathcal{G}_\times} P_\gamma(g_i,g_j,s_n) + \sum_{\substack{c_i, c_j \in \mathcal{C} \\ i < j}} P_\rho(c_i,c_j,G) \tag{12}$$

其中 $\mathcal{T}_{\text{open}} = \{t \in \mathcal{T} \mid \text{status}(t) \notin \{\texttt{paid}, \texttt{abandoned}\}\}$，$\mathcal{G}_\times = \{(g_i, g_j) \mid (g_i, \texttt{CONFLICTS}, g_j) \in E\}$。

**定义 9**（熵产率）。定义单场景熵产率：

$$h(n) = H(G, n) - H(G, n-1) = \Delta H_{\text{plant}}(n) - \Delta H_{\text{resolve}}(n) + \Delta H_{\text{aging}}(n) \tag{13}$$

其中三项分别对应：新元素植入带来的熵增、旧元素解决带来的熵减、以及现存元素因 age 增长带来的自然熵增。

**引理 1**（自然熵增）。在无任何植入或解决操作的「空转」场景中，熵仍然增长：

$$h_{\text{idle}}(n) = \sum_{t \in \mathcal{T}_{\text{open}}} \left[\text{age}(t,n)^\beta - \text{age}(t,n-1)^\beta\right] \cdot w(t) > 0$$

*证明*。因 $\beta > 1$ 且 $\text{age} \geq 1$，$f(x) = x^\beta$ 严格递增，故差分恒正。$\square$

这意味着**不作为本身就在增加叙事压力**——拖延不解决问题会让所有悬而未决的伏笔持续积累压力。这是契诃夫守恒在动力学层面的直接体现。

### 5.2 三幕同构

**定理 4**（熵-节拍对应）。设叙事遵循标准三幕结构，总长度为 $n_{\max}$ 场景。定义植入率 $r_+(n)$ 和解决率 $r_-(n)$。在 Snyder 模型[^2]下：

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

*证明概要*。Act I 以植入为主（$r_+ \gg r_-$），加之 aging 效应，$h(n) > 0$。Act II-b 虽植入放缓，但大量累积的 Thread 因 $\beta > 1$ 使 $h_{\text{idle}}$ 持续增长。Act III 大规模解决使 $r_-$ 压过 $r_+$ 和 $h_{\text{idle}}$。$\square$

### 5.3 非平衡热力学类比

叙事系统更接近**非平衡热力学**中的耗散结构（Prigogine[^3]）而非平衡态系统：

- **开放系统**：叙事图持续有外部输入（玩家动作、新 Thread 植入）和输出（Thread 兑现、角色退场），不趋向热寂。
- **稳态非零熵产**：健康的叙事维持非零的熵产率 $h(n) \approx h^*$，对应持续的悬念与张力。
- **有序的涌现**：在远离平衡态的条件下，耗散结构可以自发形成有序模式——这对应叙事中的**涌现情节**（Emergent Plot）。

这引出本文的第一个核心猜想。

## 6. 猜想

### Conjecture 1：叙事 Le Chatelier 原理

**陈述**。设叙事图 $G$ 处于压力准平衡态 $\mathbf{p}^*$。若在场景 $n_0$ 施加外部扰动 $\delta \mathbf{p}$（例如玩家的意外行动引入新的 Thread 或破坏一段关系），则系统在后续 $k$ 个场景内的自主演化方向将**部分抵消**该扰动：

$$\langle \mathbf{p}(n_0 + k) - \mathbf{p}^*, \;\delta\mathbf{p} \rangle < \|\delta\mathbf{p}\|^2 \quad \text{for sufficiently large } k \tag{C1}$$

**动机**。热力学中的 Le Chatelier 原理指出：处于平衡态的系统受扰动后，其自发演化方向倾向于抵消扰动。在叙事语境中，这意味着：突然引入大量新 Thread（熵激增）会触发加速解决（张力压力 $P_\theta$ 的释放方向）；突然解决所有冲突（熵骤降）会触发新的 complication（$P_\theta$ 的升温方向）。叙事系统天然抗拒极端状态。

**可检验性**。在受控实验中（固定 LLM、固定叙事人格参数），记录扰动前后的 $H(G, n)$ 轨迹，验证 $\text{Corr}(\Delta H_{\text{post}}, \Delta H_{\text{perturb}}) < 0$。

### Conjecture 2：谱间隙与叙事聚合度

**陈述**。叙事图 $G$ 的 Fiedler 值 $\lambda_2(L_\omega)$ 与主观叙事聚合度评分 $Q(G) \in [0, 1]$（由人类评估者给出的「故事是否感觉紧凑/聚焦」评分）之间存在正相关：

$$Q(G) = f(\lambda_2) + \varepsilon, \quad f \text{ 单调递增}, \quad \varepsilon \sim \mathcal{N}(0, \sigma^2) \tag{C2}$$

**动机**。谱图论中，$\lambda_2$ 衡量图的连通强度——$\lambda_2 = 0$ 意味着图不连通（完全独立的子情节），$\lambda_2$ 大意味着任意两个节点间有短路径（叙事元素紧密交织）。

具有小 $\lambda_2$ 的叙事图对应「散装叙事」——多条情节线互不关联，缺乏叙事统一感。具有大 $\lambda_2$ 的叙事图对应「织网叙事」——角色、秘密、伏笔相互牵连，牵一发而动全身。

**推论**。$\lambda_2$ 可作为叙事聚合度的实时监控指标。当 $\lambda_2$ 降至阈值以下时，系统应建议引入跨线连接（如让不同子线的角色相遇、或让一个秘密关联到多条伏笔线）。

### Conjecture 3：叙事相变

**陈述**。定义**叙事密度** $\rho(G) = |E|\,/\,|V|^2$。存在临界密度 $\rho_c$ 和临界指数 $\nu > 0$，使得叙事聚合度 $Q$ 在 $\rho_c$ 附近发生相变：

$$Q(\rho) \sim \begin{cases} 0 & \text{if } \rho < \rho_c \\ (\rho - \rho_c)^\nu & \text{if } \rho \geq \rho_c \end{cases} \tag{C3}$$

**动机**。这直接类比渗流理论（Percolation Theory）中的相变：当图的边密度低于临界值时，图几乎必然是碎片化的（subcritical phase）；超过临界值后出现巨组分（giant component），对应全局连通的叙事结构。

**与实践的对应**：

- **亚临界相** $(\rho < \rho_c)$：叙事呈碎片化状态。多个独立子图对应互不关联的情节碎片。玩家感受为「发生了很多事但没有主线」。这是长会话 AI RP 的常见退化模式。
- **超临界相** $(\rho > \rho_c)$：出现跨越大部分节点的连通分量。任何局部事件都通过边传导影响全局。叙事感知为「紧凑、有主线」。
- **超密相** $(\rho \gg \rho_c)$：过度连接导致每个动作触发连锁反应，叙事失控。可能对应另一个相变点 $\rho_c'$，超过后聚合度反而下降（信息过载）。

临界指数 $\nu$ 的值取决于叙事图的拓扑类，这与底层力的类型分布有关。Erdős–Rényi 随机图的渗流相变给出 $\nu = 1$（均场近似）；叙事图由于力的类型约束，可能属于不同的普适类（universality class）。

### Conjecture 4：熵产率的上下界

**陈述**。存在叙事参与度函数 $\text{Eng}(n) \in [0, 1]$（Engagement，可由用户行为指标估计），使得：

$$\text{Eng}(n) \approx \Phi\Big(\frac{h(n) - h_{\min}}{h_{\max} - h_{\min}}\Big) \tag{C4}$$

其中 $\Phi$ 为 Sigmoid 函数，$h_{\min}, h_{\max}$ 为临界熵产率。特别地：

- $h(n) < h_{\min}$（熵产率过低）：叙事停滞，参与度趋零。对应「什么都没发生」。
- $h(n) > h_{\max}$（熵产率过高）：信息过载，参与度趋零。对应「发生了太多事、跟不上」。
- $h(n) \in [h_{\min}, h_{\max}]$：参与度处于高原区。

**动机**。认知心理学中的 Yerkes-Dodson 定律[^4]描述了刺激强度与表现之间的倒 U 形关系。这里的熵产率 $h(n)$ 类比于刺激强度——太低则无聊，太高则焦虑，最优区间对应心流状态（Csikszentmihalyi[^5]）。

**推论**。最优叙事策略是维持 $h(n)$ 在 $[h_{\min}, h_{\max}]$ 内振荡。张力压力 $P_\theta$（定义 5-V）正是为此设计的自稳定器。

### Conjecture 5：契诃夫阈值的最优性

**陈述**。定律 IV 的有界弱化版本中，阈值 $\alpha^*$ 的最优取值满足：

$$\alpha^* = \underset{\alpha}{\arg\min}\; \Big[\mathbb{E}_{\mathcal{N}}\big[\text{FP}(\alpha)\big] + \gamma \cdot \mathbb{E}_{\mathcal{N}}\big[\text{FN}(\alpha)\big]\Big] \tag{C5}$$

其中 $\text{FP}(\alpha)$ 为假阳性率（正在发展中的伏笔被误判为陈旧），$\text{FN}(\alpha)$ 为假阴性率（真正被遗忘的伏笔未被检出），$\gamma$ 为相对代价权重，$\mathcal{N}$ 为叙事分布。

**进一步猜测**。$\alpha^*$ 与 Thread 的权重类型 $\mathcal{W}$ 有关：

$$\alpha^*(w) \propto w^{-1/\beta}$$

即高权重伏笔（major）的容忍阈值更低（更早触发兑现压力），低权重伏笔（subtle）可以潜伏更久。这与观众期望一致：主线伏笔的回收应比支线更及时。

### Conjecture 6：Lyapunov 收敛性

**陈述**。定义目标熵 $H^*$ 和 Lyapunov 候选函数：

$$V(G, n) = \frac{1}{2}\big(H(G,n) - H^*\big)^2 \tag{C6}$$

若张力压力 $P_\theta$ 的参数满足 $\delta_+, \delta_- > 0$ 且 $H_{\text{low}} < H^* < H_{\text{high}}$，则 $V$ 在 $P_\theta$ 的调控下是 Lyapunov 函数：

$$\Delta V(n) = V(G, n+1) - V(G, n) < 0 \quad \text{whenever } |H(G,n) - H^*| > \varepsilon \tag{C6'}$$

其中 $\varepsilon$ 为与 $\delta_\pm$ 和压力函数的 Lipschitz 常数相关的正常数。

**含义**。这保证了叙事熵的**轨道稳定性**——在张力压力的调控下，$H(G,n)$ 不会永久偏离目标区间，而是在 $H^* \pm \varepsilon$ 的邻域内振荡。叙事既不会永远平淡（$H \to 0$），也不会永远混乱（$H \to \infty$），而是被吸引到一个稳定的**极限环**（limit cycle）。

**证明思路**。当 $H > H^* + \varepsilon$ 时，$P_\theta < 0$ 产生释放压力，$h(n)$ 中出现负项使 $H$ 下降，故 $\Delta V < 0$。对称地，当 $H < H^* - \varepsilon$ 时，$P_\theta > 0$ 产生升温压力。严格证明需要对压力函数施加 Lipschitz 连续性条件以控制单步变化量。

## 7. 推演示例

### 7.1 初始化 ($n = 0$)

```
CREATE  Character  alice   {traits: [curious, stubborn]}
CREATE  Character  bob     {traits: [secretive, protective]}
CREATE  Setting    tavern  {atmosphere: mysterious}

LINK    alice ──AT──> tavern
LINK    bob   ──AT──> tavern
LINK    alice ──TRUSTS[8]──> bob
LINK    bob   ──LOVES[rom]──> alice
```

$H(G_0) = 0$。图 Laplacian $L_\omega$ 为 $3 \times 3$ 矩阵（两角色一场景），$\lambda_2 > 0$（连通图）。

### 7.2 植入 ($n = 1$)

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

守恒验证：$\text{Law}_\text{I}$: ✓，$\text{Law}_\text{III}$: ✓（$\sigma_1$ by bob），$\text{Law}_\text{V}$: ✓（$g_1$ by alice）。

### 7.3 压力演化 ($n = 1 \to 5$)

设 $\beta = 1.5$，$w_{\text{major}} = 2$，$\alpha_k = 0.5$，$\alpha_s = 0.3$。

| $n$ | 叙事事件 | $P_\tau(t_1)$ | $P_\sigma(\sigma_1)$ | $H(G_n)$ | $h(n)$ |
|-----|---------|:------------:|:-------------------:|:---------:|:------:|
| 1 | 植入 | $1^{1.5} \cdot 2 = 2.0$ | $9(1+0.5) = 13.5$ | 15.5 | +15.5 |
| 2 | （空转） | $2^{1.5} \cdot 2 \approx 5.7$ | 13.5 | 19.2 | +3.7 |
| 3 | （空转） | $3^{1.5} \cdot 2 \approx 10.4$ | 13.5 | 23.9 | +4.7 |
| 4 | Alice SUSPECTS σ₁ | $4^{1.5} \cdot 2 = 16.0$ | $9(1+0.5+0.3) = 16.2$ | 32.2 | +8.3 |
| 5 | （空转） | $5^{1.5} \cdot 2 \approx 22.4$ | 16.2 | 38.6 | +6.4 |

注意 $n = 2,3$ 的空转场景中 $h > 0$（引理 1），且 $h$ 递增——反映了 $P_\tau$ 的超线性增长。

### 7.4 压力释放 ($n = 6$)

```
REVEAL  σ₁  to: {alice}
UPDATE  t₁  {status: paid}
DELETE  alice ──TRUSTS[8]──> bob
LINK    alice ──HATES[betrayal]──> bob
```

$$H(G_6) = 0 + 0 + P_\rho(\text{alice, bob}) \approx P_\rho$$

$t_1$ 和 $\sigma_1$ 的压力清零，$H$ 从 $\approx 39$ 骤降至 $\approx P_\rho$。但新的 HATES 边与既有 LOVES 边形成关系张力，$P_\rho > 0$。

$$\Delta H(6) \approx -(22.4 + 16.2) + P_\rho \ll 0$$

这是一个 Act III 特征：**主线压力的释放，伴随着新的、更深层的压力注入。**

## 8. 相关工作

### 编剧理论
Snyder[^2]的三幕节拍结构为宏观叙事熵波形提供了经验基准。Weiland[^6]的角色弧线模型（Lie → Truth）可直接编码为 Character 粒子的 arc 属性。Swain[^1]的 Scene-Sequel 模型启发了张力压力 $P_\theta$ 的交替自稳定设计。McKee[^7]的冲突层级理论对应了我们压力函数的多尺度结构。

### 涌现叙事系统
Dwarf Fortress[^8]的世界模拟验证了「简单规则产生复杂涌现」的可行性。RimWorld[^9]的 AI Storyteller 系统与我们的张力压力 $P_\theta$ 具有相似的自稳定目标，但缺乏形式化的压力传导和熵度量。Façade[^10]的 Drama Manager 使用了显式的张力曲线管理，但其规则是硬编码的，缺乏本文提出的基于图结构的自适应性。

### 计算叙事学
Mani[^11]的 Computational Narratology 提供了叙事的形式语言学框架，但未涉及动力学建模。Porteous & Cavazza[^12]的 Plan-based Narrative Generation 使用规划器驱动叙事，与我们的压力驱动范式形成互补。

### 图信号处理
叙事 Laplacian 的引入连接了本框架与图信号处理（GSP）领域[^13]。Conjecture 2 中的谱间隙-聚合度假说可以在 GSP 的频域分析框架内进一步精化。

## 9. 结语

本文提出的叙事物理学框架试图回答一个根本性问题：**交互式叙事的结构性质是否可以用物理系统的数学语言精确刻画？**

我们的初步回答是肯定的。守恒定律提供了一致性的不变量，压力函数族提供了演化的驱动力，叙事熵提供了宏观状态的度量，Laplacian 谱分析提供了结构性质的代数刻画。六条猜想分别涉及稳定性（C1, C6）、结构性质（C2, C3）、信息论界（C4）和算法优化（C5），覆盖了从理论基础到工程实践的不同层次。

大量工作仍有待完成：压力函数的参数理论、熵分量的非线性交互模型、LLM 对压力信号的响应函数量化、以及猜想 C3 中临界指数 $\nu$ 的普适类分类。但框架的核心直觉——**好故事不是规定出来的，是在正确的物理环境中自然生长出来的**——至少提供了一个值得严肃对待的研究方向。

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
