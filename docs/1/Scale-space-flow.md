# Scale-space flow for end-to-end optimized video compression - CVPR2020

> 创新点：

本文提出了一套全新的端到端视频编解码框架。针对现有基于学习的视频编解码需要光流、双线性warping和运动补偿，而且有相对复杂的架构和训练策略(需要预训练光流、训练各个子网络、训练过程中重建帧需要缓冲区)，本文提出一种广义warping操作，可以处理比如去遮挡、快速运动等复杂问题，而且模型和训练流程大大简化。

> 贡献

本文对现有的基于学习的包含光流估计+运动补偿的框架总结出三个问题：

- 光流预测需要解决孔径问题（光流之所以是个病态问题的原因），这个问题比压缩问题更复杂；
- 编解码框架中加入光流网络，给整个编解码框架增加了约束和复杂度；
- 稠密光流没有“no use”的概念，每个像素都要进行warped，导致无遮挡情况下会有较大残差。

针对上面三个现有框架缺点，作者提出改进措施如下：

- 提出尺度空间光流和warping，一种对光流+双线性warping的直观概述
- 简单的编解码框架和训练过程
- 实验结果显示超过了基于训练的视频编解码的state-of-the-art结果，而且消融实验也表明了方法的有效性

> 尺度空间光流

<img src="https://i.imgur.com/LYPt4DB.png" >

构造光流时引入了**scale field**

- 一般的warping:

$$
x':=Bilinear-Warp(x,f) \space \space \space \space \space \space s.t.
\space  \space  \space \space \space x'[x,y]=x[x+f_x[x,y],y+f_y[x,y]]
$$

- 包含尺度空间的warping

$$
x':=Scale-Space-Warp(x,g) \space \space \space \space \space \space s.t.\space  \space  \space \space \space x'[x,y]=X[x+g_x[x,y],y+g_y[x,y],g_z[x,y]]
$$

尺度空间$Scale-Space-Warp$，主要就是构造了一个尺度固定的volume $X$:
$$
X=[x,x*G(\sigma_0),x*G(2\sigma_0),...,x*G(2^{M-1}\sigma_0)]
$$
其中，$x*G(\sigma_0)$表示x与尺度为$\sigma$的高斯卷积核进行卷积，$X$与$x$具有不用模糊尺度的图片组成的堆栈，维度为$M\times N \times （M+1）$，作者采用5。$g^{_{_{z}}}=0$时，尺度空间的$Scale-Space-Warp$就退化为$Bilinear-Warp$：
$$
Scale-Space-Warp(x,(g_x,g_y,0))=Bilinear-Warp(x,(g_x,g_y))
$$
当$g^{x}$和$g^{y}$为0，$g^{z}=log_{2}{(\sigma/\sigma _{0}) }$时，$Scale-Space-Warp$就近似于高斯模糊：

$$
Scale-Space-Warp(x,(0,0,1)+log_2(\sigma/\sigma_0))\approx x*G(\sigma)
$$
$X$可以通过简单的三线性插值获取，而坐标z这样计算：$z=\frac{i+(\sigma^2_b-\sigma^2)}{\sigma^2_b-\sigma^2_a}$,其中，$0 < σ <2^{^{M-1}}σ_0$

> 压缩模型

主帧压缩、残差压缩、尺度空间光流计算均采用balle那一套框架。

<img src="https://i.imgur.com/R9wpujX.png" >

> 训练策略

损失函数
$$
\displaystyle\sum^{N-1}_{i=0}d(x_i,\hat{x}_i)+\lambda [H(z_0)+\displaystyle\sum^{N-1}_{i=0}H(v_i)+H(w_i)]
$$

- 先用256 x 256的图片训练，再用分辨率较大的384 x 384图片进行微调，这和复现DVC时的训练策略一致。
- N(Number of unrolled frames)采用9或12取得较好结果，在Nvidia V100 GPU上训练了**30天**，基于学习的视频编解码训练确实消耗时间。

> 结论

提出尺度空间光流和$Scale-Space-Warp$，而且简化了编解码流程，与DVC相比，是完全不同的框架。为基于深度学习的端到到视频编解码框架提供了一个新的思路。但从这个文献结果来看，依然是只比人工设计的传统的混合编解码框架H265略好，像本文这么简化的框架能达到如此效果已经很令人惊喜。