# Neural Inter-Frame Compression for Video Coding - ICCV2019

> 创新点：

- 本文提出了一个建立在基于深度学习**图像压缩**基础上的端到端的视频编解码框架。
- 时间冗余通过光流的**像素空间**中的**运动补偿**来进行。
- 通过将所需信息编码为**运动和混合系数**的潜在表示形式，从而提高解码效率和重建质量。

> 算法框架

将视频分成GOP，每一个GOP视频帧数为n，第1帧和第n帧采用图片压缩算法，其余帧采用**帧间压缩**进行压缩。

<img src="https://i.imgur.com/aM1POVH.png" >

> 深度图片压缩原理

基于神经网络的图片编解码的主要任务就是达到重构图片与原始图片的失真预期时尽量减小图片映射空间（编码）的码流长度，也即优化如下的率失真（ rate-distortion ）函数：
$$
L(g_\phi,g_{\phi'},p_{\hat{y}})=E_{x\sim p_x}[\underbrace{-log_2 p_{\hat{y}}(\hat{y})}_{rate}+\lambda \underbrace {d(x,\hat{x})}_{distortion}]
$$

- $g_\phi$图像空间投影到潜在空间的投影函数----编码器
- $g_{\phi'}$潜在空间反投影到图像空间的函数----解码器
- $p_{\hat{y}}$熵编码
- $\hat{y}$编码的量化表示
- $d(x,\hat{x})$失真度量，比如MSE

本文通过运动补偿信息传输来利用时间冗余

$$
x_{intrp}=\displaystyle\sum^{k}_{i=1}\hat{\alpha}_iw(x_i,\hat{f}_i)\space with \space \displaystyle\sum^{k}_{i=1}\hat{\alpha_i}=1
$$

- ${\hat{\alpha}}_i$混合系数
- $w$Warping function
- $x_i$参考帧
- $\hat{f}_i$位移图

> 创新一：具有压缩限制的插值

<img src="https://i.imgur.com/WrJ4JC7.png" >

两个优势：

- 原始帧x一起被送到网络中提供warping结果，可以更好地预测混合系数（ blending coefficients ）。此外，对重构帧而不是对重构的运动场进行惩罚，可以使网络自己推断出对插帧更有利的运动场向量。
- 减少了计算时间，在解码端避免了复杂的帧插值。

> 创新二：潜在空间残差

上述的框架重构出的视频帧会有明显误差，需要通过残差来降低误差。本文没有为残差编解码设计独立的网络，而是利用图片编解码对$(g_{\phi},g_{\phi'})$将残差信息投影到潜在空间上

$$
r=y-y(intrp)=g_\phi(x)-g_\phi(x_{intrp})
$$

$$
\hat{x}=g_{\phi'}(y_{intrp}+\hat{r})
$$

<img src="https://i.imgur.com/zhsW0Bm.png" >

> 网络选择

- 图片编解码网络选择经典的 Balle等提出的含有GDN/IGDN结构的图片压缩网络。
- 插帧和主帧都用此网络架构，不同的是图片压缩网络解码端输出通道3，对应RGB通道，插帧网络的解码端输出通道是5，其中四个通道对应运动向量，一个通道对应混合系数。
- 光流采用PWC-net。
- 量化：训练时加均匀噪声，推理时直接rounding取整

> 结论

- 与传统的经过数十年工程改进的算法（如H265）相比，本文算法有一定竞争力
- 内插任务中嵌入了压缩约束，编码阶段利用了所有可用的信息 
- 关键帧和残差都使用相同的网络压缩，简化了视频编解码任务
- 本文只研究了利用前后主帧进行插帧，并未研究只利用历史帧进行帧间预测