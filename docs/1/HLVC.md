# Learning for Video Compression with Hierarchical Quality and Recurrent Enhancement
> [代码](https://github.com/RenYang-home/HLVC)

> 创新点：

提出层级式视频压缩(Hierarchical Learned Video Compression,**HLVC**)，包括双向深度预测(Bi-Directional Deep Compression, **BDDC** )和单运动深度压缩(Single Motion Deep Compression,**SMDC**)。BDDC主要用于压缩第二层视频帧，SMDC采用单个运动向量预测多个帧，可以节省运动信息的码率。在解码端，利用加权递归质量增强(Weighted Recurrent Quality Enhancement,**WRQE**)网络，此网络将压缩帧和比特流作为输入。

> 算法框架

<img src="https://i.imgur.com/oTVtJm1.png" >

- 第一层 图片压缩技术进行压缩
- 第二层 BDDC网络根据相邻帧进行压缩
- 第三层 采用SMDC网络进行压缩
- 第四步 WRQE进行递归增强

> HLVC

- 优势一 高质量的视频帧可以给其他帧提供更多的信息
- 优势二 由于相邻帧之间的高相关性，在解码器端，可以通过利用高质量帧中的有利信息来增强低质量帧
<table>
    <tr>
        <td >
            <center>
                <img src="https://i.imgur.com/8WP7uiC.png" >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图1 HLVC</b>
                </div>
    			<br>
            </center>
        </td>
    </tr>
</table>

> BDDC

<img src="https://i.imgur.com/bHFkC6s.png" >			
     
- 进行光流估计(采用**spyNet**网络)，两个光流一起送到光流压缩编码网络中
- 编码后的特征采用算术编码进行编码，再进行round量化(可考虑深度学习的熵编码替换）
- 送到运动后处理网络（DVC中运动补偿网络)
- 进行残差压缩

**与DVC不同的就是运动压缩和运动补偿网络是前后两帧和运动矢量一起送给网络（MP和RC网络Balle的GDN的网络)**

<table>
    <tr>
        <td >
            <center>
                <img src="https://i.imgur.com/PKW9HNb.png" >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图1 MP网络结构</b>
                </div>
    			<br>
            </center>
        </td>
    </tr>
</table>

> SMDC

<table>
    <tr>
        <td >
            <center>
                <img src="https://i.imgur.com/fjJaGlK.png" >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图1 SMDC</b>
                </div>
    			<br>
            </center>
        </td>
    </tr>
</table>
$x_0->x_2$的光流需要网络估计，$x_1$的光流假设运动均匀的情况下进行估算

- $f_{inv}(a+\Delta{a}(a,b),b+\Delta{b}(a,b))=-f(a,b)$

- ${\hat{f}}_{1 \to 0}=Inverse\underbrace{(0.5 \times \underbrace{Inverse({\hat{f}}_{2 \to 0})}_{{\hat{f}}_{0 \to 2}})}_{{\hat{f}}_{0 \to 1}}$

- ${{\hat {f}}_{1 \to 2}}=Inverse(\underbrace {0.5 \times {{\hat {f}}_{2 \to 0}}}_{{\hat {f}}_{2 \to 1}})$



> WRQE(加权递归增强)

<table>
    <tr>
        <td >
            <center>
                <img src="https://i.imgur.com/fO6tI7a.png" >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图1 基于QG-ConvLSTM的网络结构</b>
                </div>
    			<br>
            </center>
        </td>
    </tr>
</table>


> 训练策略
>
>> 率失真损失函数

- BDDC损失函数：$L_{BD}=\lambda_{BD}\cdot D(\underbrace{x_5,x^{C}_5}_{Distortion})+\underbrace{R({\hat {q}_{m}+R({\hat {q}_r})})}_{Total\space bit-rate}$

- SMDC损失函数：$L_{SM}=\lambda_{SM}\cdot\underbrace{(D(x_1,x^{C}_1)+D(x_2,x^{C}_2)}_{Total\space distorsion}+\underbrace{R({\hat{q}_m})+R({\hat{q}_{r1}})+R({\hat{q}_{r2}})}_{Total \space bit-rate}$

- WRQE损失函数：$L_{QE}=\frac{1}{N}\displaystyle\sum_{i=1}^{n}D(x_i,x^{D}_i)$

> 结论

这篇文献没有大的突破，同时第二层的复杂度和DVC相当，再加上第三层和后增强网络，这个框架过于复杂，尤其增强网络是LSTM。另外，第三层网络利用一个光流预测多帧，直观上看是节省了一些光流，这个实验有尝试过，其实大多数情况下，光流所占的比特还是很小的，然而这种方法会使利用估算光流编码的帧的指标有所下降，这个方法对光流估算要求比较高。本文利用高质量帧辅助编解码低质量帧，且进行后增强，也许会对编码过程中的错误传播抑制有用。