
## Manual Neural Nets

### 从线性假设类到非线性假设类

#### 线性假设类问题：

- 对于线性分类问题，我们会构建一个假设函数将输入 $\mathbb{R}^{n}$ 映射到输出（类别：Logits）$\mathbb{R}^{k}$ ，该线性假设类：$$h_{\theta}(x)=\theta^{T}x$$其中 $\theta\in\mathbb{R}^{n\times k}$

- 这种分类器本质上构造了输入的 `k` 个线性函数，然后预测值最大的那个类别：相当于将输入空间划分为对应于每个类别的 `k` 个线性区域

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251216213729132.png" width="250px"/>
</div>



#### 非线性假设类问题：

- 但对于非线性问题，即数据无法被一组线性区域分割开该如何处理？我们希望能通过非线性的类别边界集合来分割这些点

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251216214801313.png" width="250px"/>
</div>

- 一个想法：对数据的某些特征（可能是将数据转化为更高维度）应用线性分类器：$$h_{\theta}(x)=\theta^{T}\phi(x)$$ 其中 $\theta\in\mathbb{R}^{d\times k}, \phi:\mathbb{R}^{n}\rightarrow\mathbb{R}^{d}$

- $\phi$ 表示维度转换， $\phi(x)$ 也称为特征

---
- **如何创建非线性特征函数 $\phi$**：

- 两个思路：
	1. 通过手动特征工程构建与问题相关的特征（机器学习的"旧"方法） 
	2. 通过从数据中学习的方式（机器学习的"新"方法）

- 非线性特征：
	- 本质上：线性特征的任何非线性函数，即该特征是线性特征 `x` 的一个待定的非线性函数 $$\phi(x)=\sigma(W^{T}x)$$ 其中 $W\in\mathbb{R}^{n\times d}$，且 $\sigma:\mathbb{R}^{d}\rightarrow\mathbb{R}^{d}$ 表示任何非线性函数

- 由此，非线性模型假设如上，但我们该如何得到这个更为准确的假设结果 $\phi(x)$，使得分类的效果更好（最小化损失）—— **神经网络**







### 神经网络/深度学习

- **神经网络**：一种特定类型的假设类，由多个参数化的可微函数（又称"层"（`layer`））以任意方式组合而成，从而形成输出

- "**深度网络**"只是“神经网络”的同义词（但也确实，现代神经网络涉及将大量函数组合在一起，因此"深度"通常是一个恰当的限定词），而**深度学习** —— 使用神经网络假设类进行机器学习

####  two layer neural network（两层神经网络）

- 我们可以从最简单的神经网络形式 —— 两层神经网络开始

- 两层神经网络 —— 该非线性特征 $\phi(x)$ 中含有两组权重 $W_i$ 是可学习的参数 $$h_{\theta}(x)=W_{2}^{T}\sigma(W_{1}^{T}x)$$$$\theta=\{W_{1}\in\mathbb{R}^{n\times d},W_{2}\in\mathbb{R}^{d\times k}\}$$ 其中 $\sigma:\mathbb{R}\rightarrow\mathbb{R}$ 是逐元素应用于向量的非线性函数（例如 `sigmoid`、`ReLU`）
- 通过链式法则（维度匹配）写成批量矩阵的形式：$$h_{\theta}(X)=\sigma(XW_{1})W_{2}$$
- 接下来我们对特征函数中的各项待定参数（包括 $\sigma$ 、$W_i$）进行讨论

1. **非线性函数 $\sigma$**：

- **核心思想**：
	- 对于 $h_{\theta}(x)=W_{2}^{T}\sigma(W_{1}^{T}x)$ 非线性分类器模型，我们希望**非线性函数 $\sigma$ 提供非线性特性**，由训练得到权重参数 $W_i$ 对函数进行调整拟合使得该模型的分类效果最好
	- 简单点说，即非线性函数 $\sigma$ 仅需要提供非线性特性，使得分类器模型具有"曲度"，可以去拟合/逼近任何函数 —— 最终结果得到一个损失最小的函数

- **通用函数逼近（Universal Function Approximation）**：
- 定理（1 维情况）：给定任何平滑函数 $f:\mathbb{R}\rightarrow\mathbb{R}$，封闭区域 $\mathcal{D}\subset\mathbb{R}$，以及 $\epsilon>0$，我们可以构建一个单隐藏层神经网络 $\hat{f}$，使得 $$max_{x\in\mathcal{D}}|f(x)-\hat{f}(x)|\le\epsilon$$
- 证明思路：
	- 在 $\mathcal{D}$ 上选择一些密集的点集 $(x^{(i)},f(x^{(i)}))$，创建一个恰好经过这些点的神经网络。因为神经网络是分段线性的，且函数 $f$ 是平滑的，通过选择足够接近的 $x^{(i)}$, 我们可以任意紧密地逼近该函数

- 该**单隐藏层神经网络 ReLU** （带偏置）：
$$
\hat{f}(x)=\sum_{i=1}^{d}\pm max\{0,w_{i}x+b_{i}\}
$$
<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251216224458264.png" width="200px"/>
</div>

- 函数拟合过程：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251216224903378.png" width="400px"/>
</div>

- 综上，对于**非线性函数 $\sigma$ 的选取可为 —— ReLU 函数**


---

2. **权重参数 $W_i$ 的训练**：

- **全连接深度网络（Fully-connected Deep Networks）**
	- 一个更通用的 L-Layer 神经网络形式  —— 也称为**多层感知机（MLP）**

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251216225453091.png" width="400px"/>
</div>
- 其中 $$Z_{i+1}=\sigma_{i}(Z_{i}W_{i}), \quad i=1,...,L$$$$Z_{1}=X, \quad h_{\theta}(X)=Z_{L+1}$$$$[Z_{i}\in\mathbb{R}^{m\times n_{i}}, W_{i}\in\mathbb{R}^{n_{i}\times n_{i+1}}]$$
- 其中非线性函数 $\sigma_{i}:\mathbb{R}\rightarrow\mathbb{R}$ 逐元素应用，参数 $\theta=\{W_{1},...,W_{L}\}$

- 为什么采用深度网络：
	- 深度网络在表示默写函数（例如奇偶校验）时被证明更为高效，而这些是（浅层）神经网络无法实际学习的
	- 经验上看，在固定参数数量的情况下，深度网络的效果更好

---

- **机器学习中的神经网络**

- 神经网络制定了"机器学习算法三要素"中的一个，还需要
	- 损失函数：交叉熵损失
	- 优化过程：随机梯度下降（SGD）

- 所以主要核心问题：**优化问题 $\rightarrow$ 计算梯度**

$$\nabla_{\theta}l_{ce}(h_{\theta}(x^{(i)}),y^{(i)})$$

---

- **两层神经网络的梯度（写成批量矩阵形式）：** $$\nabla_{\{W_{1},W_{2}\}}l_{ce}(\sigma(XW_{1})W_{2},y)$$
- **关于 $W_2$ 的梯度推导**，通过链式法则可以得到：$$
\frac{\partial l_{ce}(\sigma(XW_{1})W_{2},y)}{\partial W_{2}} = \frac{\partial l_{ce}(\sigma(XW_1)W_2, y)}{\partial \sigma(XW_1)W_2} \cdot \frac{\partial (XW_1)W_2}{\partial W_2}= (S-I_{y})\cdot(\sigma(XW_{1})) 
$$其中 $S = \text{normalize}(\exp(\sigma(XW_1)W_2))$

- 所以由维度匹配（matching sizes），我们可以得到 $W_2$ 的梯度为 $$\nabla_{W_{2}}l_{ce}(\sigma(XW_{1})W_{2},y)=\sigma(XW_{1})^{T}(S-I_{y})$$
- 同理，我们也可推导得到**关于 $W_1$ 的梯度推导**：$$
\frac{\partial l_{ce}(\sigma(XW_{1})W_{2},y)}{\partial W_{1}} = \frac{\partial l_{ce}(\sigma(XW_1)W_2, y)}{\partial \sigma(XW_1)W_2} \cdot \frac{\partial (XW_1)W_2}{\partial W_1} \cdot \frac{\partial \sigma(XW_1)}{\partial XW_1} \cdot \frac{\partial XW_1}{\partial W_1}
$$$$
 = (S-I_{y})\cdot (W_2) \cdot (\sigma(XW_{1})) \cdot (X)
$$$$\nabla_{W_{1}}l_{ce}(\sigma(XW_{1})W_{2},y)=X^{T}((S-I_{y})W_{2}^{T}\circ\sigma^{\prime}(XW_{1}))$$ 其中 $\circ$ 表示逐元素乘法

---
### 前向传播与反向传播

对于全连接网络（MLP 多层感知机）：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251217111959320.png" width="400px"/>
</div>

- **前向传播（Forward Pass）**:

- 定义：前向传播是将数据 $X$ 输入进去，经过一层层 $W_i$ 权重以及 $\sigma$ 非线性处理，最后输出预测结果的过程

- 迭代公式：$$
Z_{i+1} = \sigma_i(Z_i W_i)
$$
- 可理解拆分为两个步骤：
	1. 线性调整：加权混合上一层信号 $Z_i$ 并乘以权重部分 $W_i$ 
	2. 非线性激活：通过激活函数 $\sigma$ (如 ReLU) 赋予函数非线性特性
- 而对于迭代结果 $Z_i$：
	- 第 $i$ 层输出，也是第 $i+1$ 层输入，保存了数据经过处理后的特征

- 在前向传播结束时，我们可以得到预测值 $Z_n$，并计算出其与真实值 $y$ 进行比对，得出误差（loss）


---
- **反向传播（Backward Pass）**:

- 本质：因为最终结果有误差 loss，我们要找出是谁的责任，但由于层层外包，我们需要从后往前倒推责任

- 迭代公式：$$G_{i} = (G_{i+1} \circ \sigma'(Z_i W_i)) W_i^T$$ 其中 $G_i$：表示损失函数 $l$  相对于第 $i$ 层输出 $Z_i$ 的梯度

> [!NOTE] 公式推导
> 对损失函数进行梯度求解：
> - 对于全连接网络：$$Z_{i+1}=\sigma_{i}(Z_{i}W_{i}), \quad i=1,...,L$$
> - 而其对于各项参数的偏导：$$\frac{\partial l(Z_{L+1},y)}{\partial W_i}=\frac{\partial l}{\partial Z_{L+1}} \cdot \frac{\partial Z_{L+1}}{\partial Z_L} \cdot \frac{\partial Z_{L-1}}{\partial Z_{L_1}} \cdot ... \cdot \frac{\partial Z_{i+2}}{\partial Z_{i+1}} \cdot \frac{\partial Z_{i+1}}{\partial W_i}$$
> 
> ![image.png](https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251217174711884.png)
> 
> - 定义 $G_{i+1}=\frac{\partial l(Z_{L+1},y)}{\partial Z_{i+1}}$
> - 通过反向迭代，我们可以计算得到 $G_i$ 的推导：$$G_{i}=G_{i+1}\cdot\frac{\partial Z_{i+1}}{\partial Z_{i}}=G_{i+1}\cdot\frac{\partial \sigma_i(Z_i W_i)}{\partial Z_i W_i} \cdot \frac{\partial Z_i W_i}{\partial Z_i}=G_{i+1}\cdot\sigma^{\prime}(Z_{i}W_{i})\cdot W_{i}$$
> - 进行矩阵维度转换，可得到真实梯度：$$\nabla_{W_{i}}l(Z_{L+1},y)=Z_{i}^{T}(G_{i+1}\circ\sigma^{\prime}(Z_{i}W_{i}))$$
>   


- 反向传递流程（链式法则的具象化）：
	- 起点：先计算最后一层的误差 $G_{L+1}$，通常为预测值减去真实值
	- 传递过程：
		- 要根据下一层误差 $G_{i+1}$ 推算出当前层 $Z_i$ 的误差 $G_i$

---
- **更新权重（Update）**：

- 在前向传播得到输入与当前神经层的预测结果（$Z_i$）以及反向传播得到的当前层误差梯度（$G_i$）, 我们需要根据传播的输出结果来更新每一网络层的权重，调试权重 $W_i$ 使得模型的分类误差更小
- 核心公式：$$\nabla_{W_i} l = Z_i^T (G_{i+1} \circ \sigma'(Z_i W_i))$$
- 物理意义：
	- 要更新的是连接第 $i$ 层和第 $i+1$ 层的权重 $W_i$，通过链式法则，取决于三件事：
	1. $G_{i+1}$（后一层误差）：后一层觉得误差多严重，如果后面误差很大，通过 $W_i$ 传过去的信号肯定也有大问题，所以 $W_i$ 需要进行大幅度修改
	2. $\sigma'(...)$ (激活函数的导数)：通过“安检门”时，激活函数是否激活？（如果 ReLU 小于 0 时导数为 0，意味着这里并没有信号通过，那这里的权重则没有误差责任。不需要进行修改）
	3. $Z_i$ （前一层的输入信号）：前向传播过程中 $Z_i$ 对预测结果的影响
		- 例如：最后的误差很大，但如果第 $i$ 层的某个神经元输出 $Z_i$ 是 0（没有发射信号），那么它就不应该对这个误差负责
		- 反之，如果某个神经元 $Z_i$ 发射的信号很强，而最终导致了误差，那么它所连接的权重 $W_i$ 的误差责任就比较大，需要大幅度的修改
- 总结联系：
	- $Z_i$ 是信号强度
	- $G_{i+1}$ 是错误程度
	- 乘在一起，就是具体的权重修改方案$\nabla W_i$







## Automatic Differentiation ( 自动微分)

### 机器学习中的微分：

- 每一个机器学习算法中通常包含三个核心要素：

1. **假设空间（Hypothesis class）**：即模型 $h_{\theta}(x)$ （例如一个参数化的神经网络）
2. **损失函数（Loss Function）**：衡量预测好坏的标准，例如 $l(h_{\theta}(x), y)$
3. **优化方法（Optimization Function）** ：更新参数以最小化损失，例如梯度下降：$$
		\theta:=\theta-\frac{\alpha}{B}\sum_{i=1}^{B}\nabla_{\theta}l(h_{\theta}(x^{(i)}),y^{(i)})$$
- 机器学习主要核心：
	- **计算损失函数 $h_{\theta}(x)$ 关于假设空间参数 $\theta$ 的梯度 $G$（gradient）**，即微分操作


### 常见的微分方法：

1. **数值微分（Numberical Differentiation）**：

- 即定义法微分：直接根据导数的定义计算偏导数
- 基本公式：
$$
\frac{\partial f(\theta)}{\partial\theta_{i}}=lim_{\epsilon\rightarrow0}\frac{f(\theta+\epsilon e_{i})-f(\theta)}{\epsilon}
$$
- 更精确的近似（中心差分法）：
$$
\frac{\partial f(\theta)}{\partial\theta_{i}}=\frac{f(\theta+\epsilon e_{i})-f(\theta-\epsilon e_{i})}{2\epsilon}+o(\epsilon^{2})
$$
- 缺点：
	- 这种方法由于根据定义来求取微分，根据公式可以看出，每一次求取微分导数时，都需要取一个极小的 $\epsilon$ 来求取微分值，同时在更为精确的近似（**中心差分法**）中，我们可以看到每一次微分结果中都会存在有一个极小的 $o(\epsilon^{2})$ 
	- 所以由此可以看出该方法存在下述缺点：
		- **受数值误差影响较大**
		- **计算效率较低**（需要多次前向传播的）
- 用途：
	- 但该方法在实际工程中，可以还是有所用途 - 数值梯度检测（Numberical Gradient Checking）
	- 即在单元测试中，用于验证自动微分算法实现的正确性
		- 方法：从单位球中随机选取方向 $\delta$，验证 $\delta^{T}\nabla_{\theta}f(\theta) \approx \frac{f(\theta+\epsilon\delta)-f(\theta-\epsilon\delta)}{2\epsilon}$ 是否成立

---
2. **符号微分（Symbolic Differentiation）**

- 定义：写出函数解析公式，利用**微分加法、乘法以及链式法则**推导梯度
1. 加法：
$$
\frac{\partial(f(\theta) + g(\theta))}{\partial\theta}=\frac{\partial f(\theta)}{\partial\theta}+\frac{\partial g(\theta)}{\partial\theta}
$$
2. 乘法：
$$
\frac{\partial f(\theta)g(\theta)}{\partial\theta}=g(\theta)\frac{\partial f(\theta)}{\partial\theta}+f(\theta)\frac{\partial g(\theta)}{\partial\theta}
$$
3. 链式法则：
$$
\frac{\partial(f(g(\theta))}{\partial\theta}=\frac{\partial f(g(\theta))}{\partial g(\theta)} \frac{\partial g(\theta)}{\partial \theta}
$$
- 缺点：
	- 朴素的符号微分会导致计算浪费和表达式膨胀
	- 例如：对于 $f(\theta)=\prod_{i=0}^{n}\theta_{i}$
		- 其关于每个 $\theta$ 偏导数微分为 $$\frac {f(\theta)}{\partial \theta_k}=\prod_{j \neq k}^{n}\theta_{j} $$
		- 若直接推导，计算所有偏导数 $\frac{\partial f(\theta)}{\partial\theta_{k}}$ 需要 $n(n - 1)$ 次乘法，但在实际上这中间的结果是可以进行复用的


---
- 为了更好地进行微分，我们可将计算过程表示为图：
	- **节点（Nodes）**：表示计算中的（中间）数值
	- **边（Edges）**：表示输入输出关系

- 示例：$y=f(x_{1},x_{2})=ln(x_{1})+x_{1}x_{2}-sin~x_{2}$

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251214173131549.png" width="400px"/>
</div>

- 前向计算轨迹（Forward evaluation trace）展示了从输入 $x_1, x_2$ 到中间变量 $v_1, ... , v_7$ 再到输出 $y$ 的具体数值计算过程

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251214173705076.png" width="350px"/>
</div>

---
3. **前向模式自动微分（Forward Mode AD）**

- 核心思想：
	- 定义 ： $\dot{v}_{i}=\frac{\partial v_{i}}{\partial x_{1}}$ ，即中间变量对于某个**特定输入**的导数
- 计算过程：
	- 按照计算图的**前向拓扑顺序**迭代计算 $\dot{v_i}$

- 例如：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251214173131549.png" width="450px"/>
</div>
- 其 **Forward AD trace**：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251214174122120.png" width="350px"/>
</div>

- 但该方法同样存局限性：
	- 对于函数 $f:\mathbb{R}^{n}\rightarrow\mathbb{R}^{k}$ ，为获取对于每个输入的梯度，前向模式（**Forward AD**）需要运行 $n$ 次
	- 在实际深度学习场景中，我们通常关注的是 $k = 1$（标量 Loss）且输出参数数量 $n$ 非常大的情况，因此前向模式不够高效

---
4. **反向模式自动微分（Reverse Mode AD）**

- 这是深度学习中最常用的方法（类似于 Backpropagation 反向传播）
- 核心思想：
	- 定义**伴随变量（Adjoint）**：$\overline{v_{i}}=\frac{\partial y}{\partial v_{i}}$，即最终输出 $y$ 对中间变量 $v_i$ 的偏导数
- 计算过程：
	- 按照计算图的**反向拓扑顺序**计算 $\overline{v_i}$

- 例如：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251214173131549.png" width="450px"/>
</div>

- 其 **Reverse AD evaluation trace**：
	- 从输出开始 ：$\overline{v_{7}}=\frac{\partial y}{\partial v_{7}}=1$
	- 反向进行传播

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251214180930812.png" width="450px"/>
</div>

- 在上述 Reverse Mode AD 的过程中，我们可以发现会出现一种情况：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251214181617360.png" width="450px"/>
</div>

- 当一个节点（例如 $\overline{v_i}$）被多个后续节点（$v_2、v_3$）使用时，根据多元链式法则，需要将来自所有路径的梯度相加 $$
\overline{v}_{1} = \overline{v}_{2}\frac{\partial v_{2}}{\partial v_{1}}+\overline{v}_{3}\frac{\partial v_{3}}{\partial v_{1}}
$$
- 定义**部分伴随（Patical Adjoint）**：$$\overline{v_{i\rightarrow j}}=\overline{v_{j}}\frac{\partial v_{j}}{\partial v_{i}}$$ 为节点 $j$ 传回给 $i$ 的梯度部分

- 总伴随：$$\overline{v_{i}}=\sum_{j\in next(i)}\overline{v_{i\rightarrow j}}$$
---

###   反向模式微分算法实现：

1. 初始化 `node_to_grad` 字典, 记录每个节点的梯度列表
2. 按反向拓扑顺序遍历节点 `i`
3. 求和：$\overline{v_i} = sum(node\_to\_grad[i])$
4. 传播：对于 `i` 的每个输入 `k`，计算部分伴随 $\overline{v_{i\rightarrow j}}$ 并将其添加到 `k` 的梯度列表中

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251216105528167.png" Higth="400px"/>
</div>


---

- 从拓展计算图的视角：我们可以发现，该算法不仅是计算数值，而是通过构建新的计算图来实现反向传播
- **核心思想**：构建一个能够计算伴随值 (`Adjoint values`) 的计算图，将"求导"转化为"图的构建"，使得梯度计算本身也是一个可执行的图
- 构建过程：
	- 原始图计算 `y`
	- 在图中添加新的节点来表示梯度计算
		- 例如：为计算 $\overline{v_1}$，我们图中添加乘法节点以及加法节点，这些节点操作本身也是图的一部分

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251216111657198.png" Higth="400px"/>
</div>


---
### 张量视角下的反向模式 AD：

- 在深度学习中通常处理张量（矩阵）而非标量：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251216112244670.png" Higth="400px"/>
</div>

- 标量计算：
	- Forward evaluation trace:
		- $Z_{ij} = \sum_{k} X_{ik} W_{kj}$
		- $y = f(Z)$
	- Reverse evaluation in scalar form:
		- 标量形式推导： $\overline{X_{i,k}} =\sum_{j} \frac{\partial z_{i,j}}{\partial x_{i,k}} \overline{Z}_{i,j} = W_{k,j} \overline{Z}_{i,j}$ 

- 张量计算：
	- Forward evaluation trace:
		- 前向矩阵定义：$Z = XW$
		- $y = f(Z)$
	- Reverse evaluation in tensor form:
		- 伴随张量值 (Adjoint for tensor value) ： $\overline{Z}$ 是一个与 $Z$ 形状相同的矩阵，其中每个元素为 $\frac{\partial y}{\partial Z_{ij}}$
		- 矩阵形式推导：$\overline{X}=\overline{Z}W^{T}$


---

#### 反向模式 AD 与 Backprop 的区别：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251216114436882.png" Higth="400px"/>
</div>

- **Backprop（传统反向传播）**：
	- 机制：在同一前向图中直接进行反向操作（即就地计算数值）
	- 代表框架：
		- 第一代框架：`Caffe、Cuda-convnet`
	- 特点：
		- 通常是为了计算数值而设计，梯度计算过程没有显式地构建新的图节点

- **Reverse mode AD by extending graph（拓展图的反向模式）**：
	- 机制：为伴随变量（梯度）创建独立的图节点，即，梯度的计算过程本身也会生成新的节点，添加到原有计算图中
	- 代表框架：
		- 现代框架：`PyTorch、TensorFlow、MXNet`

- **优劣势分析**
	- **扩展图方法 (现代 Reverse Mode AD) 的优势**：
	    - **支持高阶导数 (Gradient of Gradient)**：由于反向传播的结果仍然是一个计算图，我们可以对这个“梯度图”再次运行反向模式 AD 。这使得计算 Hessian 矩阵或进行元学习（Meta-learning）等需要高阶导数的任务变得非常容易，这也是现代框架采用此方法的主要原因。
        
	- **传统 Backprop 的局限**：
	    - 由于它只是在原图上进行数值计算，没有构建新的图结构，因此处理高阶导数时会更加困难或需要专门的硬编码实现。


---

#### 数据结构上的 Reverse Mode AD

- AD 不仅仅适用于标量以及张量，同样可以拓展延申到更复杂的数据结构上（如结构体、字典、元组等）

- **核心思想：**
	- 伴随量的结构：伴随值（梯度）的数据类型通常应当与前向传播时的数据类型保持一致
	- 即只要定义好每种数据类型的"伴随值"形式以及对应的传播规则，通用的 AD 算法就可以直接工作，不需要为每种数据结构写特殊的算法


- 示例：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251216125404510.png" Higth="350px"/>
</div>

- **前向 (Forward)**：
	- 假设有一个字典数据结构 `d = {"cat": a0, "dog": a1}`，并通过操作 `b = d["cat"]` 取值，最后计算得到 $v$ 。

- **反向 (Reverse)**：
	- 为了计算 `d` 的梯度 $\overline{d}$，我们需要构建一个与 `d` 结构相同的字典。
	- 结果是 $\overline{d}=\{"cat": \overline{b}, "dog": \dots\}$ 。这里 $\overline{b}$ 是从后续节点传回来的梯度。





## Fully Connected networks, optimization

### 全连接网络：

1. **数学定义**

- 一个 $L$ 层的全连接网络（又称多层感知机 MLP），先引入 **显式偏置项（Bias term）**，其迭代过程定义如下：

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20251216225453091.png" width="400px"/>
</div>

- 层递推公式：$$z_{i+1}=\sigma_{i}(W_{i}^{T}z_{i}+b_{i}), i=1,...,L$$
- 网络输出：$$h_{\theta}(x)\equiv z_{L+1}$$
- 输入初始化：$$z_{1}\equiv x$$
- 其中参数集合：$\theta=\{W_{1:L},b_{1:L}\}$ 
- 激活函数：$\sigma_i(x)$ 为非线性激活函数，通常最后一层为线性输出，即 $\sigma_k(x) = x$

2. **矩阵化以及广播（Broadcasting）**:

- 而在实际深度学习网络中，我们不会一次处理单个数据，而是通常处理一批数据（Batch），该迭代公式的矩阵形式为：$$Z_{i+1}=\sigma_{i}(Z_{i}W_{i}+1b_{i}^{T})$$
- 在矩阵运算中，偏置项 $b$ 本是一个 $n$ x 1 的向量，但为了使得维度匹配，$Z_{i+1}\in\mathbb{R}^{m\times n}$ 要求我们通常将偏置向量映射为矩阵 $1b_{i}^{T}\in\mathbb{R}^{m\times n}$，使得其与数据矩阵维度相同
	- 在实际中，我们并不显示构建这些矩阵，而是通过广播（Broadcasting）机制来完成，广播机制在处理 $n$ x 1 向量时，会在逻辑上将其视为重复排序的矩阵，而不会产生实际复制内存


---
### 训练深度网络的关键问题：

要成功训练一个全连接网络（或者其他深度网络），我们必须解决一下互相关联的问题：

1. 架构设计：如何选中网络的宽度以及深度
2. 优化算法：如何优化目标函数
3. 权重初始化：如何设定权重的初始值
4. 训练稳定性：如何确保网络在多个优化迭代中保持已于训练的状态

---
### 优化算法（Optimization）：

- 训练网络的目标是最小化损失函数，之前常用的方法是随机梯度下降（SGD），但在实际应用中通常会选择更高级的变体

#### 1. 梯度下降（Gradient Descent，GD）

- 基础的梯度下降更新公式如下：$$
\theta_{t+1}=\theta_{t}-\alpha\nabla_{\theta}f(\theta_{t})
$$
- $\alpha > 0$：步长，即**学习率 (Learning Rate)**。 
- $\nabla_{\theta}f(\theta_{t})$：在当前参数下的梯度。 

- 核心思想