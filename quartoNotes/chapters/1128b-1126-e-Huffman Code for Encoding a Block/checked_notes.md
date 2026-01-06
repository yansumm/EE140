# **通信原理深度学习笔记**  
**课程：EE140 通信系统导论 | 授课教师：廉黎祥 教授（上海科技大学）| 时间：2025年秋季学期 第11周**

---

## **课程介绍与目标**

![上海科技大课程介绍。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_15.png)

本节课是“信源编码”部分的总结与升华，随后正式进入“模拟信号采样”的核心内容。课程从信息论的基础出发，系统梳理了无损信源编码的核心理论、经典算法及其实际应用，并以**JPEG图像压缩**和**GIF动图压缩**为实例，揭示了现代数字通信中数据压缩的本质逻辑。

> 💡 **教授点拨 · 学习视角转换**  
> “我们这门课的重点不是记住公式，而是理解**为什么需要这些技术**，以及它们在真实世界中的‘生存方式’——比如一张图片是怎么被一步步压小到可以发微信而不明显失真的。”

---

## **第一部分：信源编码回顾与深化**

### **1. 基础概念：什么是信源编码？**

信源编码的目标是在**不丢失信息的前提下**，用尽可能少的比特表示原始数据。其理论极限由香农信息熵 $ H(X) $ 决定。

- **信源**：产生符号序列的数据源（如文本、语音、图像像素）。
- **编码器**：将原始符号映射为二进制比特流。
- **无损编码 (Lossless Coding)**：要求接收端能完全恢复原始数据。
- **信息熵 $ H(X) $**：衡量信源平均信息量的最小期望比特数：
  $$
  H(X) = -\sum_{i=1}^{n} p_i \log_2 p_i
  $$

> 📌 **关键提醒 · 概念辨析**  
> 信息熵是一个**统计下界**，并非所有情况下都能达到。能否逼近该极限取决于信源的概率分布是否满足特定条件。

---

### **2. 可变长编码 (Variable-Length Coding, VLC) 与 Kraft 不等式**

#### **Kraft 不等式：前缀码存在的充要条件**

对于一组码字长度 $ l_1, l_2, ..., l_n $，存在一个唯一可解的前缀码当且仅当：
$$
\sum_{i=1}^{n} 2^{-l_i} \leq 1
$$

> 🔍 **深入理解 · 为什么是这个形式？**  
> 这个不等式本质上反映了**二叉树的空间约束**。每个码字对应一棵二叉树的叶子节点，$ 2^{-l_i} $ 表示该叶子所占据的“子树比例”。总和不超过1，意味着所有叶子不会超出整棵树的容量。

> ⚠️ **常见误区纠正**  
> 即使某个编码方案的平均码长 $ L $ 满足 $ H(X) \leq L < H(X)+1 $，也不能说明它是最优或唯一的！必须检查是否满足 Kraft 不等式并验证其唯一可解性。

---

### **3. 最优编码的极限：信源编码基本定理**

> ✅ **定理名称更正**  
> 语音稿中“相同地定理”应为 **香农第一定理 (Shannon's First Theorem)** 或 **无损信源编码定理**。

![香农第一定理。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_12.png)

#### **香农第一定理（无损）**

设离散无记忆信源 $ X $ 的熵为 $ H(X) $，则：

1. **存在性**：对任意编码速率 $ R > H(X) $，总存在一种无损编码方式使得平均码长 $ \bar{L} < R $。
2. **不可能性**：若编码速率 $ R < H(X) $，则任何无损编码都将以概率1出现错误（即必然有损）。

因此，**信息熵 $ H(X) $ 是所有无损编码的平均码长下限**，无论编码方式多么复杂。

> 💡 **教授洞见 · 理论的意义**  
> “这个定理告诉我们：**压缩是有极限的**。你可以无限逼近 $ H(X) $，但永远不能突破它。就像光速之于物理世界，信息熵是数据世界的‘光速’。”

---

### **4. 如何计算最短平均码长 $ \bar{L}_{\min} $？**

这是学生最容易混淆的问题。答案取决于信源的概率分布：

| 情况 | 是否可达 $ H(X) $ | 如何计算 $ \bar{L}_{\min} $ |
|------|---------------------|-------------------------------|
| 所有 $ p_i = 2^{-k_i} $（即 $ p_i $ 是2的负整数次幂） | ✅ 可达 | $ \bar{L}_{\min} = H(X) $ |
| 否则 | ❌ 不可达 | 必须使用最优编码算法（如霍夫曼编码）计算实际能达到的最小 $ \bar{L} $ |

> 🧠 **思路解析 · 为何如此设计？**  
> 当 $ p_i = 2^{-k_i} $ 时，我们可以直接令码长 $ l_i = -\log_2 p_i = k_i $，此时 Kraft 和恰好等于1，完美填满二叉树，实现熵极限。

> 🛠️ **操作建议**  
> 在考试或作业中，若未明确说明概率分布形式，**不要直接用 $ H(X) $ 作为答案**，应说明需通过霍夫曼编码等方法求解。

---

## **第二部分：经典编码算法详解**

### **1. 霍夫曼编码 (Huffman Coding)**

![Huffman编码原理与应用](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_1.png)

霍夫曼编码是一种构造**最优前缀码**的经典贪心算法。

#### **算法步骤**
1. 将所有符号按概率从小到大排序。
2. 取出两个概率最小的符号，合并为一个新节点，其概率为二者之和。
3. 将新节点插入原序列，保持有序。
4. 重复步骤2–3，直到只剩一个根节点。
5. 从根节点回溯，分配0/1，得到各符号的码字。

#### **性能分析**
- 平均码长 $ \bar{L} $ 满足：  
  $$
  H(X) \leq \bar{L} < H(X) + 1
  $$
- 编码效率 $ \eta = \frac{H(X)}{\bar{L}} $

#### **块编码 (Block Coding) 提升效率**

将 $ N $ 个符号组合成一个“超级符号”进行编码：

- 联合熵：$ H(X^N) = N \cdot H(X) $ （因独立同分布）
- 块编码最小平均码长：$ \bar{L}_N $
- 单个符号平均码长：$ \frac{\bar{L}_N}{N} $
- 极限性质：
  $$
  H(X) \leq \frac{\bar{L}_N}{N} < H(X) + \frac{1}{N}
  $$
  当 $ N \to \infty $ 时，$ \frac{\bar{L}_N}{N} \to H(X) $

> 📊 **实例对比（来自PPT）**
>
> | 编码方式 | 平均码长 (bit/symbol) | 编码效率 |
> |--------|------------------|---------|
> | 符号级 Huffman | 1.55 | 97.6% |
> | 块大小 N=2 | 1.534 | 更高 |
> | 块大小 N→∞ | → H(X) | →100% |
>
> 结论：**增大块大小可逼近熵极限**，但代价是复杂度指数增长。

![Huffman编码效率对比。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_3.png)

---

### **2. 霍夫曼编码的实际应用：JPEG 图像压缩流程**

> 🔗 **连接理论与现实**  
> JPEG 是霍夫曼编码在工程实践中最著名的应用之一。

![Huffman编码应用流程图](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_4.png)

#### **完整流程（四步走）**

```mermaid
graph LR
A[原始高清图像] --> B[DCT 变换]
B --> C[量化]
C --> D[熵编码<br>(Zigzag + RLE + Huffman)]
D --> E[JEPG 比特流]
```

#### **(1) 离散余弦变换 (DCT)**

- 将图像分块（通常8×8），对每块做 DCT：
  $$
  F(u,v) = C(u)C(v) \sum_{x=0}^{7} \sum_{y=0}^{7} f(x,y) \cos\left[\frac{(2x+1)u\pi}{16}\right] \cos\left[\frac{(2y+1)v\pi}{16}\right]
  $$
- **作用**：能量集中。低频分量（左上角）包含主要信息，高频分量（右下角）多为细节/噪声。

#### **(2) 量化 (Quantization)**

- 使用量化表 $ Q(u,v) $ 对 DCT 系数进行除法并取整：
  $$
  F_Q(u,v) = \text{round}\left( \frac{F(u,v)}{Q(u,v)} \right)
  $$
- **非均匀量化策略**：
  - 左上角（低频）：$ Q $ 值小 → 量化误差小 → 保留更多细节
  - 右下角（高频）：$ Q $ 值大 → 量化误差大 → 允许更大压缩

> 💡 **教授点拨 · 设计哲学**  
> “人眼对高频变化不敏感，所以我们可以大胆舍弃。这就是**感知编码**的思想——根据人类感官特性来丢数据。”

![Huffman编码应用与量化策略。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_6.png)

#### **(3) 熵编码优化**

- **Zigzag 扫描**：将8×8矩阵转为一维序列，使低频在前，高频在后。
- **游程编码 (Run-Length Encoding, RLE)**：
  - 格式：`(R, S, C)`
    - `R`：当前非零值前连续零的个数（Run）
    - `S`：非零值所需比特数（Size）
    - `C`：非零值本身（Coefficient）
  - 结束标志：`(0,0)` 表示后续全为零。
- **霍夫曼编码**：对 `(R,S)` 组合进行变长编码，高频组合用短码。

> 🎯 **优势总结**  
> 通过 DCT + 量化，大量系数变为0；再通过 RLE + Huffman，实现了远超单符号编码的压缩比。

![Huffman编码应用示例。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_7.png)

![Huffman编码应用示例。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_8.png)

---

### **3. 自适应编码：Lempel-Ziv (LZ) 算法**

> ✅ **术语更正**  
> 语音稿中“LC编码”、“LV编码”应为 **Lempel-Ziv 编码**，常见变体如 LZ77、LZW。

![Lempel-Ziv数据压缩特点](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_9.png)

#### **核心思想**

- **无需先验统计**：不需要预先知道信源概率分布。
- **基于字典**：利用数据中的重复模式动态构建“词典”。
- **广泛应用**：GIF、ZIP、PNG 等格式均采用 LZ 系列算法。

#### **LZ77 算法简述（滑动窗口）**

1. 维护一个长度为 $ W $ 的滑动窗口（历史缓冲区）。
2. 对当前位置的字符串，查找其在窗口内的最长匹配子串。
3. 编码为 `(偏移量 u, 长度 n)`：
   - `u`：匹配串起始位置距当前位置的距离
   - `n`：匹配串长度
4. 若无匹配（`n=1`），直接输出原字符。

#### **编码细节**
- `n` 的编码：使用 **Unary-Binary Code**
  - 前缀：`floor(log₂n)` 个零
  - 后缀：`n` 的二进制表示
- `u` 的编码：固定长度编码，需 $ \lceil \log_2 W \rceil $ 比特

![Lempel-Ziv数据压缩算法](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_10.png)

> 💡 **教授洞见 · 通用性的力量**  
> “LZ 编码的伟大之处在于它的**普适性**。不管你是中文、英文还是DNA序列，只要有重复，它就能压缩。它不像霍夫曼那样依赖训练，是一种真正‘即插即用’的智能压缩。”

![Lempel-Ziv编码示例](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_11.png)

---

## **第三部分：从离散到连续——信号采样理论**

> 🔁 **课程转折点**  
> “前面我们假设信源是离散的。但现实世界是模拟的。现在我们要回答：**如何把连续的模拟信号变成我们可以编码的离散序列？**”

![信号采样与处理流程图](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_16.png)

### **1. 采样的本质：正交展开**

采样不仅是“每隔一段时间取一个点”，更是**在信号空间中的一组正交基下的投影**。

#### **时间有限信号 → 傅里叶级数 (Fourier Series)**

- 信号 $ u(t) $ 在区间 $[-T/2, T/2]$ 外为零。
- 可展开为：
  $$
  u(t) = \sum_{k=-\infty}^{\infty} u_k \cdot e^{j2\pi kt / T} \cdot \Pi_T(t)
  $$
  其中 $ \Pi_T(t) $ 是宽度为 $ T $ 的矩形窗函数。
- 基函数 $ \phi_k(t) = e^{j2\pi kt / T} \cdot \Pi_T(t) $ 构成一组**正交基**（非单位正交，能量为 $ T $）。
- 系数 $ u_k $ 即为采样点，可通过内积提取：
  $$
  u_k = \frac{1}{T} \int_{-T/2}^{T/2} u(t) e^{-j2\pi kt / T} dt
  $$

> 🧩 **统一视角**  
> 无论是傅里叶级数、DTFT 还是采样定理，本质都是**信号在正交基下的分解**。

![傅里叶级数与信号能量](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_17.png)

![傅里叶级数展开与收敛性分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_18.png)

#### **时间无限信号 → 分段展开 (T-Spaced Truncated Sinusoidal Expansion)**

- 将无限信号分段，每段长度 $ T $。
- 每段独立做傅里叶级数展开。
- 总信号表示为双索引展开：
  $$
  u(t) = \sum_{m=-\infty}^{\infty} \sum_{k=-\infty}^{\infty} u_{k,m} \cdot \phi_k(t - mT)
  $$
- $ u_{k,m} $ 为第 $ m $ 段的第 $ k $ 个傅里叶系数。

![信号时域有限，频域正交展开。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_19.png)

---

### **2. 带宽有限信号 → 采样定理 (Sampling Theorem)**

> ✅ **公式更正**  
> PPT 中误写为 $ 2W $，正确采样周期应为 $ T_s = \frac{1}{2W} $，即**奈奎斯特率**。

#### **定理陈述**

若信号 $ x(t) $ 的频谱 $ X(f) $ 在 $ |f| > W $ 时为零（即带宽为 $ W $ Hz），则：

- 以采样间隔 $ T_s = \frac{1}{2W} $ 采样。
- 原信号可由采样点完美重构：
  $$
  x(t) = \sum_{k=-\infty}^{\infty} x\left(\frac{k}{2W}\right) \cdot \text{sinc}(2Wt - k)
  $$
  其中 $ \text{sinc}(x) = \frac{\sin(\pi x)}{\pi x} $。

#### **证明思路（频域 ↔ 时域）**

1. 带限信号 → DTFT 展开其频谱：
   $$
   X(f) = \sum_{k} x_k \cdot e^{-j2\pi kf / (2W)} \cdot \Pi_{2W}(f)
   $$
2. 对 $ X(f) $ 做逆傅里叶变换，利用 $ \mathcal{F}^{-1}\{\Pi_{2W}(f)\} = 2W \cdot \text{sinc}(2Wt) $。
3. 得到时域表达式，并代入 $ t = \frac{k}{2W} $，利用 sinc 函数的正交性完成重构。

> ⚠️ **重要前提**  
> 采样定理成立的关键是信号**严格带限**。现实中信号总是无限带宽，因此需加抗混叠滤波器。

> 💡 **教授点拨 · 收敛性的本质**  
> “我们说重构信号等于原信号，不是指**逐点相等**，而是指**两者之差的能量为零**。这叫做 $ L^2 $ 收敛或均方收敛。在工程意义上，这就足够了。”

![信号采样定理讲解](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_20.png)

![采样定理证明。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_21.png)

![采样定理与信号重构](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_22.png)

---

## **课程总结与展望**

![香农第一定理总结](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_13.png)

### **知识体系总览**

| 编码类型 | 特点 | 极限 | 应用场景 |
|--------|------|------|----------|
| 定长码 | 忽略统计特性 | $ \log M $ | 简单协议 |
| 变长码 (Huffman) | 利用概率分布 | $ H(X) $ | JPEG、MP3 |
| 块编码 | 联合编码提升效率 | $ \to H(X) $ | 高效存储 |
| LZ 编码 | 自适应、无需先验 | 接近熵 | ZIP、GIF |

### **核心结论**

1. **信息熵是无损压缩的终极边界**。
2. **霍夫曼编码是最优的前缀码**，但需已知概率分布。
3. **LZ 编码是自适应的通用压缩器**，适合未知信源。
4. **采样是信号在正交基下的投影**，采样定理给出了完美重构的条件。

> 🌟 **教授寄语**  
> “学通信，不要只盯着公式。要问自己：**这个技术解决了什么问题？它在和谁斗争？** 比如霍夫曼在和冗余斗争，LZ在和未知斗争，采样在和连续斗争。理解了‘斗争’，你就理解了本质。”

![上海科技大学感谢提问。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1128b-1126-e-Huffman%20Code%20for%20Encoding%20a%20Block/page_14.png)

---

**下次课预告**：我们将深入探讨**量化理论**与**有损信源编码**，揭开 MP3、AAC 等音频压缩背后的奥秘。