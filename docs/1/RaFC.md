# Improving Deep Video Compression by Resolution-adaptive Flow Coding2020.09
> 创新点：

DVC通过运动矢量（MV）编码器来压缩**像素级**的光流图。在这项工作中，本文提出了一种称为Resolution-adaptive Flow Coding (RaFC)的新框架来有效地压缩全局和局部的流图，其对MV编码器的输入流图和输出运动特征使用**多分辨率**表示。**RaFC-frame**自动决定每个视频帧的最佳流图分辨率，有益于全局处理复杂或简单的运动模式。**RaFC-block**为每个局部运动特征块选择最佳分辨率，有益于局部处理不同类型的运动模式。

> 算法框架

将DVC的运动估计网络、运动编码网络进行了改进

 <img src="https://i.imgur.com/lJMwHkI.png" >
 
> RaFC-frame

给定解码帧缓冲器中的输入帧$X_t$及其对应的参考帧${\hat{X}}_{t-1}$，利用ME生成**多尺度流图**。结构采用Spynet中已有的金字塔结构，生成了两个分辨率分别为$W\times H$和$\frac{W}{2}\times \frac{H}{2}$的流程图$V^1_t$和$V^2_t$,目标是从当前帧的多尺度光流图中选择最佳分辨率，以便全局处理复杂或简单的运动模式。

> RaFC-block

借鉴H.264、H.265中的不同的块大小用于运动估计，设计一种有效的多尺度运动特征来处理不同类型的运动模式。

<table>
    <tr>
        <td >
            <center>
                <img src="https://i.imgur.com/X1aD0fd.png" >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图1 RaFC-block</b>
                </div>
    			<br>
            </center>
        </td>
        <td >
            <center>
                <img src="https://i.imgur.com/8tJOWl0.png" >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图2 Mask</b>
                </div>
    			<br>
            </center>
        </td>
    </tr>
</table>

给定一个分辨率的光流图$V_t$，光流图$V_t$来生成多尺度运动特征$m^1_t$和$m^2_t$。RaFC-block将基于RDO技术为重建帧中的每个分块选择最佳的运动特征分辨率。步骤如下：

- 在A、B、C、D中选择$m^1_t$和$m^2_t$中较小的RD值，来确定每个块将使用哪个运动特征表示
- 根据上步得到的最优选择来进行运动特征重组

> 实验

移动对象边界周围的区域优先选择小尺寸块，平滑区域优先选择大尺寸块。

<img src="https://i.imgur.com/ONVzRr0.png" >

> 结论

在未来的工作中，本文将使用所提出的框架来编码残差信息，研究更有效的块分割策略。
