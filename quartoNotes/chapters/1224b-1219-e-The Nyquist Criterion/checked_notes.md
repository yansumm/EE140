# **通信原理：奈奎斯特准则与无码间干扰传输**

> **课程信息**  
> 授课教师：廉黎祥  
> 课程名称：通信原理  
> 讲授时间：2025年12月24日（第15周）  
> 备注：本笔记基于课堂语音转录稿与PPT内容整理，融合知识线与思想线，旨在还原教学精髓，提供准确、深刻且富有洞见的复习材料。

---

## **一、课程安排与期末考试提醒**

### ✅ 考试信息
- **考试时间**：2026年1月12日 10:30–12:30  
- **考试地点**：教学中心 214 教室（非日常上课教室，请提前确认）  
- **考试形式**：
  - 允许携带一张 A4 纸（可正反书写）
  - 可使用简单计算器
  - 题型与期中考试一致
- **考试范围**：
  - **重点在期中之后的内容**（调制、解调、奈奎斯特准则等）
  - 前半部分内容（如概率论、信号系统基础）将作为背景知识融入考题
  - 概念贯穿全课程，建议全面复习

### 📚 作业安排
- 第12次作业：**必须按时提交**，计入总成绩  
- 第13次作业（最后一次）：**不强制提交**，发布后无截止日期，仅作练习用途
  - 内容重要，涉及"检测"部分核心知识点
  - 鼓励完成并用于自测，有问题可咨询助教或老师

### 🔔 温馨提示（教授点拨）
> "有任何关于考试的问题，请**务必提前沟通**，不要等到考试当天才提出来。"  
> —— 廉老师特别强调，体现了对学生的负责态度。提前解决疑问，才能安心备考。

> "大家记得去做课程评价，现在评价率比较低。"  
> —— 虽是小事，但也反映出师生互动的重要性。积极反馈有助于改进教学。

---

## **二、核心概念回顾：从符号到带通信号的生成**

我们之前的学习构建了一个完整的数字通信链路框架：

```
{u_k} → [发送滤波器 p(t)] → u(t) (基带信号) → [载波调制] → s(t) (带通信号)
```

其中：
- $\{u_k\}$ 是离散的符号序列（如 QPSK 中的 ±1±j）
- $p(t)$ 是发送端脉冲成形滤波器
- $u(t) = Σ u_k · p(t - kT)$ 是由符号和脉冲卷积生成的基带波形
- $s(t)$ 是最终通过信道传输的带通信号

本次课的核心，是深入理解接收端如何从接收到的信号中**无误差地恢复出原始符号序列 $\{u_k\}$**。

![Nyquist准则与无ISI信号恢复。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_1.png)

---

## **三、核心问题：如何实现无码间干扰（No ISI）的信号恢复？**

### 3.1 问题建模与目标

假设信道无噪声（理想情况），接收端收到的是基带信号 $u(t)$。我们需要设计一个接收系统，使其能完美恢复 $\{u_k\}$。

接收结构如下：
```
u(t) → [接收滤波器 q(t)] → r(t) → [周期采样 T] → {r(kT)}
```

我们的目标是：  
> **选择合适的 $p(t)$ 和 $q(t)$，使得采样输出 $r(nT) = u_n$ 对所有整数 n 成立。**

这意味着在每个符号周期的采样时刻，只读取到当前符号的信息，不受前后符号波形的干扰——即**无码间干扰（Inter-Symbol Interference, ISI）**。

![Nyquist准则与理想波形分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_2.png)

### 3.2 关键函数：g(t) 的定义

定义复合脉冲响应：
> **$g(t) = p(t) * q(t)$**

则接收信号为：
> **$r(t) = u(t) * q(t) = Σ u_k · g(t - kT)$**

采样后得到：
> **$r(nT) = Σ u_k · g((n - k)T)$**

为了实现无 ISI，我们必须有：
> **$g(mT) = δ[m] = \{ 1, m=0; 0, m≠0 \}$**

这个条件称为 **理想奈奎斯特脉冲响应（Ideal Nyquist Pulse）**。

---

## **四、奈奎斯特准则：时域与频域的双重视角**

### 4.1 时域视角：理想奈奎斯特波形

> **定义**：若函数 $g(t)$ 满足：
> - $g(0) = 1$
> - $g(kT) = 0$，当 $k ≠ 0$（整数 k）

则称 $g(t)$ 为**周期为 T 的理想奈奎斯特波形**。

#### 🎯 为什么这能消除 ISI？
考虑采样点 $r(nT)$：
```
r(nT) = Σ u_k · g((n-k)T)
```
由于 $g(mT) = 0$ 当 $m ≠ 0$，只有当 $k = n$ 时项非零：
```
r(nT) = u_n · g(0) + Σ_{k≠n} u_k · 0 = u_n
```
✅ 完美恢复！

#### 💡 直观图示理解
想象多个平移后的 $g(t)$ 波形叠加形成 $r(t)$：
- 在 $t = 0$ 处，只有 $u_0·g(t)$ 贡献值 $u_0$
- 在 $t = T$ 处，只有 $u_1·g(t-T)$ 贡献值 $u_1$
- 其他波形在此刻恰好过零点

这种"错峰采样"的设计，正是奈奎斯特思想的精妙所在。

---

### 4.2 频域视角：奈奎斯特准则（Nyquist Criterion）
> 奈奎斯特第一准则 (Nyquist's First Criterion for ISI)

> 当我们在信道中传输数字脉冲信号时，面临的挑战不再是"如何采样"，而是"如何保证脉冲之间不互相打架"。这被称为消除码间串扰（Intersymbol Interference, ISI）。
> 严谨定义
>   - 对于一个基带传输系统，若其等效系统函数为 $H(f)$，冲激响应为 $h(t)$，要使采样时刻 $t = nT_s$ 处不存在 ISI，则 $h(t)$ 必须满足：$$h(nT_s) = \begin{cases} 1, & n = 0 \\ 0, & n \neq 0 \end{cases}$$
>     即：在当前码元的采样点上，其他所有码元的值都必须刚好为 0。
> 
> 频域判据
>   - 哈里·奈奎斯特在 1928 年证明，上述时域条件等价于频域的周期性累加：$$\sum_{k=-\infty}^{\infty} H\left(f + \frac{k}{T_s}\right) = T_s \quad (\text{常数})$$
> 
> 直观理解：
>   - 如果将 $H(f)$ 按 $1/T_s$ 为间隔切开并叠在一起，它们累加的结果应该是一个平坦的直线。
> 
> 经典例子：
>   - 升余弦滚降滤波器 (Raised Cosine Filter)由于理想的"砖墙式"低通滤波器在时域表现为 $Sinc$ 函数，其收敛速度慢（$1/t$），且对定时抖动极其敏感，工程中通常使用升余弦滤波器。它通过牺牲一部分带宽（引入滚降系数 $\alpha$），使得时域波形的尾部衰减更快。它依然满足奈奎斯特第一准则，即在 $nT_s$ 处保持零交叉。

我们利用**混叠定理（Aliasing Theorem）** 推导频域条件。

#### 🔁 混叠定理回顾
对连续信号 $x(t)$ 以周期 $T$ 采样，其采样信号的频谱为：
> **$X_s(f) = (1/T) Σ X(f - k/T)$**

即原频谱以 $1/T$ 为间隔复制并叠加。

![采样定理与混叠现象。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_3.png)

#### 🧮 推导过程
为了消除码间串扰（ISI），我们需要脉冲在采样时刻满足：
$$g(kT) = \begin{cases} 1, & k=0 \\ 0, & k \neq 0 \end{cases}$$

第一步：构造采样信号
对连续信号 $g(t)$ 进行理想冲激采样：
$$g_s(t) = g(t) \cdot \sum_{k=-\infty}^{\infty} \delta(t - kT) = \sum_{k=-\infty}^{\infty} g(kT) \delta(t - kT)$$

第二步：代入无 ISI 条件
根据无 ISI 的要求，$g(kT)$ 只有在 $k=0$ 时为 1，其余全为 0。

因此，采样后的信号退化为一个简单的单位冲激：
$$g_s(t) = 1 \cdot \delta(t-0) = \delta(t)$$

第三步：频域转换
根据狄拉克梳状函数的性质，采样信号 $g_s(t)$ 的频谱 $G_s(f)$ 等于原频谱 $G(f)$ 的周期搬移：$$G_s(f) = \frac{1}{T} \sum_{k=-\infty}^{\infty} G\left(f - \frac{k}{T}\right)$$另一方面，直接对 $g_s(t) = \delta(t)$ 求傅里叶变换：$$\mathcal{F}\{\delta(t)\} = 1$$第四步：联立得到准则将上述两个结果相等，得：$$\frac{1}{T} \sum_{k=-\infty}^{\infty} G\left(f - \frac{k}{T}\right) = 1$$整理即得奈奎斯特频域判据：$$\sum_{k=-\infty}^{\infty} G\left(f - \frac{k}{T}\right) = T$$

![Nyquist准则解析与应用。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_4.png)

---

## **五、关键参数定义与关系**

| 符号 | 名称 | 含义 | 单位 |
|------|------|------|------|
| $T$ | 符号周期（Symbol Duration） | 每个符号占用的时间 | 秒 |
| $R_s = 1/T$ | 符号速率（Symbol Rate） | 每秒传输的符号数 | 波特（Baud）或 符号/秒 |
| $W_b = 1/(2T) = R_s / 2$ | **奈奎斯特带宽（Nyquist Bandwidth）** | 理论最小带宽要求 | Hz |
| $B_b$ | 实际基带带宽（Actual Baseband Bandwidth） | $g(t)$ 或 $u(t)$ 的实际带宽 | Hz |





> **⚠️ 注意区分**：$W_b$ 是一个理论量，不是物理信号的真实带宽。它标志着无 ISI 传输的"门槛"。

![奈奎斯特准则图示。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_5.png)

---

## **六、带宽约束与滚降因子：工程上的权衡艺术**

### 6.1 最小带宽要求

要满足奈奎斯特准则，必须有：
> $$B_b ≥ W_b = R_s / 2$$

否则无法在 `[-1/(2T), 1/(2T)]` 内折叠出常数。

- 若 `B_b < W_b`：不可能满足准则，必然存在 ISI。
- 若 `B_b = W_b`：唯一可能是 `G(f)` 为矩形窗（即 `g(t)` 为 sinc 函数）

![奈奎斯特准则详解](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_6.png)

### 6.2 sinc 函数的缺陷

当 `g(t) = sinc(t/T)` 时：
- 时域：无限长，衰减慢（~1/t）
- 问题：对定时抖动极其敏感！微小的采样偏差会导致严重 ISI。

> **💡 教授点拨**：  
> "如果我的采样时间稍微有一点偏差……就会受到非常强的别的 symbol 的干扰。"  
> 这揭示了理论与实践的差距：数学完美的 sinc 脉冲在现实中不可行。

![奈奎斯特准则与带宽关系。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_7.png)

### 6.3 解决方案：引入滚降因子（Roll-off Factor）

实际带宽 $B_b$ 大于理论最小带宽 $W_b$ ，**让频谱在边缘处平滑过渡，从而加快时域衰减**【时域与频域的对偶性】。

> 当你决定以 $R_s$ 的速率发送码元时，理想情况下最省带宽的波形是 $Sinc$ 函数，对应频域是"砖墙"矩形 $rect$函数。但因为"砖墙"不可实现且对定时抖动敏感，我们引入滚降因子 $\alpha$ 使得频谱边缘变得平滑。
> 结果：带宽从 $R_s/2$ 扩展到了 $\frac{R_s}{2}(1+\alpha)$。物理意义：这是一种 **"频谱冗余"**。我们多用了一点频率资源，换取了时域脉冲的快速衰减，从而让系统更健壮。



定义：
> $$α = (B_b - W_b) / W_b = (B_b - R_s/2) / (R_s/2)$$

其中 $α ∈ [0, 1]$，称为**滚降因子（Roll-off Factor）**。

于是：
> $$B_b = W_b (1 + α) = (R_s / 2)(1 + α)$$


> 区分滚降因子与过采样
> 
| 维度 | 滚降因子法 (Pulse Shaping) | 过采样 (Oversampling) |
|------|---------------------------|---------------------|
| 所属范畴 | 奈奎斯特第一准则（消除码间串扰 ISI） | 奈奎斯特-香农采样定理（信号重建） |
| 核心目的 | 压缩带外辐射，平衡带宽效率与抗抖动能力 | 简化模拟滤波器设计，提高信噪比（SNR） |
| 数学表现 | 改变频谱的形状（变得平滑） | 改变频谱副本之间的间距（拉开距离） |
| 关键参数 | 滚降系数 $\alpha \in [0, 1]$ | 过采样率 $L = f_s / f_{Nyquist}$ |

#### ⚖️ 权衡（Trade-off）分析

| α 值 | 优点 | 缺点 |
|------|------|------|
| $$α ≈ 0$$（接近 sinc） | 带宽效率高（高效利用频谱） | 时域衰减慢，抗定时误差能力差 |
| $$α ≈ 1$$ | 时域衰减快（~1/t³），鲁棒性强 | 带宽更宽，频谱效率低 |

> **🎯 工程智慧**：  
> 实际系统通常选择 `α ∈ (0, 1)`，常见值为 0.25, 0.35, 0.5。  
> 在 **可靠性** 与 **频谱效率** 之间取得平衡。

![奈奎斯特准则核心要点。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_8.png)

---

## **七、升余弦滤波器：实际系统的标准答案**

为了满足上述要求，通信系统广泛采用**升余弦滚降滤波器（Raised Cosine Filter）**。

其频域表达式为：
$$
G(f) = 
\begin{cases}
T, & |f| \leq \frac{1-\alpha}{2T} \\
\frac{T}{2}\left[1 + \cos\left(\frac{\pi T\left(|f| - \frac{1-\alpha}{2T}\right)}{\alpha}\right)\right], & \frac{1-\alpha}{2T} < |f| \leq \frac{1+\alpha}{2T} \\
0, & |f| > \frac{1+\alpha}{2T}
\end{cases}
$$

特点：
- 在 `[(1-α)/(2T), (1+α)/(2T)]` 区间内平滑过渡
- 满足奈奎斯特准则
- 时域 `g(t)` 快速衰减

> **📊 PPT 图表洞察**：  
> 不同 `α` 值对应的时域波形显示，`α` 越大，旁瓣衰减越快，定时容限越好。

![Nyquist准则与滤波器特性。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_11.png)

---

## **八、匹配滤波与正交性：系统设计的统一视角**

### 8.1 匹配滤波器设计

常用方法：令接收滤波器 `q(t)` 为发送滤波器 `p(t)` 的**匹配滤波器**。

即：
> $$Q(f) = P^*(f)$$ （频域共轭）  
或等价地：
> $$q(t) = p(T - t)$$ （时域时间反转）

此时：
> $$G(f) = P(f)Q(f) = |P(f)|²$$

若进一步要求 $G(f)$ 满足奈奎斯特准则，则 $g(t)$ 为理想奈奎斯特脉冲。

### 8.2 深层联系：正交波形与信号空间

有一个深刻结论将多个概念统一起来：

> **定理**：以下两个条件等价：
> 1. ${p(t - kT)}$ 构成一组单位正交函数集（即 $\langle p(t-kT), p(t-lT) \rangle = \int_{-\infty}^{\infty} p(t-kT)p(t-lT) dt = \delta[k-l]$），且 $q(t)$ 是 $p(t)$ 的匹配滤波器。
> 2. $g(t) = p(t) \ q(t)$ 是理想奈奎斯特脉冲。

#### 🌟 洞见解读
这揭示了：
- **发送过程**：$u(t) = \sum u_k p(t - kT)$ 是将符号向量 ${u_k}$ 映射为信号波形的过程，相当于在由 ${p(t-kT)}$ 张成的信号空间中做线性组合。
- **接收过程**：通过匹配滤波 + 采样，本质上是在计算接收信号与各基函数的**内积（投影）**，从而提取出原始系数 $u_k$。

> **🧠 思维升华**：  
> "这个过程其实等价于去计算 $U(t)$ 到 ${p(t-kT)}$ 张成的信号空间的 projection。"  
> 
> 这句话打通了**时域滤波**与**向量空间投影**的任督二脉，是理解现代通信本质的关键。

![Nyquist准则与正交移位。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_14.png)  
![匹配滤波与正交性分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_15.png)  
![Nyquist准则与正交函数](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_16.png)  
![Nyquist准则推导。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_17.png)  
![奈奎斯特准则核心要点。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_18.png)  
![通信系统信号处理流程。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_19.png)

---

## **九、带宽效率分析：基带 vs. 带通**

### 9.1 数字基带传输
- 传输带宽 $B ≈ B_b = (R_s / 2)(1 + α)$
- 带宽效率：
  > $$η_{base} = R_s / B = 2 / (1 + α) ≤ 2 Baud/Hz$$

### 9.2 双边带调制（DSB）
- 带通信号带宽 $B = 2 × B_b = R_s (1 + α)$
- 带宽效率：
  > $$η_{pass} = R_s / B = 1 / (1 + α) ≤ 1 Baud/Hz$$

> **📌 结论**：  
> 调制会损失一半的带宽效率。这也是为何高效调制（如 SSB）在频谱受限场景中更受青睐。

![通信系统符号速率与带宽关系。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_12.png)

---

## **十、典型例题与应用总结**

> 教授点拨 · 发展脉络
> 奈奎斯特准则最早由 Harry Nyquist 在 1928 年提出，用于分析电报系统中的最大传输速率。它揭示了一个根本规律：信道带宽决定了符号传输的极限速度。这一思想后来成为香农信息论的基础之一。


> 注意$R_s$是码元发射速度，$R_s$ (Symbol Rate)：代表每秒钟向信道发送了多少个这样的物理脉冲。
> 不能发射太快
> $$R_b = R_s \cdot \log_2 M$$

### ✅ 给定符号速率，求最小带宽
- 例：$R_s = 100 Baud$，要求无 ISI
- 最小基带带宽：$B_b ≥ R_s / 2 = 50 Hz$
- 若用 DSB 调制，信道带宽至少需 $100 Hz$

### ✅ 给定带宽，求最大符号速率
- 例：基带系统 $B_b = 50 Hz$
- 最大 $R_s ≤ 2 B_b = 100 Baud$

![带宽与符号率关系。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_13.png)

### ✅ 如何判断能否无 ISI 传输？
三种等价方法：
1. **时域法**：检查 $g(kT)$ 是否满足 $δ[k]$
2. **频域法**：检查 $G(f)$ 折叠后是否为常数
3. **正交性法**：检查 ${p(t-kT)}$ 是否正交，并配合匹配滤波

![奈奎斯特准则示例。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1224b-1219-e-The%20Nyquist%20Criterion/page_20.png)

---

## **十一、结语：超越公式的思考**

> **"这里面就会有一个 trade-off。"**  
> —— 这是本节课最核心的思想。

通信工程的本质，就是在各种限制之间寻找最优平衡：
- **带宽 ↔ 速率**
- **效率 ↔ 可靠性**
- **理论完美 ↔ 工程可行**

奈奎斯特准则不仅是一条数学公式，更是一种**系统设计哲学**。它告诉我们：**没有免费的午餐**。每一次性能提升的背后，都是资源的重新分配与妥协。

希望这份笔记不仅能帮你通过考试，还能启发你像工程师一样思考问题。

---
**笔记整理完毕**  
祝各位复习顺利，期末取得佳绩！