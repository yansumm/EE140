# **通信原理：信源编码与最优变长码理论**  
**主讲人：廉黎祥 | 上海科技大学创管学院 | 第11周 2025-11-26**

---

## **课程介绍**
本节课系统讲解了**离散信源的无损编码理论**，重点围绕**前缀码（Prefix-Free Code）的设计原则、Kraft不等式的作用、信息熵作为编码极限的理论依据**，以及**霍夫曼编码（Huffman Coding）作为一种实现最优编码的具体算法**。课程内容涵盖严格的数学推导，并深入剖析编码设计背后的直觉与思想方法。

> **教授点拨**：  
> “信源编码的本质不是‘压缩数据’，而是**逼近信源的信息熵**。你无法突破这个下界——这不是技术问题，是自然规律。”

![Kraft不等式与前缀码。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_1.png)

---

## **核心概念回顾：从信源到编码**

### **1. 离散无记忆信源（Discrete Memoryless Source, DMS）**
我们研究的起点是一个理想化的信源模型：
- 每个符号 $ X $ 独立地从一个有限字母表 $ \mathcal{X} = \{x_1, x_2, ..., x_M\} $ 中随机选取。
- 符号出现的概率分布为 $ P(x_i) = p_i $，且满足 $ \sum_{i=1}^M p_i = 1 $。

> **深入理解**：  
> “无记忆”意味着当前输出不影响下一个输出。现实中信源往往有记忆性（如英文中“q”后几乎总是“u”），但DMS是分析的基础模型，后续可通过块编码等方式处理相关性。

---

### **2. 编码的基本要素**
- **码字（Codeword）**：每个符号 $ x_i $ 被映射为一个由 `0` 和 `1` 构成的二进制序列，记作 $ c_i $。
- **码长（Length）**：码字 $ c_i $ 的比特数记为 $ l_i $。
- **平均码长（Average Length）**：
  $$
  \overline{L} = \mathbb{E}[L] = \sum_{i=1}^M p_i l_i \quad \text{(单位：bit/symbol)}
  $$
  这是衡量编码效率的核心指标，也称为**编码速率（Coding Rate）**，记作 $ R $。

> **教授点拨**：  
> “我们的目标很明确：在保证能唯一解码的前提下，让 $ \overline{L} $ 尽可能小。”

![最小化平均码长的前缀码设计。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_2.png)

---

## **编码方式的分类与发展脉络**

### **3. 定长编码（Fixed-Length Code）**
最简单的编码方式是将所有符号编码为相同长度的码字。

- 对于大小为 $ M $ 的信源，最短定长需满足：
  $$
  L = \lceil \log_2 M \rceil
  $$
- 当 $ M = 2^n $ 时，可达到精确值 $ L = \log_2 M $。

#### **更一般的定长编码：块编码（Block Fixed-Length Coding）**
将每 $ N $ 个符号视为一个元组进行联合编码。

- 共有 $ M^N $ 种可能的元组。
- 所需码长 $ L_N $ 满足 $ 2^{L_N} \geq M^N $，即 $ L_N \geq N \log_2 M $。
- 每个符号的平均码长为：
  $$
  \frac{L_N}{N} \geq \log_2 M
  $$

当 $ N \to \infty $ 时，这种分组定长编码的平均码长趋近于 $ \log_2 M $。

> **发展脉络**：  
> 从单符号定长 → 分组定长，体现了通过增加编码单元来逼近理论极限的思想。这为后面引入**变长编码**和**渐近等分性（AEP）** 埋下了伏笔。

---

### **4. 变长编码（Variable-Length Code）**
考虑到不同符号出现频率不同，高频符号应使用短码，低频符号用长码，以降低平均码长。

#### **编码的可译性分类**
| 类型 | 特点 |
|------|------|
| **奇异码（Singular Code）** | 至少有两个不同符号映射到同一码字，不可译。❌ |
| **非奇异码（Non-Singular Code）** | 所有符号对应不同码字，单个符号可区分。✅ |
| &emsp;└─ **唯一可译码（Uniquely Decodable, UD Code）** | 任意有限长码字序列均可唯一分割还原。✅ |
| &emsp;&emsp;&emsp;└─ **即时码 / 前缀码（Instantaneous / Prefix-Free Code）** | 任一码字都不是其他码字的前缀，可边接收边解码。✅ |

> **关键区分**：  
> - **唯一可译 ≠ 即时可译**：例如 `{0, 01}` 是UD码（因`0`必为结束），但不是前缀码。
> - **所有前缀码都是唯一可译码**，反之不成立。

> **趣闻实例**：  
> 教授举例：“想象摩尔斯电码，如果‘E’是`.`，‘I’是`..`，那收到两个点你就得等第三个信号才能判断是不是‘I’。但如果规定没有码字是另一个的开头，比如‘E’=`.`，‘I’=`-`，那你一听到点就知道是‘E’，这就是即时码的优势。”

---

## **核心理论：Kraft不等式与熵界限**

### **5. Kraft不等式：前缀码存在的充要条件**

对于一组码长 $ \{l_1, l_2, ..., l_M\} $，存在对应的前缀码当且仅当：
$$
\sum_{i=1}^M 2^{-l_i} \leq 1 \quad \text{(Kraft Inequality)}
$$

- 若取 **等号**，则该前缀码是**完备的（Complete）**，即其码树是**满二叉树**，没有任何码字可以缩短或新增。
- 若取 **严格小于**，则码树不满，仍有扩展空间。

> **思路与方法**：  
> Kraft不等式提供了**构造性判据**：给定一组理想的码长（如 $ l_i = \lceil -\log_2 p_i \rceil $），先检查是否满足Kraft不等式，再决定能否实现为前缀码。

> **教授点拨**：  
> “Kraft不等式像是一把尺子，它不告诉你怎么造最好的码，但它告诉你某个设计方案在结构上是否可行。”

---

### **6. 信源熵：编码的理论极限**

信源 $ X $ 的信息熵定义为：
$$
H(X) = -\sum_{i=1}^M p_i \log_2 p_i \quad \text{(单位：bit/symbol)}
$$

熵反映了信源的不确定性，也是无损编码的**绝对下限**。

![熵与前缀码长度关系。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_3.png)

---

### **7. 主要定理：最优前缀码的性能界限**

> **定理（Optimal Prefix Code Bound）**：  
> 对于任意离散信源 $ X $，其所有前缀码中可达到的最小平均码长 $ \overline{L}^* $ 满足：
> $$
> H(X) \leq \overline{L}^* < H(X) + 1
> $$

#### **左侧：$ \overline{L}^* \geq H(X) $ —— 熵是下界**

**证明思路**：对任意前缀码，考虑 $ H(X) - \overline{L} $ 并利用不等式 $ \log_2 x \leq x - 1 $ 放松。

**推导过程**：
$$
\begin{align*}
H(X) - \overline{L} &= \sum p_i \log_2 \frac{1}{p_i} - \sum p_i l_i \\
&= \sum p_i \left( \log_2 \frac{1}{p_i} - l_i \right) \\
&= \sum p_i \left( \log_2 \frac{2^{-l_i}}{p_i} \right) \\
&\leq \sum p_i \left( \frac{2^{-l_i}}{p_i} - 1 \right) \quad \text{(利用 } \log_2 x \leq x - 1\text{)} \\
&= \sum 2^{-l_i} - \sum p_i \\
&= \sum 2^{-l_i} - 1 \leq 0 \quad \text{(由Kraft不等式)}
\end{align*}
$$
因此 $ H(X) - \overline{L} \leq 0 \Rightarrow \overline{L} \geq H(X) $。

> **深刻洞见**：  
> “这意味着**任何无损编码的平均长度都不可能低于熵**。如果你发现某个编码的 $ \overline{L} < H(X) $，那它一定是**有损的**，或者根本不是唯一可译的！”

![信息熵与编码效率关系。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_4.png)

#### **何时取等？**
当且仅当对所有 $ i $，有 $ p_i = 2^{-l_i} $，即每个符号的概率恰好是 $ 2 $ 的整数次幂时，存在前缀码使得 $ \overline{L}^* = H(X) $。

> SELF：这就是一种最佳前缀码，即概率越低的，码长越高

> **实例说明**：  
> 设信源：$ a_1 (p=1/2), a_2 (p=1/4), a_3 (p=1/4) $。  
> 则 $ H(X) = 1.5 $ bit。若编码为：$ a_1 \to 0 $（1位），$ a_2 \to 10 $，$ a_3 \to 11 $（各2位），  
> 平均码长 $ \overline{L} = 0.5 \times 1 + 0.25 \times 2 + 0.25 \times 2 = 1.5 $，等于熵。

![熵与前缀码长度关系。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_5.png)

---

#### **右侧：$ \overline{L}^* < H(X) + 1 $ —— 上界可达**

**证明思路**：构造一种具体编码方案，使其平均码长小于 $ H(X) + 1 $。

**构造方法**：令 $ l_i = \lceil -\log_2 p_i \rceil $

**验证**：
1. **满足Kraft不等式**：
   $$
   \sum 2^{-l_i} \leq \sum 2^{\log_2 p_i} = \sum p_i = 1
   $$
   因此存在对应前缀码。
2. **平均码长上界**：
   $$
   l_i < -\log_2 p_i + 1 \Rightarrow \overline{L} = \sum p_i l_i < \sum p_i (-\log_2 p_i + 1) = H(X) + 1
   $$

> **经验与建议**：  
> “虽然 $ l_i = \lceil -\log_2 p_i \rceil $ 不一定是最优的，但它保证了性能不会太差——最多只比熵多不到1比特。这是一个非常稳健的设计起点。”

![熵与前缀码长度关系。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_6.png)

---

## **最优编码实践：霍夫曼编码（Huffman Coding）**

### **8. 霍夫曼编码：一种实现最优前缀码的算法**

霍夫曼编码是一种贪心算法，用于构造给定概率分布下的**最优前缀码**。

![Huffman编码原理与性质](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_7.png)

#### **算法步骤**
1. 将所有符号按概率从小到大排序。
2. 取出概率最小的两个符号，合并为一个新“超符号”，其概率为二者之和。
3. 将新符号放回集合，重新排序。
4. 重复步骤2-3，直到只剩两个符号，最后将它们分别编码为 `0` 和 `1`。
5. 自底向上展开，每次分裂时分别添加 `0` 和 `1`。

> **示例**：  
> 信源：$ A(0.4), B(0.2), C(0.15), D(0.15), E(0.1) $
>
> **编码过程**：
> 1. 合并 $ D,E $ → $ DE(0.25) $
> 2. 合并 $ B,C $ → $ BC(0.35) $
> 3. 合并 $ DE,A $ → $ DEA(0.65) $
> 4. 合并 $ BC,DEA $ → 根节点
>
> **最终码字**（从根读向叶）：
> - $ A: 1 $
> - $ B: 011 $
> - $ C: 010 $
> - $ D: 001 $
> - $ E: 000 $
>
> **平均码长**：
> $$
> \overline{L} = 0.4 \times 1 + (0.2+0.15) \times 3 + (0.15+0.1) \times 3 = 2.2 \text{ bit}
> $$
>
> **信息熵**：
> $$
> H(X) \approx 2.1464 \text{ bit}
> $$
>
> **编码效率**：
> $$
> \eta = \frac{H(X)}{\overline{L}} \approx \frac{2.1464}{2.2} \approx 97.56\%
> $$

> **教授点拨**：  
> “霍夫曼编码的妙处在于它的**递归最优性**：每次合并最小两个，相当于假设对剩余部分已是最优编码，然后在此基础上构建整体最优。”

![Huffman编码示例与效率计算。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_8.png)

![Huffman编码示例与效率分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_9.png)

---

### **9. 最优编码的性质与常见误区**

#### **最优前缀码的三个关键性质**
1. **概率越大，码越短**：若 $ p_i > p_j $，则 $ l_i \leq l_j $。
2. **码树是满的（Full Tree）**：每个中间节点都有两个子节点，无浪费。
3. **最小概率符号成对出现**：存在一种最优码，使得两个概率最小的符号具有相同的码长，且码字仅最后一位不同（称为“兄弟节点”）。

> **深入理解**：  
> 性质3是霍夫曼算法的直接体现。若两个最小概率符号未被合并，总可以通过交换使它们成为兄弟而不增加平均长度。

![Huffman编码最优性证明。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_10.png)

---

#### **重要澄清：关于“最优”的误解**

| 误解 | 正确理解 |
|------|----------|
| **最优码是唯一的** ❌ | 最优码不唯一！只要平均码长相同，多种编码方案都可能是最优的。 |
| **所有最优码都是霍夫曼码** ❌ | 霍夫曼码只是其中一类。例如，对概率为 $ \{0.3, 0.3, 0.2, 0.2\} $ 的信源，定长码 `{00,01,10,11}` 也是最优的（$ \overline{L}=2 $），但它不满足霍夫曼的“最小两合并”规则。 |
| **所有最优码都是前缀码** ❌ | 存在非前缀的最优唯一可译码。例如，对 $ \{1/2, 1/4, 1/8, 1/8\} $，编码 `{0, 10, 110, 111}`（霍夫曼）和 `{0, 11, 100, 101}` 都是最优的，后者虽非前缀但仍是唯一可译。 |

> **思想升华**：  
> “霍夫曼编码提供了一种**构造最优解的方法**，但它并未穷尽所有最优解。理解这一点，有助于我们看清算法的本质——它是工具，而非真理本身。”

![Huffman编码最优性分析。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_11.png)

![非前缀码与Huffman编码关系。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_12.png)

---

## **推广：块变长编码（Block Variable-Length Coding）**

为了进一步逼近熵，我们可以将 $ N $ 个符号作为一个块进行联合变长编码。

- 记 $ X^N $ 表示 $ N $ 个符号组成的序列。
- 对 $ X^N $ 应用最优前缀码，得到最小平均码长 $ \overline{L}_N^* $。
- 每个原始符号的平均码长为：
  $$
  \frac{\overline{L}_N^*}{N}
  $$

根据前述定理，有：
$$
H(X^N) \leq \overline{L}_N^* < H(X^N) + 1
\Rightarrow
H(X) \leq \frac{\overline{L}_N^*}{N} < H(X) + \frac{1}{N}
$$

> 这里最重要的是：刚刚在单符号变长编码中，我们证明了 $ H(X) \leq \overline{L}^* < H(X) + 1 $。
> 而在块变长编码中，+1，变成了 $+\frac{1}{N}$, 这使得在N足够大时，最佳编码的长度是一个定值（$ H(X) $），而不是一个区间。

当 $ N \to \infty $ 时，$ \frac{\overline{L}_N^*}{N} \to H(X) $。

> **发展脉络**：  
> 从单符号变长编码 → 块变长编码，这是通向**香农第一定理（信源编码定理）** 的关键一步。它表明，通过增大编码块长度，我们可以无限逼近信息熵这一终极极限。

> **课后思考**：  
> $ \frac{\overline{L}_N^*}{N} $ 与 $ \frac{\overline{L}_{N-1}^*}{N-1} $ 之间有何关系？是否单调递减？

![Huffman编码原理与应用](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_13.png)

![Huffman编码与信息熵关系](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1126b-1121-e-Minimum%20L%20for%20Prefix-free%20Codes/page_14.png)

---

## **总结与洞见**

| 概念 | 关键结论 | 启发 |
|------|----------|------|
| **信源熵 $ H(X) $** | 无损编码的绝对下界 | 任何压缩方案都无法突破此极限 |
| **Kraft不等式** | 前缀码存在的充要条件 | 是连接码长与可译性的桥梁 |
| **最优前缀码** | $ H(X) \leq \overline{L}^* < H(X) + 1 $ | 给出了实用编码的性能范围 |
| **霍夫曼编码** | 实现 $ \overline{L}^* $ 的一种有效算法 | 贪心策略在信息论中的成功应用 |

> **教授结语**：  
> “今天我们学的是如何‘省着用比特’。但更重要的是，我们看到了**信息本身是有‘重量’的**，这个重量就是熵。编码的艺术，就是在尊重这个重量的前提下，找到最精巧的搬运方式。”

> **学习建议**：  
> 1. 动手实践：对任意给定概率分布，亲手画出霍夫曼树。  
> 2. 反例思考：尝试构造一个非霍夫曼的最优码。  
> 3. 概念辨析：反复区分“唯一可译”、“前缀码”、“最优码”三者的关系。

--- 
**（完）**