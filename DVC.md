# An End-to-end Deep Video Compression Framework

>代码:[Tensorflow开源实现](https://github.com/RenYang-home/OpenDVC)  <a href="https://github.com/GuoLusjtu/DVC" target="_blank">原作实现</a>

<table>
    <tr>
        <td >
            <center>
                <img src="https://i.imgur.com/r9n9hZ5.png" height="400"weight="45%">			</center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图1 传统编码器</b>
                </div>
    			<br>
            </center>
        </td>
        <td >
            <center>
                <img src="https://i.imgur.com/DizXzpl.png" height="400"weight="45%">			</center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图2DVC</b>
                </div>
    			<br>
            </center>
        </td>
    </tr>
</table>

> 区别一：将 Motion Estimation based Block 更改为Motion Estimation based Optical Flow ，采用CNN估算光流，通过运动编码-解码网络对运动信息进行压缩
>

运动估计方式|优点|缺点
:------------:|:--:|:--:
Block|更容易压缩|容易产生块效应、伪影
Optical flow| 不会产生块效应 |不易压缩，传输比特量大

>
> 区别二：将所有Transform更改为Encoder-Decoder Net
>
> 区别三：将Entropy Coding更改为Bit Rate Estimation Net网络节省训练时间
>
> 

>流程对比

步骤|传统编解码器|DVC
:------------:|:--:|:--:
Step1运动估计|当前帧$x_t$与前一个重构帧${\hat{x}}_{t-1}$进行运动估计，运动信息为$v_t$|当前帧$x_t$与前一个重构帧${\hat{x}}_{t-1}$进行运动估计，运动信息为$v_t$。采用CNN估算光流，然后采用运动编码-解码网络对运动信息进行压缩，量化后的运动信息记为${\hat{m}}_t$。$v_t$经过运动解码网络解码出运动信息为${\hat{v}}_t$。
Step2运动补偿| 根据$v_t$，将前一个重构帧${\hat{x}}_{t-1}$的对应像素进行移动获取当前预测帧${\bar{x}}_t$,预测残差$r_t=x_t-{\bar{x}_t}$ |基于${\hat{v}}_t$和前一个重构帧${\hat{x}}_{t-1}$，利用深度网络进行运动补偿得到${\bar{x}}_t$。首先，利用$v_t$对前一重构帧进行warped操作，得到$w({\hat{x}}_{t-1},{\hat{v}}_t)$,$w({\hat{x}}_{t-1},{\hat{v}}_t)$、${\hat{x}}_{t-1}$、${\hat{v}}_t$作为输入送到CNN网络改善预测帧${\bar{x}}_t$
Step3变换与量化| 预测残差$r_t$经过量化被量化成${\hat{y}_t}$。在量化前需要经过线性变化（比如DCT） |预测残差$r_t$经过残差编码网络编码成$y_t$，然后量化成${\hat{y}_t}$。网络采用文献Variational image compression with a scale hyperprior的网络
Step4反变化| 对Step3中${\hat{y}_t}$，进行反变换，获取重构残差${\hat{r}}_t$ |${\hat{y}_t}$经过残差解码网络得到${\hat{r}}_t$。
Step5熵编码| Step1中的$v_t$和Step3中${\hat{y}_t}$经过熵编码编译成二进制，并且发送给编码器 |Step1中的${\hat{m}}_t$和Step3中${\hat{y}_t}$进行熵编码
Step6视频帧重构| Step2中的${\bar{x}}_t$和Step4中$r_t$“相加”获取重构帧${\hat{x}}_{t}$ |Step2中的${\bar{x}}_t$和Step4中${\hat{r}}_t$“相加”获取重构帧${\hat{x}}_{t}$

>网络结构
<table>
    <tr>
        <td >
            <center>
                <img src="https://i.imgur.com/R5bOB4V.png" height="400"weight="400">			
                </center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图1 运动编解码网络</b>
                </div>
    			<br>
            </center>
        </td>
        <td >
            <center>
                <img src="https://i.imgur.com/9YqUKpv.png" height="400"weight="600">			</center>
        	<center>
                <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    				display: inline-block;
    				color: #999;
    				padding: 2px;">
                    <b>图2 运动补偿网络</b>
                </div>
    			<br>
            </center>
        </td>
    </tr>
</table>

> 其他优化方式

损失函数|$D+\lambda{R}=d(x_t,{\hat{x}}_{t})+\lambda{}(H({\hat{m}}_t)+H({\hat{y}_t}))$
:------------:|:--:
**量化**|训练阶段直接加一个噪声，推理阶段直接四舍五入取整
**比特率估计**| 比特率衡量采用文献Variational image compression with a scale hyperprior的网络 
**缓冲前帧**| 为了减少一组帧占用过多的GPU，文献采用一个缓冲机制，在线实时更新缓冲器里的帧 

> 实验

  本文结果略好于H264和H265，在UVG测试集上还要好于Wu_ECCV2018(Video Commpression through Image Interpolation).




