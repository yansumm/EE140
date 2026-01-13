# **通信原理：量化理论与应用——从通信原理到现代AI**

> *整理自 廉黎祥 教授《通信原理》第13周课程 | 上海科技大学创管学院*

---

## **课程介绍**

本节课系统回顾了**标量量化（Scalar Quantization）** 与**向量量化（Vector Quantization, VQ）** 的核心理论，并深入探讨其在信息论、信源编码以及现代人工智能中的深刻联系。课程不仅涵盖量化器设计的经典算法（如Lloyd-Max），还引入了**熵约束下的最优量化（Entropy-Coded Quantization）** 和**高码率假设（High-Rate Assumption）** 等高级概念。

教授通过清晰的数学推导与生动的类比，揭示了“离散化”这一基本操作如何贯穿于从传统通信系统到前沿大模型的技术脉络之中。尤其值得强调的是，**向量量化不仅是图像压缩的基石，更是理解VQ-VAE、Codebook-based生成模型乃至聚类算法的关键桥梁**。

![标量量化原理与公式推导。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_1.png)

---

## **第一部分：标量量化基础回顾**

### **1.1 量化过程建模**

量化是将一个连续随机变量 $ U $ 映射为一个离散随机变量 $ V $ 的过程。该过程由三个核心要素定义：
> 下面这三个要素可能有些问题，传统意义上的三要素是：M、$ \{R_j\} $、$ \{a_j\} $
1.  **量化区间（Quantization Regions）$ \{R_j\} $**: 将输入 $ u $ 的取值空间划分为 $ M $ 个互不相交的子区间。
2.  **量化电平/代表点（Quantization Levels / Representatives）$ \{a_j\} $**: 每个区间 $ R_j $ 对应一个输出值 $ a_j $。
3.  **量化规则**: 若输入 $ u \in R_j $，则输出 $ v = a_j $。

经过量化后，得到离散序列 $ V $，其概率质量函数（PMF）可通过原连续变量的概率密度函数（PDF） $ f_U(u) $ 在对应区间上的积分求得：
$$
P(V = a_j) = Q_j = \int_{R_j} f_U(u) \, du
$$

---

### **1.2 核心性能指标**

量化器的设计围绕两个关键目标展开：

#### **(1) 量化输出的熵 $ H(V) $**
-   **意义**: 衡量量化输出的不确定性，直接决定了对其进行**无损信源编码**时所需的平均比特数下限。
-   **计算公式**:
    $$
    H(V) = -\sum_{j=1}^{M} Q_j \log_2 Q_j
    $$
-   **重要性**: $ H(V) $ 是后续变长编码（如Huffman编码）的理论依据，也是衡量编码效率的基准。

#### **(2) 均方误差 $ MSE $ (或称失真 $ D $)**
-   **意义**: 衡量量化过程引入的失真，即重建信号与原始信号之间的平均平方偏差，是重构质量的核心度量。
-   **计算方法**:
    1.  **直接积分法**:
        $$
        MSE = E[(U - V)^2] = \sum_{j=1}^{M} \int_{R_j} (u - a_j)^2 f_U(u) \, du
        $$
    2.  **条件期望分解法**:
        $$
        MSE = \sum_{j=1}^{M} P(R_j) \cdot E[(U - a_j)^2 | U \in R_j] = \sum_{j=1}^{M} Q_j \cdot MSE_j
        $$
        其中 $ MSE_j $ 是在区域 $ R_j $ 内的**条件均方误差**。

> **📌 教授点拨：权衡的艺术**  
> 量化本质上是一个 **`rate-distortion trade-off`** 问题。降低 $ MSE $（提高质量）通常需要更多的量化级 $ M $，这会导致 $ H(V) $ 增加（更高的编码速率）。反之，追求低码率会牺牲信号质量。**理解这个权衡关系是设计任何实际量化系统的出发点**。

---

### **1.3 Lloyd-Max 算法：最优标量量化器的设计**

当唯一目标是最小化均方误差 $ MSE $ 时，可使用 **Lloyd-Max 算法** 迭代地设计最优的 $ \{R_j\} $ 和 $ \{a_j\} $。

#### **算法核心思想与必要条件**

该算法基于以下两个在最优解处必须满足的**必要条件**（但非充分条件）：

1.  **Nearest Neighbor Rule (最近邻法则)**: 给定一组代表点 $ \{a_j\} $，最优的划分边界是相邻代表点的**垂直平分线**。这意味着对于任意输入 $ u $，应将其分配给距离最近的 $ a_j $。
2.  **Centroid Condition (质心条件)**: 给定一组划分区间 $ \{R_j\} $，使 $ MSE $ 最小的代表点 $ a_j $ 是该区间内 $ U $ 的**条件期望**：
    $$
    a_j^* = E[U | U \in R_j] = \frac{\int_{R_j} u f_U(u) \, du}{\int_{R_j} f_U(u) \, du}
    $$

#### **算法流程**

Lloyd-Max 算法通过交替优化上述两个条件来迭代收敛：

1.  **初始化**: 随机选择 $ M $ 个初始代表点 $ \{a_j\} $。
2.  **划分区间 (Update $ R_j $)**: 基于当前的 $ \{a_j\} $，根据最近邻法则划分量化区间 $ \{R_j\} $。
3.  **更新代表点 (Update $ a_j $)**: 基于新划分的 $ \{R_j\} $，计算每个区间的条件期望，更新 $ \{a_j\} $。
4.  **迭代**: 重复步骤2和3，直到 $ \{a_j\} $ 或 $ MSE $ 的变化小于预设阈值。

> **⚠️ 重要提醒**  
> 如图所示，Lloyd-Max 算法只能保证找到**局部最优解 (Local Optimum)**，而非全局最优。最终结果高度依赖于初始点的选择。但在大多数实际问题中，它已被证明是非常有效的。

![Lloyd-Max算法要点。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_2.png)

---

### **1.4 实例分析：1比特均匀量化器**

考虑一个经典例子：对 $ U \sim \text{Uniform}(0,1) $ 设计一个 $ M=2 $ 的标量量化器以最小化 $ MSE $。

1.  **初始猜测**: 直觉上，将区间在中心 $ b_1 = 0.5 $ 处分割。
2.  **计算条件PDF**:
    -   $ R_1 = [0, 0.5), R_2 = [0.5, 1] $
    -   $ Q_1 = Q_2 = 0.5 $
    -   条件PDF: $ f_{U|R_j}(u) = \frac{f_U(u)}{Q_j} = \frac{1}{0.5} = 2 $ （在 $ R_j $ 内）
3.  **应用质心条件**:
    -   $ a_1^* = E[U | U \in R_1] = \int_0^{0.5} u \cdot 2 \, du = \left[ u^2 \right]_0^{0.5} = 0.25 $
    -   $ a_2^* = E[U | U \in R_2] = \int_{0.5}^{1} u \cdot 2 \, du = \left[ u^2 \right]_{0.5}^{1} = 1 - 0.25 = 0.75 $
4.  **验证最近邻法则**: 新的边界应在 $ a_1 $ 和 $ a_2 $ 的中点 $ (0.25 + 0.75)/2 = 0.5 $，与初始猜测一致。
5.  **计算最终 $ MSE $**:
    -   利用均匀量化公式：$ MSE = \frac{\Delta^2}{12} $，其中 $ \Delta = 0.5 $ 是区间宽度。
    -   $ MSE = \frac{(0.5)^2}{12} = \frac{1}{48} \approx 0.0208 $

> **✅ 结论**  
> 如图所示，最优的1比特量化器具有阈值 $ b_1 = 1/2 $ 和重建电平 $ a_1 = 1/4, a_2 = 3/4 $，最小 $ MSE = 1/48 $。此例完美演示了Lloyd-Max算法的快速收敛。

![1比特量化最小化MSE。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_3.png)

---

## **第二部分：向量量化（Vector Quantization, VQ）**


> **SELF：从标量量化到向量量化**
> 向量量化区别于标量量化，标量量化下，样本之间独立量化，而向量量化中，将几个样本联合量化，利用样本之间的相关性，把联合分布密集的地方量化也做的更好更密，
> 但是如果多个样本之间的相关性较低甚至独立，则向量量化相比标量量化的增益就变得极小，同时向量量化带来的计算复杂度极大。
> 故实际应用中需要根据相关性决定是向量量化还是标量量化。


### **2.1 从标量到向量：维度的跃迁**

标量量化一次只处理一个样本。**向量量化**则将 $ N $ 个连续的样本 $ \mathbf{u} = (u_1, u_2, ..., u_N) $ 视为一个 $ N $-维向量进行整体量化。

-   **输入**: $ N $-维向量 $ \mathbf{u} \in \mathbb{R}^N $
-   **输出**: 一个索引 $ j $，指向码本（Codebook）中的第 $ j $ 个码字（Codeword） $ \mathbf{a}_j $。

---

### **2.2 二维Lloyd-Max算法与Voronoi图**

以 $ N=2, M=4 $ 为例，假设 $ U_1, U_2 \sim \text{Uniform}(-1,1) $ 且独立。

1.  **输入空间**: 输入向量 $ (u_1, u_2) $ 落在一个 $ [-1,1] \times [-1,1] $ 的正方形区域内。
2.  **联合PDF**: 由于独立，联合PDF $ f_{U_1,U_2}(u_1,u_2) = f_{U_1}(u_1) \cdot f_{U_2}(u_2) = \frac{1}{4} $，在整个正方形内为常数。
3.  **Lloyd-Max 迭代**:
    -   **初始化**: 随机选择4个初始码字位置。
    -   **划分区域**: 对每一对码字，画出它们的**垂直平分线**。所有平分线的交集将平面划分为若干多边形区域。
    -   **Voronoi Region**: 每个码字 $ \mathbf{a}_j $ 所对应的区域 $ R_j $ 称为**Voronoi区域**，包含所有离 $ \mathbf{a}_j $ 最近的点。
    -   **更新码字**: 计算每个 $ R_j $ 内的**条件期望** $ \mathbf{a}_j^* = E[\mathbf{U} | \mathbf{U} \in R_j] $，并移动码字至此。
4.  **收敛**: 迭代直至码字位置稳定。

> **📌 图解说明**  
> 下图直观展示了这一过程：初始码字（红点）划分出初始Voronoi区域，计算各区域质心得到新码字（黑叉），再以新码字重新划分，如此往复，最终形成规整的六边形或矩形网格（对于均匀分布）。

![矢量量化原理与算法](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_4.png)

![矢量量化迭代过程。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_5.png)

---

### **2.3 向量量化的深层洞见与现代应用**

#### **(1) 与机器学习的惊人联系**

> **教授点拨：超越通信的视野**  
> “同学们，你们可能觉得这只是老掉牙的通信技术。但我要告诉你们，**向量量化是理解现代AI的一把钥匙**。”

-   **聚类 (Clustering)**: VQ 的本质就是**K-means聚类**！码字 $ \mathbf{a}_j $ 就是聚类中心（质心），Voronoi区域就是聚类簇。Lloyd-Max算法与K-means算法完全同构。
-   **表示学习 (Representation Learning)**: 在深度学习中，我们希望学习数据的“好”表示。VQ 强制将连续的特征空间离散化为有限个“原型”（码字），这本身就是一种强大的表示学习机制。
-   **大模型 (LLMs) 中的应用**: 当前的大语言模型（LLM）处理的是离散的文本。若要用LLM生成图像，就必须先将连续的像素空间离散化。**向量量化正是实现这种跨模态转换的核心技术之一**。

#### **(2) VQ-VAE：向量量化变分自编码器**

> **📌 实例解析**  
> 如图所示，VQ-VAE 是一个革命性的生成模型架构：
>
> 1.  **Encoder**: 将输入图像映射到一个连续的潜在向量 $ \mathbf{z}_e $。
> 2.  **Quantizer (VQ Layer)**: 在预定义的**码本 (Codebook)** $ \mathcal{C} = \{\mathbf{e}_1, \mathbf{e}_2, ..., \mathbf{e}_K\} $ 中，找到与 $ \mathbf{z}_e $ 最接近的码字 $ \mathbf{e}_k $，并用 $ k $ 的one-hot编码作为离散表示 $ \mathbf{z}_q $。
> 3.  **Decoder**: 将离散表示 $ \mathbf{z}_q $ 解码回图像。
>
> **这里的码本 $ \mathcal{C} $ 就是我们量化器的 $ \{ \mathbf{a}_j \} $！** 而码本本身是通过反向传播与整个网络一起学习的，目标是最小化重建误差。这完美体现了“量化器不再是固定的，而是可学习的”这一现代思想。

> **理解VQ-VAE中的码本中的向量，以及Latent Space的大小**：
> - 码本中的所有向量维度必须完全一样。
> - 维度 $D$ 在结构上确实类似于原图像的 RGB 通道，但其代表的含义从“原始像素值”升华为“高维语义特征”。
> - Latent Space的总维度是 $ K^{H\times W}$

> **从VAE到VQ-VAE**：
> 在 VQ-VAE 出现之前，传统的 VAE 使用连续的高斯分布 $z \sim \mathcal{N}(\mu, \sigma^2)$。但这存在两个严重问题：
>   - 后验崩溃（Posterior Collapse）：强大的 Decoder 可能会忽略隐变量 $z$，导致隐空间失去表达力。
>   - 计算浪费：图像中的很多结构（如物体的边缘、颜色块）本质上是具有类别特征的，用连续分布去逼近这些“台阶式”的特征效率极低。
> 
> VQ-VAE相比VAE的优势：
>   - 组合爆炸：虽然码本大小 $K$ 是有限的（例如 512 或 1024），但如果隐空间大小是 $32 \times 32$，那么总的组合数是 $K^{1024}$。这个空间之大，足以覆盖极其复杂的图像分布。
>   - 避开维数灾难：在离散空间上通过 Transformer 学习分布，比在超高维连续空间上学习多模态分布要稳定得多。

![向量量化研究要点](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_6.png)

![VQ-VAE 架构示意图](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_8.png)

---

### **2.4 向量量化的挑战与适用场景**

尽管强大，VQ也面临严峻挑战：

| 挑战 | 说明 |
| :--- | :--- |
| **指数复杂度** | 码本大小 $ M $ 和维度 $ N $ 呈指数关系 ($ M \propto e^N $)，搜索和存储成本极高。 |
| **严重局部最优** | 高维空间中存在大量局部极小值，Lloyd-Max算法极易陷入次优解。 |
| **增益有限（对独立信源）** | 如果输入样本 $ u_i $ 之间相互独立，则VQ相比标量量化带来的 $ MSE $ 改善非常有限。 |

> **💡 关键洞见：相关性是VQ的生命线**  
> “那么，什么情况下VQ才真正值得做？”教授提出了一个深刻的问题。
>
> **答案是：当输入数据具有强相关性时**。
>
> **举例对比**:
> -   **情况一 (独立)**: $ U_1, U_2 \sim \text{Uniform}(-1,1) $ 且独立。输入点均匀分布在 $ 2\times2 $ 正方形内。$ MSE \propto \Delta^2 = 1 $。
> -   **情况二 (强相关)**: $ U_1 \sim \text{Uniform}(-1,1), U_2 = U_1 $。输入点被限制在一条斜率为1的直线上。此时，即使 $ M=4 $，量化区间沿直线划分，$ \Delta $ 可减半。$ MSE \propto (\Delta/2)^2 = 0.25 $，显著降低！
>
> **现实意义**: 图像、音频、视频等自然信号在时间和空间上都具有极强的相关性。因此，VQ在这些领域的压缩和生成任务中能发挥巨大威力。

![量化误差分析与计算。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_7.png)

---

## **第三部分：熵约束下的量化（熵编码量化）**

### **3.1 问题提出：超越MSE的优化目标**

前面的Lloyd-Max算法仅优化 $ MSE $，忽略了编码效率。在实际系统中，我们往往有**固定的编码速率 $ R $**（例如，每样本2比特）。此时，目标变为：
> **在保证量化输出熵 $ H(V) \leq R $ 的前提下，最小化均方误差 $ MSE $**。

这类量化器被称为 **`熵编码量化器 (Entropy-Coded Quantizer)`**。

> **📌 双重视角理解**  
> 一个最优的熵编码量化器可以从两个等价的角度理解：
> 1.  **约束优化视角**: 它是上述带熵约束的优化问题的解。
> 2.  **编码效率视角**: 当该量化器的输出 $ V $ 跟随一个**熵编码器（如Huffman编码）** 时，该编码器能达到**香农极限**（Shannon Limit），即平均码长 $ L_{avg} = H(V) $，实现了无冗余编码。

![熵编码量化概念](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_9.png)

![熵编码量化问题与解决方案。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_10.png)

---

### **3.2 高码率假设（High-Rate Assumption）**

直接求解上述优化问题是极其困难的。为此，我们引入一个强大的简化假设——**高码率假设**。

#### **核心假设内容**

1.  编码速率 $ R $ 很高（即 $ H(V) $ 很大）。
2.  量化级数 $ M $ 非常大。
3.  量化间隔 $ \Delta $ 非常小。
4.  **最关键的一条**: 每个量化区间足够小，以至于在该区间内，原PDF $ f_U(u) $ 可以近似为一个**常数** $ \bar{f}_j $。

> **📌 直观理解**  
> “想象一下数值积分。”教授解释道，“当你把积分区间划分得足够细时，曲线下面积就可以用一个个小矩形来近似。这里的‘高码率’就相当于让每个‘小矩形’的宽度趋近于零。”

![高码率量化器分析](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_11.png)

![高率量化器优化分析](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_12.png)

---

### **3.3 核心结论：均匀量化的最优性**

在高码率假设下，我们可以得出一个优美而实用的结论：

> **对于熵编码量化，均匀标量量化器（Uniform Scalar Quantizer）是接近最优的。**

#### **数学推导摘要**

1.  **计算 $ MSE $**:
    -   在每个小区间 $ R_j $ 内，$ f_U(u) \approx \bar{f}_j $。
    -   区间内的条件PDF是均匀的，因此最优代表点 $ a_j $ 为区间中点。
    -   区间内的条件 $ MSE_j = \frac{\Delta_j^2}{12} $。
    -   总 $ MSE \approx \sum_j \bar{f}_j \Delta_j \cdot \frac{\Delta_j^2}{12} = \frac{1}{12} \sum_j \bar{f}_j \Delta_j^3 $。

2.  **计算 $ H(V) $**:
    -   $ P(V=a_j) = Q_j = \int_{R_j} f_U(u) du \approx \bar{f}_j \Delta_j $。
    -   $ H(V) = -\sum_j Q_j \log_2 Q_j \approx -\sum_j \bar{f}_j \Delta_j \log_2(\bar{f}_j \Delta_j) $。
    -   在高码率下，可进一步近似为：
        $$
        H(V) \approx h(U) - \log_2 \Delta
        $$
        其中 $ h(U) = -\int f_U(u)\log_2 f_U(u) du $ 是 $ U $ 的微分熵。

3.  **优化结论**:
    -   通过拉格朗日乘子法，在约束 $ H(V) = R $ 下最小化 $ MSE $。
    -   **最终发现，当所有 $ \Delta_j $ 相等（即采用均匀量化）时，$ MSE $ 最小**。

> **📌 结论的意义**  
> 这意味着，在高码率条件下，复杂的非均匀量化带来的性能增益微乎其微。**简单的均匀量化器就能逼近理论最优性能**。这也是为什么在许多实际系统（如PCM音频）中，均匀量化被广泛采用。

![均匀标量量化器特性与优势。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_13.png)

![高率熵编码量化分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_14.png)

![高率熵编码量化分析](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_15.png)

![高码率熵编码量化。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_16.png)

---

### **3.4 性能曲线与设计指导**

综合以上分析，我们可以描绘出 $ MSE $ 与 $ H(V) $ 的关系曲线：

-   **横轴**: 熵 $ H(V) $ 或编码速率 $ R $。
-   **纵轴**: 均方误差 $ MSE $。
-   **曲线趋势**: 随着 $ R $ 增加，$ MSE $ 单调递减。

> **📌 教授点拨：Lower Bound 的启示**  
> 在高码率下，均匀量化器的性能曲线提供了一个**近乎最优的下界 (near-optimal lower bound)**。任何其他量化方案的性能都不应低于此曲线太多。这为系统设计者提供了明确的性能预期。

![信息论与编码理论核心公式。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1210b-1205-e-Lloyd-Max%20Algorithm/page_17.png)

---

## **总结与展望**

本节课构建了一个从基础到前沿的完整知识框架：

1.  **基础**: 掌握了标量量化的核心概念、Lloyd-Max算法及其局限性。
2.  **拓展**: 理解了向量量化的工作原理、巨大潜力及其在高维空间面临的挑战，特别是**相关性是发挥VQ优势的关键**。
3.  **深化**: 学习了在实际约束（熵）下设计量化器的方法，并通过**高码率假设**洞察了**均匀量化器的近似最优性**这一深刻结论。
4.  **贯通**: 认识到量化理论不仅是通信的基石，更是连接聚类、表示学习和现代生成式AI（如VQ-VAE）的桥梁。

> **最后的思考**  
> “技术看似在飞速迭代，但底层的数学原理却历久弥新。”廉教授总结道，“今天讲的‘离散化’、‘最近邻’、‘质心’，这些思想一百年后依然有用。希望大家不仅能记住公式，更能体会到其中蕴含的智慧。”

下节课将开启新章节，敬请期待。