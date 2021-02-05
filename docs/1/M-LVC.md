# Multiple Frames Prediction for Learned Video Compression
> [代码](https://github.com/JianpingLin/M-LVC_CVPR2020)

> 创新点：

DVC的编解码框架类似于混合编解码框架，减少MV和残差的码率是优化的关键，DVC框架的P帧编码是借助**前一个**解码帧进行运动估计、运动补偿、残差编解码等相关操作，而M-LVC是借助**前面多帧**进行这些操作，理论上是可以提升DVC的编解码性能，可以减缓错误传播的速度。另外一个改进不直接编解码光流，而是对预测光流与原始**光流的残差**进行编解码。
<table>
    <tr>
        <td >
            <center>
                <img src="https://i.imgur.com/uBoBdBA.png" >			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图1 对DVC增加四个蓝色模块</b>
                </div>
    			<br>
            </center>
        </td>
    </tr>
</table>

>MAMVP-Net,多尺度对齐运动预测网络


<table>
    <tr>
        <td >
            <center>
                <img src="https://i.imgur.com/ffbluUD.png" >			
                </center>
        </td>
    </tr>
</table>

- 步骤一
> 采用先前多个重构的MV对当前MV进行预测，上面网络图中采用先前三个重构MV进行预测。首先，对先前每个MV进行金字塔特征提取，见图（a）。
$$
\{f^{l}_{\hat{v}_{t-i}}|l=0,1,2,3\}=H_{mf}({\hat{v}_{t-i}}),i=1,2,3
$$

- 步骤二
> 考虑到先前重构MV有错误，对抽取的金字塔进行特征扭曲。
$$
f^{l,w}_{{\hat {v}}_{t-3}}=Warp(f^{l}_{{\hat {v}}_{t-3}},{\hat{v}^{l}_{t-1}})+Warp({\hat{v}^{l}_{t-2}},{\hat{v}^{l}_{t-1}})
$$
$$
f^{l,w}_{{\hat {v}}_{t-2}}=Warp(f^{l}_{{\hat {v}}_{t-2}},{\hat{v}^{l}_{t-1}}),l=0,1,2,3
$$

-  步骤三
> 利用金字塔网络从粗到细预测当前MV，见图（b）。

>MMC-Net,多参考运动补偿网络

<img src="https://i.imgur.com/pEnNRz1.png" >			

>Residual Refine-Net,残差优化网络

<img src="https://i.imgur.com/Mci2mg8.png" height=“40” weight="50" >

>光流优化网络，降低量化引起的压缩错误

<img src="https://i.imgur.com/7MbKXp7.png" height=“40” weight="50" >

> 训练策略
>
>> 率失真损失函数

$$
J=D+\lambda R=d(x_t,{\hat {x}_t})+\lambda (R_{mvd}+R_{res})
$$

>> Step-by-Step训练步骤

>>作者原来是除了光流网络（FlowNet2.0，初始化采用原作者参数）外其他网络一起联合训练，发现码率严重不均衡：残差码率很大，而光流码率很小，于是采用分步训练。

DVC训练策略：
- 先固定MV参数，不引入光流编解码和光流编码熵估计网络，先联合训练运动补偿（原始光流进行运动补偿）和残差编解码、残差编码熵估计网络；
- 然后再加入光流编解码和光流编码熵估计；
- 除了光流估计网络，所有网络联合训练并迭代一些epoch后，最后放开光流网络参数固定限制，所有网络联合训练。残差熵估计和光流编码熵估计网络也可以先不进行联合训练，等其他网络收敛到一定程度后，再加入熵估计网络联合训练也可以。

> 实验
>>
- 在UVG、HEVC Class B、HEVC Class D三个数据集上，PSNR和MS-SSIM指标均有显著提高。
- 参考帧数量的探讨
- 消融实验
- 编解码时间

> 结论

多参考帧确实可以去掉更多的时间冗余。