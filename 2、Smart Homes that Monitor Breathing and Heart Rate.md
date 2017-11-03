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

