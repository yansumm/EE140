# **通信原理：采样、正交展开与混叠现象**  
**主讲人：廉黎祥教授 | 课程：通信系统导论（EE140）| 上海科技大学 | 第12周**

---

## **课程介绍**

本节课围绕“**采样的本质**”展开，深入探讨了信号采样过程的数学基础——**正交展开理论**。通过将时间有限、频率有限、时间无限和带宽无限四类信号统一纳入正交基函数框架下分析，我们不仅重新理解了经典**采样定理**，还揭示了其背后深刻的工程意义。

更重要的是，课程进一步引出了**量化误差最小化**的设计哲学，并以概率建模视角为后续信源编码打下理论基础。最后，通过对**混叠现象**的时域与频域双重剖析，建立起对奈奎斯特准则必要性的直观认知。

> 📌 **教授点拨**：  
> “采样不是简单的‘拍照’，而是一次在特定基底下的投影。你选择什么样的‘镜头’（基函数），决定了你能多精确地还原原貌。”

![上海科技大通信系统课件。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_23.png)

---

## **核心概念：采样即正交展开**

### **1. 正交展开的基本思想**

在信号处理中，任何信号都可以被看作是在一组**正交基函数**上的线性组合。这种表示方法称为**正交展开**（Orthogonal Expansion）。  
设原始信号为 $ x(t) $，一组正交基函数为 $ \{\phi_k(t)\} $，满足：
$$
\langle \phi_k, \phi_l \rangle = \int \phi_k(t)\phi_l^*(t)dt = 
\begin{cases}
E_k, & k=l \\
0, & k \neq l
\end{cases}
$$
则信号可表示为：
$$
x(t) = \sum_{k=-\infty}^{\infty} c_k \phi_k(t)
$$
其中系数 $ c_k $ 可通过内积计算：
$$
c_k = \frac{1}{E_k} \int x(t) \phi_k^*(t) dt
$$

> 🔍 **深入理解**：  
> 系数 $ c_k $ 实际上就是信号在第 $ k $ 个方向上的“投影长度”。这就像三维空间中一个向量可以用三个坐标轴分量表示一样，只不过这里的“维度”是无穷维的函数空间。

![信号采样与重建问题](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_9.png)

---

### **2. 四类信号的采样模型统一于正交展开**

#### **(1) 时间有限信号 → 傅里叶级数展开**

- **定义**：若信号 $ x(t) $ 满足 $ x(t)=0 $ 当 $ |t| > T/2 $，称其为**时间有限信号**。
- **正交基**：在区间 $[-T/2, T/2]$ 上的一组复指数函数：
  $$
  \phi_k(t) = e^{j2\pi kt/T}, \quad t \in [-T/2, T/2]
  $$
  这些函数在该区间内正交，且能量 $ E_k = T $。
- **展开形式**：
  $$
  x(t) = \sum_{k=-\infty}^{\infty} c_k e^{j2\pi kt/T}
  $$
  其中 $ c_k $ 是傅里叶级数系数，也即采样值的一种抽象表示。

> ✅ **知识纠正**：  
> 转录稿中多次出现“负列级数”，应为“**傅里叶级数**”（Fourier Series），属同音误写。

![傅里叶级数与采样定理](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_5.png)

#### **(2) 频率有限信号（带限信号）→ 采样定理**

- **定义**：若信号 $ x(t) $ 的频谱 $ X(f) = 0 $ 当 $ |f| > W $，称其为**带宽受限信号**（Band-limited Signal）。
- **正交基**：sinc函数族：
  $$
  \phi_k(t) = \text{sinc}(2Wt - k), \quad \text{sinc}(x) = \frac{\sin(\pi x)}{\pi x}
  $$
  这些函数以 $ T = 1/(2W) $ 为周期平移，在整个实轴上构成正交集。
- **能量特性**：每个基函数的能量为 $ E_k = T = 1/(2W) $。
- **采样定理表达式**：
  $$
  x(t) = \sum_{k=-\infty}^{\infty} x\left(\frac{k}{2W}\right) \text{sinc}(2Wt - k)
  $$

> 🧩 **发展脉络**：  
> 这正是香农（Shannon）在1948年《通信的数学理论》中提出的著名结论，奠定了现代数字通信的基础。它告诉我们：只要采样率足够高，连续信号可以被完全离散化而不失真。

![带限信号与采样定理。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_10.png)

#### **(3) 时间无限信号 → 分段截断 + 傅里叶展开（T-Spaced Truncated Sinusoid Expansion）**

- **策略**：将无限长信号按时间段 $ T $ 截成若干段，每段独立做傅里叶级数展开。
- **双索引展开**：
  $$
  x(t) = \sum_{m=-\infty}^{\infty} \sum_{k=-\infty}^{\infty} c_{m,k} \cdot \phi_{m,k}(t)
  $$
  其中 $ m $ 表示第 $ m $ 个时间段，$ k $ 表示该段内的第 $ k $ 个傅里叶系数。
- **基函数形式**：
  $$
  \phi_{m,k}(t) = 
  \begin{cases}
  e^{j2\pi kt/T}, & t \in [(m-\frac{1}{2})T, (m+\frac{1}{2})T] \\
  0, & \text{否则}
  \end{cases}
  $$

> 🎯 **教授点拨**：  
> “这就是OFDM等现代调制技术的思想雏形——把一个大问题分解成多个小问题并行解决。”

![信号分段与采样分析](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_4.png)

#### **(4) 带宽无限信号 → 频域分段 + 移位采样定理（T-Spaced Sinc-Weighted Sinusoid Expansion）**

- **策略**：在频域将无限宽频谱按宽度 $ 2W $ 切割成多个带限片段，分别进行逆变换后采样。
- **关键变化**：由于各频段中心不在零频，对应时域信号会出现相位偏移（exp项）。
- **重建公式**：
  $$
  x(t) = \sum_{m=-\infty}^{\infty} \sum_{k=-\infty}^{\infty} v_{m,k} \cdot \text{sinc}(2Wt - k) \cdot e^{j2\pi (m/W)t}
  $$
  其中 $ v_{m,k} $ 是第 $ m $ 个频段在第 $ k $ 个采样点的值。

> 💡 **思路与方法**：  
> 频域平移 ↔ 时域相移，这是傅里叶变换的核心对偶性之一。理解这一点，就能明白为何此处必须引入复指数权重。

![正交展开与采样定理](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_6.png)

---

## **采样定理的再认识：从公式到物理意义**

### **1. 定理重述**

> 若信号 $ x(t) $ 的最高频率为 $ W $ Hz，则当采样频率 $ f_s \geq 2W $ 时，可用采样点 $ \{x(kT)\}, T=1/f_s $ 完全重建原始信号。


> SELF：时域的sinc来自于频域的抗混叠（低通）滤波器
> 时域的sinc来自于对频域进行低通滤波的rect，如果频域不进行低通滤波，则时域不弥散，成为δ函数

等价形式：
$$
x(t) = \sum_{k=-\infty}^{\infty} x(kT) \cdot \text{sinc}\left( \frac{t}{T} - k \right), \quad T = \frac{1}{2W}
$$

- $ T $：采样周期  
- $ f_s = 1/T = 2W $：奈奎斯特速率（Nyquist Rate）  
- sinc函数：理想低通滤波器的冲激响应  

![采样定理核心要点。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_11.png)

### **2. 两种重建方式的等价性**

| 方法       | 频域操作                   | 时域操作                     |
|------------|----------------------------|------------------------------|
| **滤波法** | 采样后频谱周期延拓 → 理想低通滤波 | 卷积 sinc 函数               |
| **插值法** | ——                         | 加权叠加平移 sinc 函数        |

> ✅ **深入理解**：  
> 两者本质相同！理想低通滤波器的冲激响应正是 sinc 函数。因此，“用采样点卷积 sinc” 和 “在频域乘矩形窗” 是同一过程的两种表述。

> 📚 **PPT对照**：  
> 图片3、11、13均展示了该公式的推导与图示，强调了 sinc 插值的几何直观。

![采样定理证明。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_1.png)  
![采样定理与信号重构](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_2.png)  
![采样定理证明过程。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_3.png)  
![采样与重建示意图](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_12.png)  
![频域采样与重建](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_13.png)

---

## **为什么使用正交展开？—— 量化误差解耦的关键**

### **1. 问题背景：从模拟到数字的全过程**

完整流程如下：
```
模拟信号 x(t)
    ↓ 采样（正交展开）
离散样本 {u_k}
    ↓ 量化（幅度离散化）
离散符号 {v_k}
    ↓ 编码（变长码）
比特流 {0,1}^n
```

接收端反向操作，但**量化不可逆**，只能用 $ v_k $ 重建波形。

![源波形编码步骤。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_7.png)

### **2. 重建误差分析**

令接收端重建信号为：
$$
\hat{x}(t) = \sum_k v_k \phi_k(t)
$$
真实信号为：
$$
x(t) = \sum_k u_k \phi_k(t)
$$
二者差值的平方误差（能量）为：
$$
\|\hat{x} - x\|^2 = \int |\hat{x}(t) - x(t)|^2 dt = \sum_k |u_k - v_k|^2 \cdot \|\phi_k\|^2
$$
若所有基函数具有相同能量 $ E = T $，则：
$$
\|\hat{x} - x\|^2 = T \sum_k |u_k - v_k|^2
$$

> ✅ **核心洞察**：  
> **波形重建误差完全由量化误差决定，且二者呈线性关系**！

### **3. 正交性的价值**

- 若基函数不正交，则交叉项不为零，误差传播复杂，难以优化。
- **正交基使得误差可分离，从而可以*独立设计每个量化器*来最小化整体失真**。

> 🧠 **教授点拨**：  
> “正交展开就像给每一根‘数据管道’加上绝缘层，防止它们相互干扰。这样你调一个参数，不会影响别的部分。”

> 📚 **PPT对照**：  
> 图片7、8清晰地展示了这一因果链：正交展开 → 解耦误差 → 最小化MSE → 高效编码。

![正交性与量化误差。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_8.png)

---

## **混叠现象（Aliasing）：采样不足的代价**

### **1. 发生条件**

当以下任一条件成立时，发生混叠：
- 信号带宽 $ W > f_s / 2 $
- 采样频率 $ f_s < 2W $
- 采样周期 $ T > 1/(2W) $

> ⚠️ **术语澄清**：  
> “混别”、“混蝶”均为“**混叠**”（Aliasing）的语音转录错误。

![信号采样与混叠现象。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_15.png)

### **2. 时域视角：采样点相同，信号不同**

考虑两个信号 $ x(t) $ 和 $ \tilde{x}(t) $，尽管它们整体不相等，但在采样点 $ t = kT $ 处取值相同：
$$
x(kT) = \tilde{x}(kT), \quad \forall k
$$
此时，若用这些采样点重建，得到的是 $ \tilde{x}(t) $ 而非 $ x(t) $。

> 🎯 **教授点拨**：  
> “就像一个人在固定的离散时间点拍合影，在每张照片中看起来一样，但中间的转移动作可以完全不同。”

### **3. 频域视角：频谱折叠与叠加**

原始信号频谱 $ X(f) $ 被以 $ f_s $ 为周期进行周期延拓：
$$
X_s(f) = \sum_{n=-\infty}^{\infty} X(f - n f_s)
$$
若 $ f_s < 2W $，相邻周期的频谱会发生重叠，导致无法分离出原始频谱。

具体表现为：
- 将频谱按 $ f_s $ 宽度切片
- 所有切片向 $ [-f_s/2, f_s/2] $ 区间折叠
- 折叠后的频谱直接相加，形成失真的 $ \tilde{X}(f) $

> ✅ **比喻理解**：  
> 想象一条长长的地毯（原始频谱），你要把它塞进一个小盒子（基带区间）。如果盒子太小，你就只能把多余的部分折回来叠上去——结果就是上下图案混在一起，再也分不清哪是哪了。

> 📚 **PPT对照**：  
> 图片15–19详细图示了混叠的频域表现，尤其是图片17中的数学推导，验证了“折叠+叠加”的结论。

![信号采样与混叠现象。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_16.png)  
![频率域混叠分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_17.png)  
![信号频谱分析图示。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_18.png)  
![频率域混叠现象分析](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_19.png)  
![信号频谱与采样定理。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_21.png)

---

## **信源编码前奏：量化与概率建模**

### **1. 采样输出的概率建模**

- 采样值 $ u_k $ 是连续随机变量（Continuous Random Variable）  
- 假设平稳且独立同分布（i.i.d.），记其概率密度函数（PDF）为 $ f_U(u) $  
- 示例：温度传感器读数、语音幅度等  

### **2. 量化过程的本质**

- 输入：连续随机变量序列 $ \{U_k\} $  
- 输出：离散随机变量序列 $ \{V_k\} $，取值于有限集合 $ \{v_1, ..., v_L\} $  
- 输出服从概率质量函数（PMF） $ P_V(v) $  

> 🤔 **经验提醒**：  
> “不要忽视 PMF 的重要性！它直接决定信源编码的压缩极限。熵越大，越难压缩。”

### **3. 量化目标：最小化失真**

常用失真度量：**均方误差**（Mean Squared Error, MSE）
$$
D = \mathbb{E}[(U - V)^2] = \int (u - Q(u))^2 f_U(u) du
$$
其中 $ Q(\cdot) $ 是量化器映射函数。

目标：设计 $ Q(\cdot) $ 使得 $ D $ 最小。

### **4. 两类量化方法**

| 类型         | 名称                   | 特点                                   |
|--------------|------------------------|----------------------------------------|
| 标量量化     | Scalar Quantization    | 逐点量化，简单高效                     |
| 向量量化     | Vector Quantization    | 分组量化，利用相关性，性能更优但复杂度高 |

> 📚 **PPT对照**：  
> 图片24、25给出了完整的量化流程框图及目标说明。

![量化过程示意图。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_24.png)  
![量化：目标、方法与失真度量。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_25.png)

---

## **总结与洞见**

| 主题             | 关键结论                                 | 工程启示                               |
|------------------|------------------------------------------|----------------------------------------|
| **采样本质**     | 采样是信号在正交基下的投影               | 设计采样方案即选择合适的“观测角度”     |
| **正交性优势**   | 实现重建误差与量化误差**解耦**               | 是实现最优编码的前提                   |
| **采样定理**     | $ f_s \geq 2W $ 可无失真重建             | 是数字系统的生命线                     |
| **混叠现象**     | 频谱折叠导致信息混淆                     | 必须前置抗混叠滤波器（实际就是一个**低通滤波器**）                   |
| **量化设计**     | 目标是最小化MSE                          | 决定了信源编码的效率上限               |

> 🔚 **教授结语**：  
> “今天我们打通了从模拟世界通往数字世界的桥梁。下节课我们将踏上这座桥，深入研究如何让每一步都走得更稳、更快。”

![傅里叶级数与采样定理](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_20.png)

---

## **附录：常见误区提醒**

❌ **误区1**：“只要采样点够密，总能还原信号。”  
✅ 正解：必须满足奈奎斯特条件。否则即使点再多，也可能陷入混叠陷阱。

❌ **误区2**：“sinc函数只是数学工具，没有物理意义。”  
✅ 正解：sinc函数是理想低通滤波器的冲激响应，代表了物理上最理想的重构能力。

❌ **误区3**：“量化误差无关紧要。”  
✅ 正解：它是数字系统中最主要的噪声来源之一，直接影响SNR和通信质量。

--- 

> *笔记整理完毕。建议结合PPT图片（特别是page_3, page_8, page_15）进行复习，强化对正交展开与混叠机制的理解。*

![上海科技大学感谢提问。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1203b-1128-e-sampling%20theorem/page_22.png)