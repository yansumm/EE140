# **通信原理深度学习笔记：从PAM到QAM的调制与系统设计**

**课程信息**  
- **课程名称**：通信原理  
- **授课教师**：廉黎祥  
- **时间**：2025年12月19日（第14周）  
- **整理目标**：基于课堂语音转录稿与PPT内容，融合知识线与思想线，构建一份结构清晰、洞见丰富、便于复习的深度学习笔记。

---

## **一、课程回顾：调制与解调系统流程**

我们首先回顾了数字通信系统的整体架构，重点聚焦于**信道调制与解调过程**。整个系统的核心任务是将原始的二进制比特流高效且可靠地传输到接收端。

### **1. 系统基本流程**
1. **输入**：二进制比特序列（如 `0101...`）。
2. **映射（Mapping）**：将每 $ b $ 个比特组合成一个符号（symbol），共 $ M = 2^b $ 种可能。
3. **星座图构建**：为每个符号分配一个具体的信号点，构成“星座图”。
4. **基带波形生成**：将离散符号序列 $ \{u_k\} $ 转换为连续时间基带信号 $ u(t) $。
5. **带通调制**：通过载波调制，将基带信号搬移到高频，形成可在信道中传输的实信号 $ s(t) $。
6. **信道传输**：信号在有噪声或衰落的信道中传播。
7. **解调与恢复**：接收端对接收信号进行相干解调，恢复出基带信号 $ u(t) $。
8. **采样与判决**：对基带信号采样并判决，还原符号序列 $ \{u_k\} $，再逆映射回原始比特流。

> **教授点拨·核心思路**  
> 整个调制过程可视为两个关键步骤：
> - **离散到连续**：利用脉冲成型滤波器将离散符号序列平滑为模拟波形；
> - **低频到高频**：借助正弦载波实现频谱搬移，以适应无线信道的频率特性。
>
> 解调则是其逆过程，但难点在于如何在加性高斯白噪声（AWGN）干扰下准确恢复原始符号——这引出了“误码率”和“抗干扰能力”的设计权衡问题。

![PAM信号处理流程图](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_14.png)

---

## **二、PAM：脉冲幅度调制的基础模型**

PAM（Pulse Amplitude Modulation）是最基础的数字调制方式之一，它通过对基础波形 $ p(t) $ 的幅度进行控制来携带信息。

### **1. 核心概念与参数定义**
| 参数 | 符号 | 单位 | 物理意义 |
|------|------|------|----------|
| 比特率 | $ R_b $ | bps | 每秒传输的比特数 |
| 每符号比特数 | $ b $ | bit/symbol | 每个符号代表的比特数量 |
| 符号率（波特率） | $ R_s $ | Baud 或 symbol/s | 每秒传输的符号数，$ R_s = R_b / b $ |
| 符号周期 | $ T $ | s | 相邻符号的时间间隔，$ T = 1/R_s $ |
| 基础波形能量 | $ E_p $ | J | $ E_p = \int |p(t)|^2 dt $ |

### **2. 星座图与信号表示**
- **星座图**：由 $ M $ 个实数点组成的一维点集，通常对称分布在原点两侧。
- **标准构造**：假设最小间距为 $ d $，则星座点可表示为：
  $$
  a_m = \left( m - \frac{M+1}{2} \right)d, \quad m=1,2,\dots,M
  $$
  例如，4-PAM 的星座点为 $ \{-\frac{3d}{2}, -\frac{d}{2}, \frac{d}{2}, \frac{3d}{2}\} $。

- **发送信号表达式**：
  $$
  u(t) = \sum_k u_k p(t - kT)
  $$
  其中 $ u_k \in \mathcal{A} $ 是第 $ k $ 个时刻的符号值，$ p(t) $ 是归一化或任意形状的基础波形。

![PAM信号星座图分析](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_1.png)

### **3. 平均符号能量计算**
平均能量 $ E_s $ 反映了系统的功耗水平，是衡量效率的重要指标。

$$
E_s = \mathbb{E}[|u_k|^2] \cdot E_p
$$

当所有符号等概率出现时，期望 $ \mathbb{E}[|u_k|^2] $ 即为各星座点到原点距离平方的均值。

#### **推导结果（等概分布）**
对于标准对称 PAM 星座，有：
$$
E_s = \frac{d^2 (M^2 - 1)}{12} \cdot E_p
$$

> ✅ **校正说明**：原始语音稿中提到“十二分之 D平方 M平方减一”，经核实公式正确，应为 $ \frac{d^2(M^2 - 1)}{12} E_p $。

> **教授点拨·直观理解**  
> - 能量本质是“星座点离原点的分散程度”。越集中，能量越小。
> - 对称设计是为了最小化平均能量：若整体右移，则所有点离原点更远，平均能量上升。
> - 这类似于信源编码中的霍夫曼编码思想：高频符号用短码（低能量），低频符号用长码（高能量）以降低总体开销。

![PAM信号星座图分析](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_2.png)

---

## **三、性能分析：可靠性与效率的权衡（Trade-off）**

通信系统的设计本质上是在多个性能指标之间寻求平衡。以下是三大核心维度：

### **1. 能量效率（Energy Efficiency, E.E.）**
定义为单位能量下能传输的比特数：
$$
\text{E.E.} = \frac{R_b}{E_b} = \frac{R_b}{E_s / b} = \frac{b R_b}{E_s}
$$
其中 $ E_b = E_s / b $ 是每比特能量。

代入 $ E_s $ 表达式后可见：
- $ M $ 越大 → 分母增长快（$ M^2 $）→ 能量效率下降
- $ d $ 越小 → 分子减小 → 能量效率提升

👉 因此，**提高能量效率的方法是使用小 $ M $ 和小 $ d $**。

### **2. 带宽效率（Bandwidth Efficiency, B.E.）**
定义为单位带宽内能传输的比特速率：
$$
\text{B.E.} = \frac{R_b}{B}
$$
基带信号带宽主要由基础波形决定。若 $ p(t) $ 的持续时间为 $ T $，则其带宽约为 $ 1/T = R_s $。

由于 $ R_s = R_b / \log_2 M $，故：
$$
\text{B.E.} \propto \log_2 M
$$

👉 所以，**增大 $ M $ 可显著提高带宽效率**。

### **3. 可靠性：误码率（Pe）与最小距离 $ d $**
- 接收信号受噪声影响会发生偏移。
- 若偏移导致落入邻近星座点的判决区域，则发生误判。
- 最小距离 $ d $ 越大，抗噪能力越强，误码率 $ P_e $ 越低。

> **教授点拨·设计哲学**  
> 我们面临三个相互冲突的目标：
> 1. **高能量效率** ← 要求小 $ d $
> 2. **高带宽效率** ← 要求大 $ M $
> 3. **高可靠性（低 $ P_e $）** ← 要求大 $ d $

> 实际设计必须根据应用场景做出取舍：
> - 卫星通信：能量受限 → 优先考虑能量效率
> - 光纤通信：带宽充裕 → 优先考虑带宽效率
> - 关键指令传输：可靠性第一 → 加大 $ d $ 或降低 $ M $

![通信系统性能分析](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_3.png)

---

## **四、QAM：正交幅度调制的突破**

QAM（Quadrature Amplitude Modulation）是对 PAM 的自然扩展，它将调制空间从一维提升到二维复平面，从而同时利用**幅度**和**相位**两个自由度。

### **1. QAM 的本质与优势**
- **星座点为复数**：$ u_k = u_{k,I} + j u_{k,Q} $
- **基带信号为复信号**：
  $$
  u(t) = \sum_k u_k p(t - kT)
  $$
- **问题**：复信号无法直接在物理信道中传输（需为实信号）。

#### **解决方案：正交载波双边带调制（DSB-QC）**
我们将复基带信号 $ u(t) $ 映射为实带通信号 $ s(t) $：
$$
s(t) = 2 \cdot \text{Re}\left[ u(t) e^{j2\pi f_c t} \right] = 2 u_I(t) \cos(2\pi f_c t) - 2 u_Q(t) \sin(2\pi f_c t)
$$

> 🔍 **推导详解**  
> 设 $ u(t) = u_I(t) + j u_Q(t) $，$ e^{j2\pi f_c t} = \cos(2\pi f_c t) + j\sin(2\pi f_c t) $，则乘积为：
> $$
> u(t)e^{j2\pi f_c t} = (u_I \cos - u_Q \sin) + j(u_I \sin + u_Q \cos)
> $$
> 取实部得 $ u_I \cos - u_Q \sin $，乘以2即得最终表达式。

> **教授点拨·工程洞察**  
> - 该方法称为“正交调制”，因为 $ \cos $ 和 $ \sin $ 是正交函数，在积分意义上互不干扰。
> - I路（同相）用余弦调制，Q路（正交）用负正弦调制，二者可在接收端完全分离。
> - 接收端只需分别用 $ \cos $ 和 $ \sin $ 相干解调即可独立提取两路信号。

![QAM基带到带通转换流程图](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_12.png)

![QAM基带转频带转换](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_10.png)

![QAM基带转带通信原理。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_11.png)

---

## **五、QAM 星座图设计：从笛卡尔积到几何优化**

### **1. 标准 QAM 构造：笛卡尔积法**
标准 $ M $-QAM 星座可通过两个 $ \sqrt{M} $-PAM 星座的笛卡尔积生成：

设 $ \mathcal{A}_I $ 和 $ \mathcal{A}_Q $ 分别为 I 路和 Q 路的 PAM 星座，则：
$$
\mathcal{A}_{\text{QAM}} = \{ a_I + j a_Q \mid a_I \in \mathcal{A}_I, a_Q \in \mathcal{A}_Q \}
$$

> 📌 示例：16-QAM = 4-PAM × 4-PAM  
> - 4-PAM 点集：$ \{\pm d/2, \pm 3d/2\} $  
> - 16-QAM 形成 $ 4\times4 $ 方格阵列

### **2. 平均能量分析**
对于标准 $ \sqrt{M} \times \sqrt{M} $ QAM：
$$
E_s = \mathbb{E}[|u_k|^2] E_p = \left( \mathbb{E}[u_I^2] + \mathbb{E}[u_Q^2] \right) E_p = 2 \cdot \frac{d^2 (M - 1)}{12} E_p = \frac{d^2 (M - 1)}{6} E_p
$$

> ✅ **对比总结**
> | 调制方式 | 平均能量 $ E_s $ |
> |---------|------------------|
> | PAM     | $ \frac{d^2 (M^2 - 1)}{12} E_p $ |
> | QAM     | $ \frac{d^2 (M - 1)}{6} E_p $ |

> 当 $ M > 4 $ 时，QAM 在相同 $ d $ 下的能量消耗远低于 PAM！

![QAM星座图与信号速率](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_6.png)

![QAM星座图解析](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_7.png)

![QAM星座图与信号点排列](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_8.png)

### **3. 星座图的几何优化**
虽然方形 QAM 最常用，但它并非理论上最优。

#### **思考题**：给定 $ M $ 和最小距离 $ d $，何种形状星座图能量最小？

> **答案**：**圆形排列最理想！**
>
> - 圆形布局使所有点尽可能靠近原点，最大化能量效率。
> - 实际中因实现复杂而少用，但启发了现代高阶调制设计（如 APSK — 幅相移键控）。

#### **其他特殊星座设计**
- **PSK（Phase Shift Keying）**：所有点位于同一圆周上 → 恒包络，抗非线性好  
  - 优点：功率恒定，适合非线性放大器  
  - 缺点：当 $ M $ 大时点密集，抗噪差 → 仅用于小 $ M $（如 BPSK, QPSK）
- **星型星座**：相邻点间相位差更大 → 抗相位抖动能力强

> **教授点拨·设计启示**  
> 星座图设计没有绝对最优解，只有“最适合当前信道条件”的方案：
> - AWGN 信道：QAM 性能优越  
> - 存在相位噪声：优先选择 PSK 或星型星座  
> - 功率受限且需恒包络：采用 π/4-DQPSK 等改进型  

![QAM星座图与PSK对比](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_9.png)

---

## **六、综合性能比较：PAM vs QAM vs PSK**

| 维度         | PAM                  | QAM                              | PSK                     |
|--------------|----------------------|-----------------------------------|--------------------------|
| 调制自由度   | 幅度                 | 幅度 + 相位                      | 相位                     |
| 星座维度     | 1D                   | 2D                                | 2D（环形）               |
| 能量效率     | 中                   | **高**（优于 PAM 和 PSK）        | 低（大 $ M $ 时能量高） |
| 带宽效率     | 中                   | **高**                            | 高                       |
| 抗噪能力     | 中                   | **高**（相同能量下 $ P_e $ 更低）| 中（大 $ M $ 时差）      |
| 实现复杂度   | 低                   | 中                                | 中                       |
| 典型应用     | 基带传输             | WiFi, LTE, Cable Modem           | 卫星通信, RFID           |

> **结论**：QAM 凭借其在能量和带宽效率上的双重优势，成为现代高速通信系统的主流选择。

![PAM与QAM调制对比。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_5.png)

---

## **七、系统实现关键：无码间干扰（ISI-Free）设计**

无论哪种调制，最终都要解决同一个根本问题：**如何从连续波形 $ u(t) $ 中无失真地恢复出离散符号 $ \{u_k\} $？**

### **1. 问题建模**
- 发送信号：$ u(t) = \sum_k u_k p(t - kT) $
- 接收端先经过接收滤波器 $ q(t) $，得到：
  $$
  r(t) = u(t) * q(t) = \sum_k u_k g(t - kT), \quad \text{其中 } g(t) = p(t) * q(t)
  $$
- 再以周期 $ T $ 采样：$ r(kT) $

目标：**使 $ r(kT) = u_k $**，即当前符号不受其他符号干扰。

### **2. 奈奎斯特准则（Nyquist Criterion）**
要实现零码间干扰（Zero ISI），复合脉冲响应 $ g(t) $ 必须满足：
$$
g(nT) = 
\begin{cases}
1, & n = 0 \\
0, & n \neq 0
\end{cases}
$$

这样的 $ g(t) $ 称为“奈奎斯特脉冲”。

> 📈 **图形理解**：$ g(t) $ 在 $ t=0 $ 处为1，在 $ t=\pm T, \pm 2T, \dots $ 处恰好过零点。

![Nyquist准则与无ISI设计。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_15.png)

![Nyquist准则核心要点。](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_16.png)

### **3. 工程实践建议**
- 若忽略信道影响，只需设计 $ p(t) $ 和 $ q(t) $ 使得 $ g(t) = p*q $ 满足奈奎斯特准则。
- 常见选择：
  - **升余弦滚降滤波器**：在频域平滑过渡，避免陡峭截止带来的时域振荡。
  - **匹配滤波器设计**：令 $ q(t) = p(-t) $，最大化信噪比。

> **教授点拨·研究视角**  
> 当考虑真实信道 $ h(t) $ 时，问题变为：
> $$
> g(t) = p(t) * h(t) * q(t)
> $$
> 此时发送与接收端需联合设计（如采用均衡器），这也是现代通信算法的核心挑战之一。

![QAM实现与PAM关系](https://raw.githubusercontent.com/yansumm/picbed/master/class/EE140/1219b-1217-e-PAM%20Signal%20Constellation/page_13.png)

---

## **八、结语：通信系统设计的艺术**

本节课从 PAM 出发，逐步深入至 QAM 与系统级设计，揭示了通信工程的本质：

> **通信不是简单的“传数据”，而是在资源约束下追求最优折衷的艺术。**

### **三大核心思想总结**
1. **抽象之美**：将复杂通信链路分解为“比特 → 符号 → 波形 → 频谱”的逐层映射。
2. **几何直觉**：用星座图将抽象信号可视化，使能量、距离、效率等概念变得可感可知。
3. **权衡思维**：任何性能提升都有代价，优秀工程师懂得在 $ E_s $、$ B $、$ P_e $、$ M $ 之间找到最佳平衡点。

> “未来的通信不再只是更快，而是更智能。”  
> —— 随着机器学习介入调制识别、自适应星座设计等领域，通信系统正走向认知化与智能化的新阶段。

---

## **附录：课后作业与延伸学习建议**

### **本次作业**
1. 完成一道 PAM 调制相关习题（计算能量、误码率等）
2. 完成一道 QAM 调制相关习题（构造星座、分析性能）
3. （选做）预习下一讲内容，思考如何设计满足奈奎斯特准则的 $ p(t) $ 和 $ q(t) $

### **推荐自学方向**
- **数学基础**：信号与系统、概率论、线性代数
- **进阶主题**：
  - 匹配滤波器理论
  - 升余弦滚降滤波器设计
  - 最大似然序列估计（MLSE）
- **前沿领域**：
  - **压缩感知**（Compressed Sensing）：稀疏信号下的高效采样与重构  
    > 注：该方向曾在2010年代火热，现已部分融入深度学习框架中。
  - **深度学习辅助通信**：神经网络用于信道估计、调制识别、端到端通信系统设计

> **教授寄语**  
> “不要只记住公式，要去理解它背后的物理图像和设计动机。当你能画出一张图就解释清楚一个问题时，说明你真的懂了。”

--- 

📌 **笔记更新日期**：2025年12月20日  
✅ **已校验术语准确性**：PAM, QAM, PSK, DSB, ISI, Nyquist 等  
✅ **已补充发展背景与设计洞见**  
✅ **结构清晰，主次分明，适用于高效复习**