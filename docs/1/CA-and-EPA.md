# Content Adaptive and Error Propagation Aware Deep Video Compression - ECCV2020



> 创新点：

同样出自DVC的研究团队，主要为了解决基于学习的视频编解码的**错误传播**和**视频内容自适应**问题。错误传播的问题通过在训练阶段考虑连续多帧的压缩来解决，是一个训练策略的改进；本文提出的内容自适应方案，可以根据视频内容在线更新编码器，区别于传统的手工编码模式。

<table>
    <tr>
        <td >
            <center>
                <img src="https://i.imgur.com/iYtJBaQ.png" >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图1 错误传播</b>
                </div>
    			<br>
            </center>
        </td>
        <td >
            <center>
                <img src="https://i.imgur.com/FbMCaE8.png" >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图2 不能自适应更改编码器设置</b>
                </div>
    			<br>
            </center>
        </td>
    </tr>
</table>

- 图一 错误传播：由于帧间预测的原因，当前帧编解码依赖前一个解码帧，在一个GOP内，随着时间的推移，误差会逐渐累计，传统算法和基于学习的方法都会遇到错误扩散的问题。
- 图二 内容自适应：传统方法为了能够达到最佳压缩，在图像平滑区域采用大的块，在内容复杂（纹理丰富）的区域采用较小的块；基于学习的方法，在训练阶段通过优化率失真（rate-distortion optimization -RDO）对网络进行指导，然而训练样本与测试样本存在domain gap，训练得到的参数可能对测试集并不是最优的。

> 框架对比

<table>
    <tr>
        <td >
            <center>
                <img src="https://i.imgur.com/ymeN2w8.png" >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图1 DVC框架</b>
                </div>
    			<br>
            </center>
        </td>
        <td >
            <center>
                <img src="https://i.imgur.com/HVAxqqo.png" >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图2 内容自适应和错误传播感知方法 </b>
                </div>
    			<br>
            </center>
        </td>
    </tr>
</table>

> 错误传播

**DVC问题：忽略重构${\hat {x}}_t$时存在的重构误差，对重构$x_{t+1}$由内在影响，从而导致误差扩散
解决方案：针对错误传播问题，在训练阶段对率失真损失进行改进**

$$
L^T=\frac{1}{T}\displaystyle\sum^{T}_{t=1}L_t=\frac{1}{T}\displaystyle\sum^{T}_{t=1}\{\lambda d(x_t,\hat{x}_t)+[H(\hat{y}_t)+H(\hat{m}_t)]\}
$$

> 内容自适应

**DVC问题：编码器参数固定，不依赖当前帧$ x_t$，势必会影响推理阶段的编解码性能
解决方案：编码端的CNN参数会根据帧的内容进行在线更新，保持解码端参数不变**

<img src="https://i.imgur.com/tO6KloC.png" >

本文最大迭代次数设置为10。这个算法就是固定解码器的参数，然后编码器的参数根据实际内容（测试集）进行更新。这样如果参数量比较大，进行10次前传和反向传播+参数更新，势必会增加很多编码端的负担，文献后面也提到，编码端的速度慢了5倍。在线更新策略前后残差编码器参数在线更新前后的输出结果比对，光流结果比对，熵编码的结果比对。

> 训练策略

训练了四个码率的模型，即率失真公式中的$\lambda$采用256,512, 1024, 2048。主帧采用经典的Balle的算法。DVC和主帧模型都还没有**码率控制版本**，即一个模型通过调参进行码率控制，目前已经有部分文献开始研究图片和视频编解码的码率控制问题，在实践中也复现了一些码率控制的文献，但是都达不到单码率版本的编解码效果。

> 结论

提出的错误传播感知策略可以有效缓解帧间预测的错误传播问题，编码端的在线更新策略可以自适应视频内容。本文提出的方法简单而有效。改进后在两个指标上全面超越了H265算法，在一篇文献中看到，H266算法比H265算法编解码性能提高了31.36%，诚然DVC已经比较优秀，但是可以看到，离H266还有很长的路要走,应该要引入其它模块或对DVC架构的策略进行调整。