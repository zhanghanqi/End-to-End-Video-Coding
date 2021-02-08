# Learning for Video Compression with Recurrent Auto-Encoder and Recurrent Probability2021

> [代码](https://github.com/RenYang-home/RLVC)

> 创新点：

现有的视频压缩方法只能压缩**少量**的参考帧，本文提出基于**递归自动编码器**（RAE）和**递归概率模型**（RPM）的递归学习视频压缩（RLVC）方法。RAE在编码器和解码器中采用循环单元，大范围帧中的时间信息可用于生成潜在表示和重构压缩输出。此外，所提出的RPM网络根据先前潜在表示的分布情况，反复估计潜在表示的**概率质量函数**（PMF）。由于连续帧之间的相关性，**条件交叉熵**可以低于独立交叉熵，从而降低比特率。

> 算法框架


 <img src="https://i.imgur.com/3W5hYTS.png" >

> RAE

<table>
    <tr>
        <td >
            <center>
                <img src="https://i.imgur.com/8A3arzt.png"  >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图1 RAE</b>
                </div>
    			<br>
            </center>
        </td>
        <td >
            <center>
                <img src="https://i.imgur.com/KNnXopS.png"  >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图2 RAE网络结构</b>
                </div>
    			<br>
            </center>
        </td>
    </tr>
</table>


> RPM

<img src="https://i.imgur.com/7d9xWSs.png" >

> 实验

本文提出的时间条件交叉熵小于独立交叉熵。
$$
E_{y_t\sim p_t}[-log_2 q_t (y_t |y_1,...,y_{t-1})]<E_{y_t \sim p}[-log_2 q(y_t)]
$$
本文提出的时间条件交叉熵小于空间条件交叉熵。
$$
E_{y_t\sim p_t}[-log_2 q_t (y_t |y_1,...,y_{t-1})]<E_{y_t \sim p_y}[-log_2 q_{y|z}(y_t|z_t)]+E_{z_t\sim p_z}[-log_2q_z(z_t)]
$$


> 结论

未来工作可以通过使用递归网络来进一步改进传统的编解码器，例如HEVC。在本文中，该方法的递归框架仍然依赖于扭曲操作和运动补偿来减少时间冗余。消除对基于光流的运动检测的依赖，学习一个完全循环的网络或采用一种基于注意机制的学习视频压缩是一个很有前途的未来工作。