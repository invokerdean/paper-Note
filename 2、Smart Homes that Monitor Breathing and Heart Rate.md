# Smart Homes that Monitor Breathing and Heart Rate

## abstract
whether it is possible for smart environments to monitor our vital signs remotely, without instrumenting our bodies
——Vital Radio

**原理**：wireless signals are affected by motion in the environment, including chest movements due to inhaling and exhaling and skin vibrations due to heartbeats. 

**准确度**：median accuracy of 99%, even when users are 8 meters away from the device, or in a different room

**并发性**：it can monitor the vital signs of multiple people simultaneously
## introduction
**Vital Radio可以解决的问题**：“Do my breathing and heart rates reflect a healthy lifestyle?”“Does my child breathe normally during
sleep?” or “Does my elderly parent experience irregular heartbeats?”

**动机**：typical technologies for tracking vital signals require body contact, and most of them are intrusive.they are inconvenient,and sometimes undesirable for some population

**目标**：remotely,without contacting

**现有的非接触式**：more appropriate for controlled settings,fail in the presence of multiple users or extraneous motion,accurate only when close 

**Vital Radio**:works correctly in the presence of multiple users in the environment and can track the vital signs of the present users simultaneously

**工作原理**:it transmits a low-power wireless signal and measures the time it takes for the signal to reflect back to the device.The reflection time depends on the distance of the reflector to the device, and changes as the reflector moves.

**其中一个核心特点**：multiple people,without requiring the users to lie still;*挑战*：any motion in the environment can affect the
wireless signal;*解决方法*：first localizes each user,then zooms in on the signal reflected from each user,analyzes variations in his reflection to extract his breathing and heart rate ;By isolating a user’s reflection, Vital-Radio also eliminates other sources of interference including noise or extraneous motion in the environment

**验证性能实验**:14 subjects; use FDA-approved breathing and heart rate monitors for baselines;compare the output of Vital-Radio with the ground
truth from the FDA-approved baselines;Over more than 200 two-minute experiments;In an area that spans 8 m×5 m

**实验结果**：Vital-Radio’s median accuracy for breathing is 99.3% (error of 0.09 breath/minute) and for heart rate is 98.5% (0.95 beat/minute) when the person is 1 m away from the device.;The accuracy decreases to 98.7% (error of 0.15 breath/minute) and 98.3% (1.1 beat/minute) when the person is 8 m away from the device;In an area that spans 8 m×5 m, Vital-Radio can monitor the
vital signs of up to three individuals with the same accuracy as for one person.

**基于活动的实验**：accurately measure while users are typing on their computer or using their cell phones.accurately tracks the change
in breathing and heart rates after exercising.

## conclusion
Vital Radio doesn’t require users to to be aware of its presence, and hence doesn’t interfere with user experience.new interface and interaction capabilities

**应用**:enable environments to adapt the music or lighting by sensing the user’s vital signs and inferring his mood;a user walking up to a Vital-Radioenabled kiosk in an unfamiliar location (such as an airport) might receive customized assistance based on his stress level.can also impact a wide array of areas in HCI including quantified self, smart homes, elderly care, personal health and well-being, and mobile emotional sensing.
## procedure

**操作原理**:use the reflection time to compute the distance from the device to the human body.Vital-Radio captures these minute changes in distance and uses them to extract the user’s vital signs.

**处理多反射问题**:
1.Isolate reflections from different users and eliminate reflections off furniture and static objects.
2.identify the signal variations that are due to breathing and heartbeats, and separate them from variations due to body or limb motion.
3.Analyze signal variations to extract breathing and heart rates.

#### Step 1: Isolate Reflections from Different Users and Eliminate Reflections off Furniture and Walls
**技术手段**:FMCW (Frequency Modulated Carrier Waves);it enables separating the reflections from different objects into buckets based on their reflection times

**过去做法**:使用FMCW来感测从不同距离到达的功率量,从而定位用户。
**本文做法**:Vital-Radio使用FMCW技术作为滤波器，使用它来隔离从环境中的不同距离到达的反射信号分为不同的bucket，然后进行分析每个桶中的信号以提取生命体征（下面的步骤2）。
**具体实现**:buckets:8cm;意义在于，一是使得相距几英尺的物体分为不同的bucket（理论上越小越好？）二是隔离由心跳和呼吸引起的肢体运动和胸腔运动，例如用户移动脚不会影响到心跳和呼吸的提取。分bucket后，消除由静态物体引起的反射。
**特殊情况**:若有两个object处于相同的distance，解决方法是采用多个设备，使得这两个物体距离其他设备距离不同。

#### Step 2: Identifying Reflections Involving Breathing and Heart Rate
**目标**:identify whether the user in bucket is quasi-static and his motion is dominated by his vital signs, or whether he is walking around or moving a limb.(滤除无关运动)

**公式原理**:φ(t) = 2πd(t)/λ，通过测量反射信号的相位变化，可以识别出由于吸气、呼气、心跳引起的d(t)的变化，其中，d(t)是从设备到反射物再回到设备的travel dis.φ(t)是反射波相位.λ是反射波波长.本文实现的波长为4.5cm.由此公式，在测得的相位时间函数中，波峰代表呼气，波谷代表吸气，而放大该信号，可以观察到呼吸运动之上调制的心跳。允许Vital-Radio从信号反射中提取心率的生理现象是心冲击波（BCG）。 BCG是指由于心室泵活性而引起的与心跳同步的身体运动。过去的工作记录了BCG从头部，躯干，臀部等方面的抖动。 周期性的抖动导致无线信号的周期性变化，使我们能够捕捉到心率。 这些运动转化为无线反射中呼吸运动顶部的较小波动。（注：周期性与用户对设备朝向无关）

**肢体运动干扰问题**:利用fact(由于生命体征引起的运动是周期性的，而身体或肢体运动是非周期性的)从而识别出该interval是否是肢体运动引起的若是，则抛弃该interval.具体实现时，将vital radio在time window（本文为30s）上运行，评估每个time window的周期性，若超过阈值，则认为是由vital sign主导的波形，否则丢弃，评估的方法是检测波形的傅里叶变换或者FFT的清晰度，因此，我们在每个窗口上执行FFT，选择FFT的峰值频率，并确定峰值是否足够高于其余频率的平均功率（本文实现中至少为5倍以上）。这个度量标准允许我们maintain用户不会执行较大肢体移动的时隙，例如在笔记本上打字或者检查手机，由身体连接的动作太弱且非周期，所以它们在FFT的输出处被稀释。相反，生命体征的周期性运动在FFT操作中被强制执行，因此这样的准静态情景的时隙被保留。



#### Step 3: Extracting Breathing and Heart Rate
###### Breathing Rate Extraction

**初步估计**将图3的波形FFT后，其峰值即呼吸频率的初步估计

Specifically, the frequency resolution of an FFT is 1/window size. For a window size of 30 seconds, the resolution of our breath rate estimate is ≈ 0.033Hz，但单纯地增大窗口尺寸，尽管能提供更大的分辨率，但其追踪呼吸频率变化的能力会减弱。

**信号处理特性**:如果信号包含一个单一的主频率，则可以通过对复数时频分量的相位进行线性回归来精确测量该频率。
 具体实现时，我们过滤FFT的输出，只保留峰值和其两个相邻的bin，使得可以消除无关和非周期性运动引起的噪音。再进行一个逆FFT，得到复数时域信号s0(t)，其相位将是线性的，并且其斜率将对应于呼吸频率。Estimate = 60 ×slope{∠s0(t)}/2π,其中60这个因子将单位由Hz转换为breath/minute
###### Heart Rate Extraction

**问题挑战**:呼吸信号比心跳信号强几个数量级，这导致了FFT中的经典问题，一个给定频率的强信号泄漏到其他频率的信号中（即，在FFT的输出处泄漏到附近的bin中）并且可能在邻近频率处掩蔽较弱的信号。

**减轻方法**:对频率在40-200次每分钟的信号进行滤波，这将处于8-16次每分钟的呼吸信号和高频噪声过滤了出来。我们选取输出的本地最大值为心跳频率的粗估计，注意，绝对最大值通常是由泄露引起的位于滤波后得到的第一个bin，因此弃用。而后类似呼吸的做法，保留峰值和相邻的bin，再逆FFT，得到的心跳频率为66.7 beats/minute,而脉冲血氧计测得为66.5。

为了计算心率，我们只使用10秒的FFT窗口。 这个窗口足够长，以捕捉心跳的周期性，但是足够短以快速反应心率的增加/减少。 还要注意，FFT是在重叠的窗口上移动了30ms，因此每30ms提供一个新的估计值。
## implementation

**组件**:
1、hardware方面，我们复制了由过去的无线定位工作设计的最先进的FMCW收音机[6]。 该器件产生一个信号，每2.5毫秒从5.46 GHz扫描到7.25 GHz，传输低于mW的功率。 这些参数在[6]中选择，使得传输系统符合针对消费电子产品的FCC规定。FMCW无线电通过以太网连接到计算机。 接收到的信号被采样（数字化）并通过以太网传输到计算机进行实时处理。
2、software方面，我们用C ++实现前几节中介绍的信号处理算法。 代码实时运行，在屏幕上绘制呼吸和心率作为时间的函数，同时将它们记录到文件中。 该代码在移位的重叠FFT窗口上运行，并每隔30ms产生一个新的估计值。 输出还显示用户运动 - 即，每隔30ms窗口显示代码标签，以显示用户是准静态还是执行主要动作。

**参与者**:
14名参与者（3名女性）。 这些参与者在21至55岁（μ= 31.4）之间，体重在52至95公斤（μ= 78.3）之间，身高在164至187厘米之间（μ= 175）。 在实验过程中，受试者穿着日常服装，包括衬衫，T恤衫，连帽衫和夹克等不同的面料。

**Ground Truth**:we compare its output against the Alice PDx,an FDA approved device for monitoring breathing and heart rate.

**实验环境**:一个标准的办公大楼里进行实验。 内墙是标准的双层干墙，上面有金属框架支撑。 评估环境包含办公家具，包括书桌，椅子，沙发和电脑。Vital-Radio的天线放在桌子上,高出地面约3英尺。用户坐在离这些天线一定距离的地方，穿着Alice PDx的胸带和脉搏血氧仪来获取真实度量

**Core Experiment: Accuracy Versus Distance**:我们希望验证Vital-Radio能够在距离设备不同距离的情况下监测受试者的呼吸和心率。我们将设备放置在一个大房间的角落，其平面图如图8所示。设备的天线指向房间的中心，以确保它们捕捉房间内的运动。 我们要求拍摄对象坐在距离设备1米至8米距离的标记位置的椅子上。 在每个实验中，受试者坐在面向Vital-Radio的天线的椅子上，戴着Alice PDx。

我们总共进行了112次实验，要求14名受试者中的每一位坐在1米至8米的标记位置。每次实验持续2分钟，用户面对每个位置的设备。 我们使用Vital-Radio实时提取呼吸和心率，并使用AlicePDx记录这些生命体征。 在每两分钟的实验期间，Vital-Radio每30毫秒输出一次生命体征估计值，在所有实验和所有位置上总共测量448,000次。
使用AlicePDx进行的Ground Truth测量，受试者的呼吸速率范围为5至23次/分钟，而其心率从53次/分钟至115次/分钟不等。 这些比率跨越了成人呼吸和心率范围[41,31] 

**多场景下的准确率**:为了验证Vital-Radio在被摄对象不直接面对设备的情况下也能正确运行，我们要求每个对象坐在离Vital-Radio 4米远的地方，以四个不同的方向进行实验：对象面向设备，对象背对着设备，对象向左或向右（垂直） 到设备。

接下来，我们验证Vital-Radio不需要用户沿着面向天线的直线。 具体来说，我们把天线放在房间的中央，要求用户坐在离天线4米远的地方，相对于天线的指向方向，角度在-90°到+ 90°之间。 我们用不同的角度进行20个一分钟的实验。 结果表明，Vital-Radio只要相对于天线的指向方向在-75°和+ 75°之间的角度就可以捕获用户的生命体征。 具体而言，当用户在相对于天线的直线上时，中值精度高于98％，而在远边（即±75°）减小到96％。(?)

为了测试Vital-Radio即使在不同房间时也能测量用户生命体征的能力，我们进行了一系列穿墙式实验，将设备放置在不同于我们的对象的房间内。 具体来说，我们使用图8中的实验设置。设备保存在较大的房间内，而主体坐在墙后的相邻房间内。 对象面对设备，距离它约4米。在所有实验中，呼吸和心率的平均准确率分别为99.2％和90.1％。 这些结果表明，无论是否存在壁（相同距离4米），呼吸率几乎保持不变。 然而，由于心脏壁信号明显衰减（这已经是一个非常微小的信号），心率中位数的准确性下降，因此降低了我们的信噪比。 即使在这种穿墙情况下，心率准确度仍然保持在90％左右。

我们希望评估Vital-Radio的多用户生命体征监测的准确性。 因此，我们进行对照实验，在这里我们要求三个用户坐在椅子上，如图8中的2米，4米和6米，每个都在距设备各自的距离处，并输出每个的生命体征; 但是，基线（AlicePDx）只能在任何时间点监视单个用户。 因此，为了评估准确性，我们首先将基线连接到第一个用户，并将其输出与在那个距离和那个时刻的用户的VitalRadio的输出进行比较。 然后，我们将基线连续地移动到其余的用户。我们对不同的受试者进行了20次实验，并绘制了图12中的精确度。图中显示，Vital-Radio的呼吸和心率监测准确率均为98％左右。 

基于活动的实验：在每个实验中，我们要求对象坐在4米处设备，并自然使用他/她的手机或笔记本电脑。 每个实验持续5分钟; 用户在最后报告他/她用笔记本电脑或手机进行的活动。在我们的受试者使用手机的整个实验过程中，呼吸中位数和心率的准确率分别为99.4％和98.9％。 当我们的受试者使用笔记本电脑时，这些精确度略降到99.3％和98.7％。 由于使用笔记本电脑通常会比使用电话动作稍微大一些，导致Vital-Radio的准确度略有下降

#### limitations

1、用户之间的最小分隔：Vital-Radio使用FMCW之前分离来自不同用户的反射提取每个用户的生命。 对于理想的点反射器，FMCW可以将两个物体的反射分开至少是C / 2B [6]，其中B是带宽和C是光的速度。 对于Vital-Radio来说，这意味着一个理论最小间隔8厘米。 但是，因为一个人不是一个点反射器，我们的实验表明高精度需要1-2米的距离。

2、监测范围：由于Vital-Radio是一个无线系统，它需要一个最小的信噪比（SNR）来提取来自噪声的信号，并且这个SNR限制了它的信号范围和准确性。 具体来说，最大距离在Vital-Radio检测用户是8m。 这是因为SNR随着用户距设备的距离而下降。

3、准静态要求：我们仅针对准静态用户（例如打字，看电视）来衡量生命迹象。 否则由全身运动引起的信号变化会压倒由于生命体征引起的小的变化，并且阻止Vital-Radio捕获微小的运动

4、非人类运动：Vital-Radio使用FMCW进行分离来自空间不同物体的反射，可以分开各种移动物体的反射。 然后分析每个移动物体的反射来检测呼吸。 由于呼吸的周期性远低于风扇，装置永远不会把风扇混淆为人。 即使混淆了风扇，也不会影响人的生命体征信号，因为他们的信号与风扇用FMCW分开了, 但是，装置仍然可以识别宠物的存在，并把它当作人类，输出呼吸和心率。
