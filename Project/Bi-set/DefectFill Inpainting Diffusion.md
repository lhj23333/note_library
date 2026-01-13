---
share_link: https://share.note.sx/kvuki5p4#ZsUNskOm5Qo0LSulbNLgeF1wVma1es8lUQgPVGT/7fA
share_updated: 2026-01-13T19:26:52+08:00
---


### 1. 模型训练

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20260111220528233.png" width="750px"/>
</div>

#### 1.1 前置基础知识

- **Diffusion 模型**（尤其是 Denoising Diffusion Probabilistic Models，**DDPM**）是一种生成式模型，其学习方式是"*先加噪，再去噪*"
	- *前向过程（加噪）*：拿一张真实图像，逐步加高斯噪声，直到变成纯噪声。这个过程是固定的、可计算的
	- *反向过程（去噪）*：训练一个神经网络（通常是 UNet），从纯噪声开始，一步步预测并去除噪声，最终恢复出真实图像
	- *训练目标*：网络在某个中间噪声水平 $t$（timestep，从 1 到 T，比如 T=1000）看到的加噪图像 $x_t$，预测"这次加的噪声 $\varepsilon$ 是多少"，损失就是预测噪声和真实噪声的 $L_2$ 差（均方误差）
---  

- **Latent Diffusion Models**（LDM，如 Stable Diffusion） ^9 b 93 f 2
	- 直接在像素上做 Diffusion 太慢，所以先用 VAE Encoder（变分自编码器）将图像压缩到低维 latent 空间（比如从 512×512 像素压缩到 64×64 的 latent）
	- 加噪/去噪都在 latent 上进行，最后用 VAE Decoder 解码回像素图像
	- *条件 Diffusion*：可以给 UNet 额外输入"条件"（如 text embedding），让生成受控

> [!NOTE] Text Condtioning（文本条件）：
> - 通过 CLIP Text Encoder 把 prompt 编码成一个向量序列 $c$
> - 这个 $c$ 通过 cross-attention 机制注入到 UNet 中：UNet 的特征图会关注 prompt 中提到的概念
> - 如果 prompt 里有特殊 token（如 [V*]）, 通过训练，这个 token 的 embedding 会逐渐学会对应某种视觉概念
> 

^8d8aef

---
- **Inpainting Diffusion**：
	- 专门用于"填补图像缺失部分"
	- 输入不仅有加噪 latent $x_t$，还额外提供：
		- 背景 latent $b$：已知部分的 latent，通常是原图去掉掩码区域
		- 掩码 $M$：二值图，1 = 需要填补，0 = 已知背景
	- 模型输入是 $\text{concat}(x_t, b, M)$，这样模型知道"哪些区域已知，哪些需要生成"

---
- **UNet 在 Diffusion 中的作用**：
	- UNet 是一个带 skip connection 的编码-解码网络
	- 在条件 Diffusion 中，它接收：加噪 latent $x_t$ + timestep $t$（嵌入式向量）+ text embedding $c$
	- 输出：预测的噪声图（和 $x_t$ 同形状）
	- 内部主要靠 cross-attention 层把 text 条件融合进视觉特征


#### 1.2 Defect 分支

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20260113154430033.png" width="750px"/>
</div>

- Defect 分支（论文 Fig. 2 上管道，Defect Loss 部分）是 DefectFill 训练的核心之一，其目标是**让模型纯粹学习缺陷本身的局部视觉特征（纹理、颜色、结构等）**，使 `[V*]` token 成为一个"独立于物体整体语义的缺陷概念"。这确保了 inference 时的高泛化能力（即使掩码形状奇怪或背景不同，也能生成真实缺陷）

- 完整流程（结合论文公式和Fig. 2）：
	1. **输入准备**：
	    - 缺陷图像 $I$ + 缺陷掩码 $M$（来自少量参考样本）
		
	2. **Latent 编码与加噪（标准 [[DefectFill Inpainting Diffusion#^9b93f2| LDM]] 过程，两个分支共享）**：
	    - 通过 VAE encoder：$x_0 = \varepsilon(I)$（将像素图像压缩到 latent 空间，避免像素级扩散的计算开销）
	    - 随机采样 timestep $t$ ∼ $p (t)$（噪声调度器）。
	    - 加噪：$x_t = \sqrt{\alpha_t} ⋅ x_0 + \sqrt{1 - \alpha_t} ⋅ \varepsilon$ （$\varepsilon$ ∼ $𝒩(0,1)$，公式(1)）
	        - 这模拟了 diffusion 前向过程，$x_t$ 是“部分加噪的 latepnt ”
			
	3. **Defect 分支特有：Inpainting 输入构造**：
	    - 背景图像：$B_{def} = (1 - M) \bigodot I$（⊙是element-wise乘，缺陷区域变成黑/零）
	    - 背景 latent：$b_{def} = \varepsilon(B_{def})$ （干净的背景参考）
	    - 模型主输入：$x_t^{def} = concat (x_t, b_{def}, M)$ （公式(4)，沿通道维度拼接）
	        - 含义：$x_t$ 提供全图加噪信息，$b_{def}$ 提供缺陷区无信息的干净背景，M告诉模型“哪里需要填补”。这强制模型知道背景已知，**防止随意修改背景，只专注掩码区域的学习与生成**
			
	4. **Text Conditioning（条件控制 [[DefectFill Inpainting Diffusion#^8d8aef|Conditioning]]）**：
	    - Prompt：$P_{def}$ = "A photo of `[V*]`" （纯粹描述 `[V*]`，不提及物体如 hazelnut）
	        - 设计目的：让 `[V*]` token 的 embedding **独立捕捉缺陷本身的视觉特征**，不与特定物体语义绑定。如果加物体描述（如"A hazelnut with `[V*]`"），`[V*]` 会过度依赖物体上下文，泛化差
	        - 结果：通过 LoRA 微调，`[V*]` 学会“纯缺陷概念”，inference 时可在任意背景/任意形状掩码中生成真实细节（论文强调这点，实现 Fig.1 和 Fig.4 的多样性）
	    - 通过 LoRA 微调的 Text Encoder：$c^{def} = \tau_{\theta}(P_{def})$ （输出文本 embedding 序列）
			
	5. **UNet 前向（LoRA 微调的 Inpainting UNet）**：
	    - 输入：
	        - 主干：$x_t^{def} = concat(x_t, b_{def}, M)$（concat 后的加噪+背景+掩码）
	        - 时间条件：$t$（嵌入为 sinusoidal 向量，告诉 UNet 当前噪声水平）
	        - 文本条件：$c^{def}$（通过 cross-attention 层注入，让 UNet 特征“关注” `[V*]` 对应的缺陷概念）
	    - 输出：预测噪声 $\varepsilon_{\theta}(x_t^{def}, t, c^{def})$
			
6. **Defect Loss 计算**：
	- $L_{def} = \mathbb{E} \left[ \left\| M \odot \left( \varepsilon - \varepsilon_{\theta}(x_t^{def}, t, c^{def}) \right) \right\|_2^2 \right]$（公式 (5)）
		- 只在缺陷掩码 $M$ 区域计算预测噪声与真实噪声 $\varepsilon$ 的 L2 误差，背景区域被 mask 掉
		- 含义：强制模型在 `[V*]` 条件 + 已知背景的前提下，把掩码区 denoise 成参考缺陷的样子，**纯粹学习缺陷细节**

> [!TIP] 整体机制总结
> - $\text{concat}(x_t, b_{def}, M)$：提供干净背景参考，防止模型改动背景，专注缺陷区域修复
> - **独立 prompt + masked loss**：让 `[V*]` 学会纯缺陷概念，不绑定物体，实现高泛化（这是 Defect 分支区别于 Object 分支的关键 —— Object 分支会补充物体语义关系）


#### 1.3 Object 分支

<div style="text-align: center;">
  <img src="https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20260113194023274.png" width="750px"/>
</div>

- Object分支（论文Fig. 2下管道，Object Loss + Attention Loss 部分）是 DefectFill 训练的“融合与定位”机制。其目标是**补充缺陷与物体整体语义的关系**（自然融入） + **强制[V] token注意力集中在缺陷区**（精准定位）。

- 完整流程（逐步骤，结合论文公式）：
	1. **共享的前置部分（和 Defect 分支完全相同）**：
	    - 输入：缺陷图像 $I$ + 真实缺陷掩码 $M$
	    - `VAE` 编码：$x_0 = \varepsilon(I)$
	    - 随机采样 timestep $t$ ∼ $p (t)$
	    - 加噪：$x_t = \sqrt{\alpha_t} ⋅ x_0+ \sqrt{1 - \alpha_t} ⋅ \varepsilon$ （相同 $\varepsilon$、$t$）
		
	2. **Object 分支特有：随机掩码 + 背景构造**：
	    - 生成随机掩码 $M_{rand}$：由**30 个随机 `boxes`** 组成（覆盖图像大部分区域，包括缺陷和正常部分）
	        - 目的：这些 boxes 覆盖图像大部分区域（包括缺陷和正常部分），模拟“大面积遮挡”，强制模型学习整个物体的完整语义（不是只看小缺陷区）。
	    - 背景图像：$B_{rand} = (1 - M_{rand}) \bigodot I$ （随机区域掩蔽为黑）
	    - 背景 latent：$b_{rand} = \varepsilon(B_{rand})$
	    - 模型主输入：$x_t^{obj} = concat (x_t, b_{rand}, M_{rand})$（公式(6)）
	        - 含义：提供大面积随机背景参考，模型必须用物体整体语义重建这些区域（类似于“泛化到大 inpainting 任务”）
		
	3. **Text Conditioning（与 Defect 分支关键区别）**：
	    - `Prompt `：$P_{obj}$ = "A `[Object]` with `[V*]`" （例如 "A hazelnut with `[V*]`"）
	        - 目的：明确绑定物体 Object，让 `[V*]` token 的 embedding 学会**在特定物体上下文下的缺陷表达**（边缘过渡自然、颜色/纹理匹配物体）
	        - 与 Defect 分支的纯 "A photo of `[V*]`" 对比：这里强制语义关联，实现“缺陷属于这个物体”的自然感
	    - 通过 LoRA 微调的 Text Encoder：$c^{obj} = \tau_{\theta}(P_{obj})$
			
	4. **UNet 前向（共享 LoRA 微调的 Inpainting UNet）**：
	    - 输入：
	        - 主干：$x_t^{obj}$（ concat 后的加噪+随机背景+随机掩码）
	        - 时间条件：$t$ embedding（告知噪声水平）
	        - 文本条件：$c^{obj}$（ cross-attention 注入，让特征关注“物体 + `[V*]`”）
	    - 输出：预测噪声 $\varepsilon_{\theta}(x_t^{obj}, t, c^{obj})$
	    - **额外提取**：在这一前向过程中，提取 cross-attention maps（用于 $L_{attn}$）
			
	5. **Object Loss (L_obj) 计算**（公式 (7)）：
	    - 构造加权掩码：$M' = M + \alpha ⋅ (1 - M)$ （$alpha$ < 1，缺陷区权重为1，背景区权重 $α$ 较小）
	    - 损失：$L_{obj} = \mathbb{E} [\begin{Vmatrix} M' \bigodot (\varepsilon - \varepsilon_{\theta}(x_t^{obj}, t, c^{obj})) \end{Vmatrix}_2^2 ]$
	    - 含义：
	        - 主要监督大随机掩码区域的重建（学整体语义）
		- 但真实缺陷区高权重为 1 → 确保缺陷细节与连接区域不被忽略
	- 目的：既学物体整体（通过随机大掩码重建），又突出缺陷区（权重为 1），实现"缺陷自然融入物体"的效果

6. **Attention Loss ($L_{attn}$) 计算**（公式 (8)，只用 Object 分支的输出）：
	- 从 UNet **decoder layers**（不是 encoder）提取 cross-attention maps
		- 为什么只用 decoder？
			- 论文引用 `[4]`：encoder 的 attention 不准，decoder 更能代表 token 的布局
		- 针对 `[V*]` token：
			- 取出它的 attention maps（每个 layer 一个），resize 到 latent 大小，平均得到 $A_t^{[V*]}$
	- 损失：
		- $L_{attn} = \mathbb{E} \left[ \left\| A_t^{[V*]} - M \right\|_2^2 \right]$
		- 含义：强制 `[V*]` token 的注意力图尽量贴近真实缺陷掩码 $M$（缺陷区高注意力，背景区低）
		- 目的：防止 `[V*]` 的关注扩散到整个物体或背景，确保生成的缺陷精准定位在掩码区（提升空间准确性）

> [!TIP] Object 分支总结
> Object 分支是 DefectFill 训练的"补丁"和"强化"机制。Defect 分支让模型学会"纯缺陷细节"（独立概念），但这可能导致生成的缺陷虽然细节丰富，却和物体整体语义不协调（比如 hole 边缘不自然、颜色不融合）。Object 分支的使命就是补充缺陷与物体整体的语义关系，让缺陷"自然地属于这个物体"（$L_{obj}$），同时强制 `[V*]` token 的注意力集中在缺陷区域（$L_{attn}$）。

> [!NOTE] 个人理解
> - 对于 Object 分支，其流程整体性与 Defect 分支相似
> - 在前置采样（Latent Diffusion Model 前向加噪）阶段：需要先将输入缺陷样本图像 $I$ 经过 VAE Encoder 由像素域转换到 latent 域 $x_0 = \varepsilon(I)$，并采用随机采样 timestep $t$ 的方式进行加噪：$x_t = \sqrt{\alpha_t} \cdot x_0 + \sqrt{1 - \alpha_t} \cdot \varepsilon$
> - 进入到构建 Inpainting Duffsion Model 构建输入阶段：在 Object 分支，为了使得模型可对缺陷图像整体（包括缺陷以及正常的部分）有更好的学习理解，即学习整个物体的完整语义，采取生成随机掩码 $M_{rand}$ —— 由 30 个随机 `boxes` 覆盖图像大部分区域的形式。与 Defect 分支相同，将 $M_{rand}$ 与输入缺陷图像进行 element-wise 处理，使得随机掩码区域变为零/黑得到 $B_{rand} = (1 - M_{rand}) \bigodot I$，再由 LDM 模型特性，对其进行 latent 维度转换处理，得到 $b_{rand} = \varepsilon(B_{rand})$，得到模型主输入：$x_t^{obj} = concat(x_t, b_{rand}, M_{rand})$，其主要含义为：*给予模型大面积掩码的背景图像 ( latent 域) $b_{rand}$ 与正确图像 $x_t$ 通道拼接，使得模型本次面对的 Inpaint 任务为大面积随机区域的 mark（类似于 Defect 分支让模型专注于缺陷 mark 区域的图像学习的泛化理解）*
> 
> - 对于 Model Conditioning，依据 LDM 模型的 Condition Model 特性：采用 Prompt $P_{obj}$ = "A `[Object]` with `[V*]`" （例如 “A hazelnut with `[V*]`"), 将生成内容 `[V*]` 的语义与物体 Object 进行关联绑定，使得生成内容 `[V*]` 学习物体整体上下文下的缺陷表达，达到较为不错的整体性生成效果。通过 LoRA 微调过的 Text Encoder 得到模型的 Text Conditioning：$c^{obj} = \tau_\theta(P_{obj})$
> 
> - 至此，Inpainting Diffusion Model 得到了它的输入：主干 $x_t^{obj} = \text{concat}(x_t, b_{rand}, M_{rand})$、timestep $t$、text condition $c^{obj}$，进行模型的反向去噪：训练 UNet 得到在学习缺陷整体过程中的预测噪声：$\varepsilon_{\theta}(x_t^{obj}, t, c^{obj})$
> - 最终计算 Object Loss ($L_{obj}$)：在 Object 模型训练过程中，主要是学习缺陷在物体 Object 整体上下文的表达，但为了使得生成的效果真实更好，即需要注重真实缺陷区域与背景区域的连接区域特征，所以引入了一个 weight param $\alpha$，对真实缺陷区域掩码进行处理：$M' = M + \alpha \cdot (1 - M)$（$\alpha < 1$），使得真实缺陷区域权重为 1，背景区权重为 $\alpha$，即在缺陷掩码中给予了背景图像一定的权重，以确保缺陷与背景的连接细节不被忽略。计算预测噪声与真实噪声的 $L_2$ 距离作为 Object Loss：$L_{obj} = \mathbb{E} \left[ \left\| M' \odot \left( \varepsilon - \varepsilon_{\theta}(x_t^{obj}, t, c^{obj}) \right) \right\|_2^2 \right]$，这样子既能保证模型能够学习物体的整体（通过随机掩码重建），又不忽略缺陷区域与背景区域的连接（通过权重 $\alpha$），以使得缺陷能自然融入物体
> - 同时在 Text-to-image 模型中，token 的 cross-attention 有时会"跑偏"（`[V*]` 关注整个物体），为使得 Text Conditioning 的 Prompt 的 `[V*]` 关注于缺陷掩码区域，引入 $L_{attn}$：通过从 UNet decoder layers 每一层的 decoder 中提取 cross-attention maps，resize 到 latent 大小，平均得到 $A_t^{[V*]}$ 以计算 $L_{attn}$：$L_{attn} = \mathbb{E} \left[ \left\| A_t^{[V*]} - M \right\|_2^2 \right]$，强制 `[V*]` token 的注意力图尽量贴近真实缺陷掩码 $M$（缺陷区高注意力，背景区低）


#### 1.4 总损失

- **总损失计算公式**：
$$L_{ours} = \lambda_{def} \cdot L_{def} + \lambda_{obj} \cdot L_{obj} + \lambda_{attn} \cdot L_{attn}$$

- **论文实验权重**：
	- $\lambda_{def} = 0.5$（最重要，细节为核心）
	- $\lambda_{obj} = 0.2$
	- $\lambda_{attn} = 0.05$（辅助，规模小）

- **三者平衡**：
	- $L_{def}$：主！纯缺陷细节
	- $L_{obj}$：整体融合 + 连接自然
	- $L_{attn}$：精准定位

- **三损失合力** → 高度逼真、泛化强的缺陷

### 2. 模型推理

> [!TODO] 待补充