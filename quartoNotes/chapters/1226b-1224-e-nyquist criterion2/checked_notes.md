# 通信原理深度学习笔记  
**课程：EE140《通信系统导论》第15讲**  
**主讲人：李翔 教授（上海科技大学）**  
**时间：2025年12月26日**

> **教授点拨**：本节课是数字通信理论的收尾与升华，核心在于从“无噪声理想传输”走向“有噪声下的最优检测”。我们不仅要掌握公式推导，更要理解其背后的物理意义、设计权衡和决策逻辑。这不仅是技术，更是工程思维的体现。

![上海科技大通信系统课件。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_8.png)

---

## 一、课程回顾：从基带波形到奈奎斯特准则

### 1.1 基带信号模型与无码间干扰（ISI）

在数字通信中，我们将离散符号序列 $\{u_k\}$ 转换为连续时间信号 $u(t)$ 进行传输：

$$
u(t) = \sum_{k} u_k p(t - kT)
$$

其中：
- $u_k$：第 $k$ 个时刻发送的符号；
- $p(t)$：**基带脉冲成形函数**（或称基函数）；
- $T$：符号周期（Symbol Duration），也称为“单符时长”。

接收端通过滤波器 $q(t)$ 处理接收到的信号，并在采样时刻 $t = nT$ 提取信息。目标是实现 **无码间干扰（Inter-Symbol Interference, ISI-Free）恢复**，即恢复出的 $\hat{u}_n = u_n$。

为此，定义系统的整体脉冲响应为：

$$
g(t) = p(t) * q(t)
$$

![通信系统信号处理流程。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_1.png)

### 1.2 奈奎斯特准则（Nyquist Criterion）

奈奎斯特准则给出了实现无ISI传输的充要条件。

#### ✅ 时域表述
$g(t)$ 必须是一个**理想奈奎斯特脉冲**，满足：

$$
g(nT) = 
\begin{cases}
1, & n = 0 \\
0, & n \neq 0
\end{cases}
$$

这意味着，在每个符号采样点上，仅当前符号有贡献，其余符号的响应恰好为零。

> **教授点拨 · 直观理解**  
> “你可以把 $g(t)$ 想象成一个‘梳子’——它的齿只在整数倍 $T$ 处起作用，其他地方都是空的。这样你每次‘抓取’信号时，只会抓到你想抓的那个符号。”

![奈奎斯特准则示例。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_2.png)

![奈奎斯特准则与信号分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_3.png)

#### ✅ 频域表述（奈奎斯特第一准则）
令 $G(f)$ 为 $g(t)$ 的傅里叶变换，则 $G(f)$ 必须满足：

$$
\sum_{k=-\infty}^{\infty} G\left(f + \frac{k}{T}\right) = T, \quad \forall f \in \left[-\frac{1}{2T}, \frac{1}{2T}\right]
$$

即将 $G(f)$ 以 $\frac{1}{T}$ 为间隔进行周期性折叠后，在基带区间 $\left[-\frac{1}{2T}, \frac{1}{2T}\right]$ 内叠加的结果为常数。

这个常数通常归一化为 $T$，表示总能量守恒。

> **教授点拨 · 发展脉络**  
> 奈奎斯特准则最早由 Harry Nyquist 在 1928 年提出，用于分析电报系统中的最大传输速率。它揭示了一个根本规律：**信道带宽决定了符号传输的极限速度**。这一思想后来成为香农信息论的基础之一。

---

## 二、脉冲成形设计：理想与现实的权衡

### 2.1 最小带宽限制：奈奎斯特带宽

根据奈奎斯特准则，最小可能的基带信号带宽（即“奈奎斯特带宽”）为：

$$
B_N = \frac{1}{2T}
$$

若实际基带带宽 $B_b < B_N$，则无法满足奈奎斯特准则，必然存在 ISI。

> **关键概念澄清**  
> 在语音稿中提到的“XX带宽”、“WB=1/(2T)”等术语，经核实应统一为 **奈奎斯特带宽 $B_N = \frac{1}{2T}$**。这是系统设计的基本约束。

### 2.2 典型脉冲函数对比

| 脉冲类型 | 时域表达式 | 频域特性 | 是否满足奈奎斯特准则 | 缺陷 |
|--------|-----------|---------|------------------|------|
| 矩形脉冲（Rectangular） | $ \text{rect}\left(\frac{t}{T}\right) $ | $\text{sinc}(fT)$ | ❌ 不满足 | 频谱无限宽，带宽利用率极低 |
| sinc脉冲 | $ \frac{\sin(\pi t / T)}{\pi t / T} $ | 矩形窗 $[ -1/(2T), 1/(2T) ]$ | ✅ 满足 | 时域衰减慢（$\sim 1/t$），对定时误差敏感 |

> **教授点拨 · 工程启示**  
> “sinc 函数虽然理论上完美，但在现实中几乎不可用。想象一下：你的信号在几千个符号周期后仍有微弱回响，只要有一点点采样偏差，就会引入严重干扰。这就是为什么我们说‘理想的往往不实用’。”

![数字通信信号处理概述](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_4.png)

### 2.3 实际解决方案：升余弦滚降滤波器（Raised Cosine Filter）

为了在**带宽效率**与**时域收敛性**之间取得平衡，实际系统采用**升余弦滚降滤波器**。

其频谱定义为：

$$
P(f) = 
\begin{cases}
\sqrt{T}, & |f| \leq \frac{1 - \alpha}{2T} \\
\sqrt{\frac{T}{2} \left[1 + \cos\left( \frac{\pi T}{\alpha} \left(|f| - \frac{1 - \alpha}{2T} \right) \right)\right]}, & \frac{1 - \alpha}{2T} < |f| \leq \frac{1 + \alpha}{2T} \\
0, & |f| > \frac{1 + \alpha}{2T}
\end{cases}
$$

其中 $\alpha \in [0,1]$ 为**滚降系数**。

- 当 $\alpha = 0$：退化为理想低通，带宽最小但不可实现；
- 当 $\alpha = 1$：带宽为 $B = \frac{1}{T}$，是奈奎斯特带宽的两倍，但时域衰减快（$\sim 1/t^3$），抗定时抖动能力强。

> **教授点拨 · 设计哲学**  
> “我们宁愿多花一点带宽，换来更强的鲁棒性。通信系统不是追求极致性能，而是要在各种不确定因素下依然可靠工作。这就是‘稳健设计’的思想。”

![通信系统符号基带波形转换](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_5.png)

![带宽效率与能量权衡](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_6.png)

---

## 三、匹配滤波与正交波形：最优接收的核心

### 3.1 匹配滤波器（Matched Filter）

当信道中存在加性白高斯噪声（AWGN）时，最优接收机使用**匹配滤波器**来最大化输出信噪比（SNR）。

若发送脉冲为 $p(t)$，则接收端滤波器应设计为：

$$
q(t) = p(T - t)
$$

即 $p(t)$ 关于 $t = T/2$ 对称的时间反转（若非因果需延迟）。该结构确保了接收机对已知信号波形的“最佳匹配”。

此时，输出信噪比达到最大值。

### 3.2 正交波形与信号空间表示

一个重要结论是：

> 若 $\{p(t - kT)\}$ 构成一组单位正交函数集，且接收滤波器为匹配滤波器，则整体响应 $g(t)$ 满足奈奎斯特准则。

这意味着我们可以将符号序列 $\{u_k\}$ 视为在一组正交基上的投影系数，而接收过程就是对该信号在对应基方向上的内积运算（即投影）：

$$
\hat{u}_k = \int u(t) \cdot p(t - kT) dt
$$

这正是解调的本质——**从信号空间中提取坐标**。

> **教授点拨 · 思路升华**  
> “你看，整个通信过程就像是一次‘向量搬家’：你在发送端把一个数字序列‘编码’成一个波形向量；经过信道传输后，接收端再把这个波形‘解码’回原来的数字序列。而奈奎斯特准则和匹配滤波，就是保证这次搬家不丢包、不错位的关键机制。”

---

## 四、带通调制：从基带到射频

完成基带成形后，需将其调制到高频以便在无线或有线信道中传输。

对于复包络信号 $u(t)$，带通信号为：

$$
s(t) = \text{Re} \left\{ u(t) e^{j2\pi f_c t} \right\}
$$

具体实现方式取决于调制类型：

- **PAM（脉冲幅度调制）**：$u(t)$ 为实信号，直接双边带调制；
- **QAM（正交幅度调制）**：$u(t) = u_I(t) + j u_Q(t)$，分别与 $\cos(2\pi f_c t)$ 和 $-\sin(2\pi f_c t)$ 相乘后叠加。

![PAM检测与解调过程](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_9.png)

---

## 五、核心挑战：含噪信道下的最优检测

至此，我们进入本课最深刻的环节——**如何在噪声中做出最佳判断？**

### 5.1 检测问题建模

考虑最简单的场景：**二进制检测（Binary Detection）**

- 发送比特 $b \in \{0, 1\}$
- 映射为星座点：$b=0 \to a_0$, $b=1 \to a_1$
- 接收信号：$v = u + z$，其中 $z \sim \mathcal{N}(0, N_0/2)$ 为 AWGN
- 观测值 $v$ 是随机变量，目标是基于 $v$ 判断原始发送的是 $a_0$ 还是 $a_1$

![二元检测原理与公式](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_14.png)

### 5.2 关键概率模型

| 名称 | 符号 | 定义 | 物理意义 |
|------|------|------|----------|
| 先验概率 | $P(a_m)$ | 发送前认为 $a_m$ 被发送的概率 | 反映发送端统计特性 |
| 似然概率 | $p(v\|a_m)$ | 给定发送 $a_m$，观测到 $v$ 的条件概率 | 描述信道行为 |
| 后验概率 | $p(a_m\|v)$ | 给定观测到 $v$，推断发送的是 $a_m$ 的概率 | 支持决策的依据 |

![检测原理与MAP准则。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_10.png)

### 5.3 最优检测准则

#### (1) 最大后验概率准则（MAP: Maximum A Posteriori）

选择使后验概率最大的假设：

$$
\hat{m} = \arg\max_m p(a_m | v)
$$

根据贝叶斯公式：

$$
p(a_m|v) = \frac{p(v|a_m) P(a_m)}{p(v)}
$$

由于分母 $p(v)$ 对所有 $m$ 相同，等价于最大化分子：

$$
\hat{m} = \arg\max_m \left[ p(v|a_m) \cdot P(a_m) \right]
$$

![二元检测：最大后验概率准则。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_15.png)

#### (2) 最大似然准则（ML: Maximum Likelihood）

当先验等概（$P(a_0)=P(a_1)=0.5$）时，MAP 退化为 ML：

$$
\hat{m} = \arg\max_m p(v|a_m)
$$

> **教授点拨 · 决策智慧**  
> “MAP 像是一个‘理性决策者’：它不仅看证据（似然），还考虑先入之见（先验）。而 ML 更像是一个‘证据至上主义者’，只相信眼前看到的数据。两者没有绝对好坏，取决于你是否了解发送端的统计规律。”

![检测理论：MAP与ML准则。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_11.png)

---

### 5.4 似然比检验（Likelihood Ratio Test, LRT）

引入**似然比**：

$$
\Lambda(v) = \frac{p(v|a_0)}{p(v|a_1)}
$$

MAP 决策规则可写为：

$$
\Lambda(v) \overset{\hat{a}=a_0}{\underset{\hat{a}=a_1}{\gtrless}} \eta = \frac{P(a_1)}{P(a_0)}
$$

即比较似然比与阈值 $\eta$。

> **教授点拨 · 数学之美**  
> “看似复杂的概率决策，最终简化为一个简单的‘比较操作’。这就是数学的力量——将复杂现象抽象为简洁规则。”

![通信系统决策区域分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_16.png)

---

### 5.5 典型案例：反极性信号检测（Antipodal Signaling）

设：
- $a_0 = +a$, $a_1 = -a$
- $z \sim \mathcal{N}(0, \sigma^2), \sigma^2 = N_0/2$

则：
- $p(v|+a) = \mathcal{N}(a, \sigma^2)$
- $p(v|-a) = \mathcal{N}(-a, \sigma^2)$

计算似然比并取对数得**对数似然比（LLR）**：

$$
\text{LLR}(v) = \log \Lambda(v) = \frac{2a}{\sigma^2} v
$$

MAP 决策规则变为：

$$
v \overset{\hat{a}=+a}{\underset{\hat{a}=-a}{\gtrless}} \frac{\sigma^2}{2a} \log \left( \frac{P(-a)}{P(+a)} \right)
$$

若先验等概（$P(+a)=P(-a)$），则阈值为 0：

$$
\hat{a} = 
\begin{cases}
+a, & v > 0 \\
-a, & v < 0
\end{cases}
$$

> **教授点拨 · 生活类比**  
> “这就像你听不清一句话，只能听到模糊的声音。如果你本来就倾向于对方说的是‘是’，那么即使声音偏向‘否’，你也可能仍判断为‘是’——除非这个反向证据足够强。这就是偏见（prior）的作用。改变一个人的观点，需要更强的证据。”

![二进制检测，抗极性信号。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_20.png)

![二元检测与似然比分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_21.png)

![二元检测与MAP准则。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_22.png)

![二叉检测与MAP规则](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_23.png)

---

### 5.6 错误概率分析

总的错误概率为：

$$
P_e = P(\text{error}|a_0)P(a_0) + P(\text{error}|a_1)P(a_1)
$$

对于反极性信号且等概先验，可推导出：

$$
P_e = Q\left( \sqrt{\frac{2E_b}{N_0}} \right)
$$

其中 $Q(x)$ 为 Q 函数，$E_b = a^2 T$ 为每比特能量。

> **教授点拨 · 物理意义**  
> “这个公式告诉我们：**性能由信噪比决定**。你可以通过增加功率（提高 $E_b$）、降低噪声（减小 $N_0$）或延长符号时间（增加 $T$）来改善性能。但这一切都受限于香农极限。”

![二元检测：ML规则与错误概率。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_17.png)

![二元检测错误概率计算。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_18.png)

---

## 六、扩展话题：从二元到多元与序列检测

### 6.1 M-ary 检测

推广至 $M$ 个符号的情况，MAP 规则仍适用：

$$
\hat{m} = \arg\max_m p(a_m|v)
$$

但随着 $M$ 增大，符号间距减小，错误率上升。因此需在**频谱效率**（高 $M$）与**可靠性**（低 $M$）之间权衡。

### 6.2 序列检测

若考虑多个符号联合检测（如 Viterbi 算法），可进一步提升抗干扰能力，尤其适用于存在 ISI 的信道。

![M-ary检测与序列检测](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_13.png)

---

## 七、总结与反思：通信系统的设计哲学

| 维度 | 权衡关系 | 工程启示 |
|------|--------|---------|
| **带宽 vs. 可靠性** | 升余弦滚降系数 $\alpha$ 越小，带宽越窄，但对定时越敏感 | 适度牺牲带宽换取稳定性 |
| **效率 vs. 能量** | 高阶调制（如64-QAM）频谱效率高，但BER差，需更高SNR | 根据信道质量动态调整调制方式（AMC） |
| **性能 vs. 复杂度** | MAP最优但需知道先验，ML次优但更通用 | 实际系统常采用ML或近似MAP |
| **理想 vs. 现实** | sinc函数理论最优，但无法实现 | 工程设计必须考虑实现可行性 |

> **教授点拨 · 课程终章寄语**  
> “同学们，通信不仅仅是发几个比特过去。它是关于**在不确定性中寻找确定性**的艺术。每一个公式背后，都是人类对抗自然限制的努力。希望你们学到的不只是知识，更是一种思维方式——如何在约束中创新，如何在噪声中倾听真相。”

---

## 附录：常见误区澄清

| 误区 | 正确认知 |
|------|----------|
| “奈奎斯特采样定理”与“奈奎斯特ISI准则”是同一回事 | ❌ 二者完全不同：前者针对模拟信号数字化，后者针对数字信号抗干扰设计 |
| 匹配滤波只是为了放大信号 | ❌ 主要目的是在AWGN下最大化输出信噪比（SNR） |
| ML总是最优的 | ❌ 仅当先验等概时，ML才等价于最优的MAP |
| 带宽越宽越好 | ❌ 带宽是稀缺资源，应在满足性能前提下尽量节省 |

---

> **作业与考试提醒**  
> - 第12次作业请按时提交  
> - 第13次作业作为复习练习  
> - 期末考试时间：**2026年1月12日 10:30–12:30**  
> - 地点：教学中心204  
> - 考试范围：期中之后内容为重点，形式同前，请准备一张A4纸  

![上海科技大学感谢提问。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1226b-1224-e-nyquist%20criterion2/page_7.png)

---  
*笔记整理完毕。愿你在通信的世界里，既能仰望理论星空，也能踏实走好每一步工程之路。*