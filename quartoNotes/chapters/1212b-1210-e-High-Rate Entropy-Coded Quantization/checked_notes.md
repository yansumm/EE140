# **通信原理：高码率熵编码量化与信号空间基础**  
**主讲人：廉黎祥教授 | 课程：通信系统导论（EE140）| 上海科技大学 | 第13周**

---

## **一、课程介绍与知识脉络**

![上海科技大通信系统课件。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_12.png)

本节课聚焦于**信源编码中的核心环节——量化（Quantization）**，深入探讨了在**高码率条件下如何设计最优的熵编码量化器**。课程从理论推导出发，结合实际电信系统（如电话语音）的应用背景，揭示了量化误差与编码速率之间的深刻权衡关系，并最终将视野拓展至**信号空间建模**这一现代数字通信的基石概念。

> **教授点拨**：  
> “我们学通信，不能只记住公式，要理解‘为什么’。比如为什么电话系统不用均匀量化？为什么六边形比正方形好但实际却用正方形？这些背后都是工程与理论的博弈。”

---

## **二、核心概念：高码率熵编码量化（High-Rate Entropy-Coded Quantization）**

### **1. 问题设定：什么是“熵编码量化”？**

在典型的信源编码流程中：
```
输入信号 u → 量化器 → 输出离散符号 v → 无损信源编码器 → 编码比特流
```

- 我们的目标是设计一个**最优量化器**，使得在给定编码速率 $ R $ 的前提下，**重建信号的失真（以均方误差 MSE 表示）最小**。
- 若后端的信源编码器是**熵编码器**（如霍夫曼编码、算术编码），其平均码长可逼近输出符号 $ v $ 的信息熵 $ H(V) $，即：
  $$
  R \approx H(V)
  $$

因此，我们可以将编码速率限制转化为对量化器输出熵的约束：
$$
\text{Minimize } \mathrm{MSE} \quad \text{subject to } H(V) = R
$$

满足此条件的量化器称为 **IB-optimal quantizer**（Information-Bottleneck Optimal Quantizer）或 **entropy-constrained quantizer**。

> **教授点拨**：  
> “这里的‘熵’不是抽象概念，它直接对应你每秒发多少比特。你要控制传输速率，就必须控制 $ H(V) $，这就是约束的物理意义。”

---

### **2. 高码率假设（High-Rate Assumption）**

直接求解上述优化问题是困难的。为此，我们引入**高码率假设**来简化分析：

> 当量化级数 $ M $ 极大时（即 $ R \gg 1 $ bit/symbol），每个量化区间非常小，区间内的概率密度函数 $ f_U(u) $ 可近似为常数。

更精确地说，在第 $ i $ 个量化区间 $ \mathcal{R}_i $ 内：
$$
f_U(u) \approx \bar{f}_i = \frac{\int_{\mathcal{R}_i} f_U(u) du}{\Delta_i}
$$
其中 $ \Delta_i $ 是该区间的长度。

这个假设极大简化了后续的 MSE 和熵的计算。

> **教授点拨**：  
> “高码率不是说信号本身信息多，而是你的表示精度很高。就像用显微镜看东西，每一小块看起来都差不多平——这就是局部均匀性的思想。”

---

![高码率熵编码量化。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_1.png)

![信息熵与编码率关系。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_2.png)

![信息率失真理论与MSE分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_3.png)

![高码率熵编码量化。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_4.png)

---

## **三、方法讲解：均匀量化器为何在高码率下最优？**

### **1. 均匀标量量化器（Uniform Scalar Quantizer）的性能分析**

考虑一个步长为 $ \Delta $ 的均匀量化器。在高码率假设下，可得以下两个关键结论：

#### **(1) 均方误差（MSE）**
$$
\mathrm{MSE} \approx \frac{\Delta^2}{12}
$$

**推导思路**：
- 每个量化区间内，输入 $ u $ 近似服从均匀分布。
- 量化误差 $ e = u - \hat{u} $ 在 $ [-\Delta/2, \Delta/2] $ 上均匀分布。
- 其方差即为 MSE，标准结果为 $ \mathrm{Var}(e) = \Delta^2 / 12 $。

> **深层理解**：  
> 这个 $ \frac{1}{12} $ 不是魔法数字，它是区间长度为 1 的均匀分布的方差。缩放后自然得到 $ \Delta^2/12 $。

#### **(2) 输出熵 $ H(V) $**
$$
H(V) \approx h(U) - \log_2 \Delta
$$
其中 $ h(U) $ 是输入连续信号 $ U $ 的**微分熵（Differential Entropy）**。

**推导思路**：
- $ P(V=i) = \int_{\mathcal{R}_i} f_U(u) du \approx f_U(u_i) \cdot \Delta $
- 因此 $ H(V) = -\sum P_i \log P_i \approx -\sum f_U(u_i)\Delta \cdot \log(f_U(u_i)\Delta) $
- 利用黎曼和逼近积分，最终得到 $ H(V) \approx h(U) - \log_2 \Delta $

> **教授点拨**：  
> “注意！$ h(U) $ 是微分熵，可能为负，但它和离散熵 $ H(V) $ 的差值是有意义的。你可以直观理解为：原始信号的不确定性减去你划分的精细程度，就是你编码需要的信息量。”

---

### **2. 编码速率与失真的权衡（Rate-Distortion Trade-off）**

将以上两式联立，消去 $ \Delta $，即可得到 **MSE 与编码速率 $ R $ 的关系**：

由 $ R = H(V) \approx h(U) - \log_2 \Delta $，解得：
$$
\Delta = 2^{h(U) - R}
$$

代入 MSE 公式：
$$
\mathrm{MSE} \approx \frac{1}{12} \left(2^{h(U) - R}\right)^2 = \frac{1}{12} \cdot 2^{2(h(U) - R)}
$$

取对数（转为 dB）：
$$
\log_2(\mathrm{MSE}) \approx 2h(U) - 2R - \log_2 12
$$

这是一个**斜率为 -2 的直线**！

> **关键洞见**：  
> - **每增加 1 比特/符号的编码速率，MSE 减少 6 dB**（因为 $ 2^{-2} = 1/4 $，即降低到 1/4，对应 6 dB）。
> - 这个 **6 dB/bit** 是高码率下均匀量化器的黄金法则。

> **趣闻实例**：  
> 教授举例：“就像你拍照，每多花一格内存，画面清晰度能翻几倍？这里告诉你：多 1 bit，噪声能量降为 1/4，听感上就是明显更干净了。”

---

### **3. 输入信号统计特性的影响**

观察权衡曲线：
$$
\mathrm{MSE} \propto 2^{-2R} \cdot 2^{2h(U)}
$$

- $ h(U) $ 越大（信号越‘分散’，如高斯分布），整个曲线**向上平移**。
- 在相同 $ R $ 下，$ h(U) $ 大意味着更大的 MSE；但在相同 MSE 下，也需要更高的 $ R $ 来编码。

> **教授点拨**：  
> “这解释了为什么压缩算法要针对信号类型优化。一个变化剧烈的语音段，天然需要更多比特来达到同样质量。”

---

## **四、高维扩展：二维均匀量化器的最优性**

### **1. 二维量化器的基本结构**

考虑将两个连续样本 $ (u_1, u_2) $ 一起量化。量化区域被划分为一个个基本单元（cell），如正方形、六边形等。

一个**均匀二维量化器**要求所有单元形状、大小相同，并能通过平移铺满整个平面。

> **合法单元的要求**：必须能通过平移实现**密铺（Tiling）**。常见选择：
> - 正方形（Square）
> - 六边形（Hexagon）
> - 三角形（Triangle）

> **教授点拨**：  
> “这不是几何游戏。如果单元不能密铺，中间就有缝隙或重叠，无法构成完整的量化器。”

---

![高维均匀量化器最优。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_5.png)

![高维量化熵分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_6.png)

### **2. 性能分析：MSE 与单元形状的关系**

对于任意形状的均匀二维量化单元，其单位面积的平均 MSE 为：
$$
\mathrm{MSE} = \frac{1}{A} \int_{\text{cell}} \| \mathbf{u} \|^2 d\mathbf{u}
$$
其中 $ A $ 是单元面积，积分中心设在原点（即量化点位于单元中心）。

- 该积分衡量的是**单元内所有点到中心的距离平方的平均值**。
- **越接近圆形的形状，该值越小**，因为点更“集中”。


#### **具体计算对比**：
> SELF：
> 注意：从严谨的数学对比来看，下面的计算中规定“边长相同”是不够公平的，甚至是有误导性的；真正有意义的对比必须规定“面积相同”（即等码率/等量化级数）。
> 在面积相同的情况下：$$MSE_{hex} (0.1424 A) < MSE_{square} (0.1667 A)$$六边形的量化误差比正方形低了约 14.6%。这证明了六边形在几何上确实更优。
> 
> 误区：认为六边形在任何维度下都最好。
> 事实：在更高维度的空间（如 $D=24$ 的 Leech Lattice），量化胞的形状会变得极其奇特且高效。正方形和六边形的对比仅仅局限在二维平面。

- **正方形**（边长 $ \Delta $，面积 $ A = \Delta^2 $）：
  $$
  \mathrm{MSE}_{\text{square}} = \frac{\Delta^2}{6}
  $$
- **六边形**（边长 $ \Delta $）：
  $$
  \mathrm{MSE}_{\text{hex}} \approx 0.370 \cdot \Delta^2 < \mathrm{MSE}_{\text{square}}
  $$

> **结论**：六边形优于正方形，**理论最优是圆形**（但不可密铺）。

> **教授点拨**：  
> “数学上六边形更好，但工程上呢？六边形的坐标映射、存储寻址都比正方形复杂得多。提升的那一点点性能，不值得付出的复杂度代价。所以现实中，**正方形足够好**。这和我们前面说的‘高维量化收益有限’是一个道理。”

---

### **3. 熵与权衡关系的延续**

二维量化器的输出熵（每符号）为：
$$
R \approx \frac{1}{2} \left( h(U_1, U_2) - \log_2 A \right)
$$
若 $ U_1, U_2 $ 独立同分布，则 $ h(U_1,U_2) = 2h(U) $，故：
$$
R \approx h(U) - \frac{1}{2} \log_2 A
$$

同样地，MSE 与 $ A $ 成正比。当面积 $ A $ 减为 1/4 时：
- $ R $ 增加 1 bit/symbol，
- MSE 减为 1/4（即降低 6 dB）。

> **统一规律**：无论是 1D 还是 2D，在高码率下，**每增加 1 bit/symbol 的速率，MSE 降低 6 dB**。

---

## **五、实际应用：电话系统中的非均匀量化（Companding）**

### **1. 为什么需要非均匀量化？**

语音信号的特点：
- 强信号（大声说话）：人耳对其失真不敏感。
- 弱信号（轻声细语）：人耳极其敏感，微小失真也易察觉。

**需求**：对弱信号使用小量化步长（低失真），对强信号允许大量化步长（高失真）。

但这与均匀量化矛盾。直接设计非均匀量化器硬件复杂。

### **2. 解决方案：压扩技术（Companding）**

利用一个简单的技巧：**用均匀量化器 + 非线性变换 实现非均匀量化效果**。

系统框图：
```
x → [Compressor] → y → [Uniform Quantizer] → ŷ → [Expander] → x̂
```

- **压缩器（Compressor）**：对输入 $ x $ 进行非线性压缩。
  - 小 $ |x| $ 区域：斜率 > 1，放大信号变化率，增大变化区间，在后续的均匀量化中增加代表点个数。
  - 大 $ |x| $ 区域：斜率 < 1，压缩信号变化率，缩小变化区间，在后续的均匀量化中减少代表点个数。
- **均匀量化器**：对被“拉伸”的 $ y $ 进行均匀量化。
- **扩展器（Expander）**：执行压缩的逆过程，恢复信号。

**效果**：
- 原始小信号 $ x $ 被放大成较大的 $ y $，在均匀量化网格中占据更多区间，**等效于小量化步长**。
- 原始大信号 $ x $ 被压缩成较小的 $ y $，占据较少区间，**等效于大量化步长**。

> **深层理解**：  
> 压扩的本质是**根据信号强度动态调整量化分辨率**，而无需改变硬件量化器。

---

![量化在电话系统中的应用。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_7.png)

![量化在电话系统中的应用。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_8.png)

![量化在电话系统中的应用。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_9.png)

### **3. 主流标准：μ律与A律**

- **μ律（mu-law）**：北美和日本采用。
  $$
  y = \frac{\ln(1 + \mu |x|)}{\ln(1 + \mu)} \cdot \mathrm{sgn}(x), \quad \mu = 255
  $$
- **A律（A-law）**：欧洲和中国采用。
  $$
  y = \begin{cases}
  \frac{A|x|}{1+\ln A} & |x| < 1/A \\
  \frac{1+\ln(A|x|)}{1+\ln A} & |x| \geq 1/A
  \end{cases}, \quad A = 87.6
  $$

> **教授点拨**：  
> “μ=255 和 A=87.6 都是经过大量实验调优的参数，能在大多数语音场景下取得最佳听觉体验。它们不是随便选的。”

---

## **六、知识总结与回顾**

### **1. 信源编码三大模块**

| 模块 | 关键概念 | 核心思想 |
|------|----------|----------|
| **采样** | 奈奎斯特采样定理、混叠（Aliasing） | 时间离散化，避免频谱混叠 |
| **量化** | MSE、熵、高码率近似、压扩 | 幅度离散化，平衡失真与码率 |
| **编码** | 香农熵 $ H(V) $、变长编码（Huffman）、最优码长 | 信息压缩，逼近熵极限 |

> **重要提醒**：  
> 对于 $ N $ 维向量量化器（如 $ N=2 $），需掌握如何设计量化区域、计算联合熵 $ H(V_1,V_2) $ 及对应的权衡曲线。

---

### **2. 信号空间：通信的向量视角**

现代通信将信号视为**向量空间中的点**，这是理解调制、检测、编码的基础。

#### **(1) 信号空间（Signal Space）**
- 所有有限能量信号 $ u(t) \in L^2(\mathbb{R}) $ 构成一个**希尔伯特空间（Hilbert Space）**。
- 任何信号可表示为一组**正交基函数**的线性组合，系数构成一个向量。

#### **(2) 向量空间公理**
信号空间满足向量空间的所有性质：
- 加法交换律、结合律
- 标量乘法分配律
- 存在零向量和负向量

#### **(3) 内积（Inner Product）与几何**
定义信号 $ s_1(t) $ 和 $ s_2(t) $ 的内积：
$$
\langle s_1, s_2 \rangle = \int s_1(t) s_2^*(t) dt
$$
由此可定义：
- **能量（范数平方）**: $ E = \|s\|^2 = \langle s, s \rangle $
- **相关性（夹角余弦）**: $ \cos \theta = \frac{\langle s_1, s_2 \rangle}{\|s_1\| \|s_2\|} $
- **正交性**: $ \langle s_1, s_2 \rangle = 0 $

#### **(4) 投影与 Gram-Schmidt 正交化**
- 信号 $ v $ 在信号 $ u $ 方向上的投影为：
  $$
  \mathrm{proj}_u(v) = \frac{\langle v, u \rangle}{\langle u, u \rangle} u
  $$
- **Gram-Schmidt 过程**：从任意一组信号出发，构造一组正交（或标准正交）基。

> **教授点拨**：  
> “为什么我们要把信号当向量？因为一旦进入向量空间，所有线性代数的工具都为你所用。距离、角度、投影……这些几何直觉能让你一眼看穿复杂的通信问题。下一讲的信道编码，就建立在这个基础之上。”

---

![向量空间与信号分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_13.png)

![向量空间公理与性质。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_14.png)

![内积空间定义与性质。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_15.png)

![二维向量空间概念与公式。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_16.png)

![一维投影公式与分解。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_17.png)

![内积空间与不等式关系。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_18.png)

![线性相关与无关。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_19.png)

![向量空间基的概念与性质。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_20.png)

![有限维投影公式。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_21.png)

![Gram-Schmidt正交化过程。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_22.png)

---

## **七、结语与展望**

本节课完成了**信源编码部分的全部内容**，建立了从模拟信号到数字比特的完整链条。我们不仅掌握了量化器的设计原则，更学会了如何在理论最优与工程实现之间做出权衡。

**下节课预告**：我们将进入**信道编码解码**部分，探讨如何在有噪信道中可靠传输信息，揭开香农第二定理的神秘面纱。

> **课后任务**：
> 1. 完成本次作业（涉及信号空间的基与投影计算）。
> 2. 预习新的作业题，为下节课做准备。

---
**上海科技大学 · 创新与管理学院 · 通信系统导论**  
*整理者注：本笔记严格依据课堂录音与PPT内容，经核实与修正后生成，力求还原教学精髓。*

![上海科技大学感谢提问。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1212b-1210-e-High-Rate%20Entropy-Coded%20Quantization/page_11.png)