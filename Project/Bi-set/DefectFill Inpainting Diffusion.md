

### 1. 模型推理：

![image.png](https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20260111220528233.png)


#### 1.0 前置基础知识：
##### Duffsion  

-  Duffsion 模型（尤其是 Denoising Diffusion  Probabilistic Models，**DDPM**）是一种生成式模型，其学习方式是 "*先加噪，再去噪*"
	- *前向过程（加噪）*：拿一张真实图像，逐步加高斯噪声，知道变成纯噪声。这个过程是固定的、可计算的
	- *反向过程（去噪）*：训练一个神经网络（通常是 UNet），从纯噪声开始，一步步预测并去除噪声，最终恢复出真实图像
	- 训练目标：网络在*某个中间噪声水平 t*（timestep，从 1 到 T，比如 T=1000）看到的加噪图像 $x_t$，预测这次“这次加的噪声的  $\varepsilon$ 是多少 ”损失就是预测噪声和真实噪声的 *L 2* 差（均方误差）
---  

- **Latent Diffusion Models** (LDM, 如 Stable  Diffusion) ^8cfb56
	- 直接在像素上做 Duffsion 太慢，所以先用 VAE encoder (变分自编码器) 将图像压缩到低维 latent 空间（比如从 512 x 512 像素压缩到 64 x 64 的 lantent）
	- 加噪/去噪都在 lantent 上进行，最后用 VAR decoder 解码回像素图像
	- *条件 Diffusion*：可以给 UNet 额外输入"条件"（如 text embedding），让生成受控

> [!NOTE] Text Condtioning（文本条件）：
> - 通过 CLIP Text Encoder 把 prompt 编码成一个向量序列 $c$
> - 这个 $c$ 通过 cross-attention 机制注入到 UNet 中：UNet 的特征图会关注 prompt 中提到的概念
> - 如果 prompt 里有特殊 token（如 [V*]）, 通过训练，这个 token 的 embedding 会逐渐学会对应某种视觉概念
> 

---
- **Inpainting Diffusion**：
	- 专门用于 "填补图像确实部分"
	- 输入不仅有加噪 lantent $x_t$，还额外提供：
		- 背景 lantent $b$ ：已知部分的 lantent，通常是原图去掉掩码区域
		- 掩码 $M$ ：二值图，1 = 需要填补，0 = 已知背景
	- 模型输入是 $concat(x_t, b, M)$, 这样模型知道 "哪些区域已知，哪些需要生成"

---
 - **UNet 在 Duffsion 中的作用**：
	 - UNet 是一个带 skip connection 的编码-解码网络
	 - 在条件 Diffusion 中，它接收：加噪 latent $x_t$ + timestep $t$ （嵌入式向量） + text embedding $c$
	 - 输出：预测的噪声图（和 $x_t$ 同形状）
	 - 内部主要靠 cross-attention 层把 text 条件融合进视觉特征

#### 1.1  Defect 缺陷学习：

![image.png](https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20260113154430033.png)

- Defect分支（论文Fig. 2上管道，Defect Loss部分）是DefectFill训练的核心之一，其目标是**让模型纯粹学习缺陷本身的局部视觉特征（纹理、颜色、结构等）**，使 `[V*]` token成为一个“独立于物体整体语义的缺陷概念”。这确保了 inference 时的高泛化能力（即使掩码形状奇怪或背景不同，也能生成真实缺陷）。

- 完整流程（结合论文公式和Fig. 2）：
	1. **输入准备**：
	    - 缺陷图像 $I$ + 缺陷掩码 $M$（来自少量参考样本）。
	2. **Latent 编码与加噪（标准LDM过程，两个分支共享）**：
	    - 通过 VAE encoder：$x_0 = \varepsilon(I)$（将像素图像压缩到 latent 空间，避免像素级扩散的计算开销）。
	    - 随机采样 timestep $t$ ∼ $p (t)$（噪声调度器）。
	    - 加噪：$x_t = \sqrt{α_t} ⋅ x_0 + \sqrt{1 - α_t} ⋅ \varepsilon$ （$\varepsilon$ ∼ $𝒩(0,1)$，公式(1)）。
	        - 这模拟了 diffusion 前向过程，$x_t$ 是“部分加噪的 latent ”。
	 
	3. **Defect 分支特有：Inpainting 输入构造**：
	    - 背景图像：$B_{def} = (1 - M) \bigodot I$（缺陷区域被掩蔽为黑/零）。
	    - 背景 latent：$b_{def} = \varepsilon(B_{def})$ （干净的背景参考）。
	    - 模型主输入：$x_t^{def} = concat (x_t, b_{def}, M)$ （公式(4)，沿通道维度拼接）。
	        - 含义：$x_t$ 提供全图加噪信息，$b_{def}$ 提供缺陷区无信息的干净背景，M告诉模型“哪里需要填补”。这强制模型知道背景已知，**防止随意修改背景，只专注掩码区域的学习与生成**。
	
	4. **Text Conditioning（条件控制）**：
	    - Prompt：$P_{def}$ = "A photo of `[V*]`" （纯粹描述 `[V*]`，不提及物体如 hazelnut）。
	        - 设计目的：让 `[V*]` token 的 embedding **独立捕捉缺陷本身的视觉特征**，不与特定物体语义绑定。如果加物体描述（如"A hazelnut with `[V*]`"），`[V*]` 会过度依赖物体上下文，泛化差。
	        - 结果：通过 LoRA 微调，`[V*]` 学会“纯缺陷概念”，inference 时可在任意背景/任意形状掩码中生成真实细节（论文强调这点，实现 Fig.1 和 Fig.4 的多样性）。
	    - 通过 LoRA 微调的 Text Encoder：$c^{def} = \tau_{\theta}(P_{def})$ （输出文本 embedding 序列）。
	
	5. **UNet 前向（LoRA 微调的 Inpainting UNet）**：
	    - 输入：
	        - 主干：$x_t^{def} = concat(x_t, b_{def}, M)$（concat 后的加噪+背景+掩码）。
	        - 时间条件：$t$（嵌入为 sinusoidal 向量，告诉 UNet 当前噪声水平）。
	        - 文本条件：$c^{def}$（通过 cross-attention 层注入，让 UNet 特征“关注” `[V*]` 对应的缺陷概念）。
	    - 输出：预测噪声 $\varepsilon_{\theta}(x_t^{def}, t, c^{def})$。

	6. **Defect Loss 计算**：
	    - $L_{def} = 𝔼 [ || M \bigodot ( \varepsilon - \varepsilon_{\theta}(x_t^{def}, t, c^{def})) ||² ]$ （公式(5)）。
	        - 只在缺陷掩码 $M$ 区域计算预测噪声与真实噪声 $\varepsilon$ 的 L2 误差，背景区域被 mask 掉。
	        - 含义：强制模型在 `[V*]` 条件 + 已知背景的前提下，把掩码区 denoise 成参考缺陷的样子，**纯粹学习缺陷细节**。
**整体机制总结**：
- $concat(x_t, b_def, M)$：提供干净背景参考，防止模型改动背景，专注缺陷区域修复。
- **独立prompt + masked loss**：让 [V*] 学会纯缺陷概念，不绑定物体，实现高泛化（这是 Defect 分支区别于 Object 分支的关键 —— Object 分支会补充物体语义关系）。




