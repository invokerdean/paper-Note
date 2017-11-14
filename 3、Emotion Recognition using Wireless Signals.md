# Emotion Recognition using Wireless Signals
## supplement
心率变异性是指逐次心搏间期的微小差异，它产生于自主神经系统对心脏窦房结的调制，使得心搏间期一般存在几十毫秒的差异和波动。
## abbr.
- ECG: Electrocardiograph 心电图
- IBI：inter-beat-intervals 心搏间期
- SINR: signal-to-interference-and-noise ratio
- IRB: Institutional Review Board 机构审查委员会,美国每所研究型大学里都必须设立IRB。
## abstract
本文提出一种新的技术从人体反射的RF信号来推断情绪，EQ-Radio发射信号，并分析其对人体的反射信号来识别情绪。EQ-Radio的原理是提出一种新的算法，可以从无线信号中提取单个心跳，其准确度与身上的心电监护仪相当。提取出的心跳随后用于计算一些情绪相关的特征，并送入Machine Learning情绪分类器。最终可以证明EQ-Radio准确率和最先进的情绪识别系统（该系统需要与ECG挂钩）相当。
## introduction
**动机**:建立感知我们情绪的机器,使智能家居对我们的情绪作出反应，并相应地调整照明或音乐； 电影制作者将有更好的工具来评估用户体验。 广告商会立即了解客户的反应。 计算机会自动检测抑郁症，焦虑症和双相情感障碍的症状，及早回应这些情况。 更广泛地说，机器不再局限于明确的命令，并且可以以人与人交互的方式与人交互。

**现有方法**:
1. 方法一依赖于视听提示，如图像和音频剪辑，但只能利用情绪的外在表达，未必是内心的真实情感，且不同人表达情绪的方式不同使问题复杂化；
2. 方法二要求人穿戴生理传感器，如ECG monitor，通过监测随着我们的情绪状态（例如心跳）而改变的生理信号来识别情绪。它使用体内传感器（例如ECG监视器）来测量这些信号并将其变化与喜悦，愤怒等相关联。这种方法与人的内心感受更相关，因为它触及自主神经系统和心律。然而，使用人体感应器麻烦，可能会干扰用户的活动和情绪，使得这种方法不适合经常使用

我们的设计使用射频信号来感知情绪。 特别地，射频信号会反射人体并受到身体运动的调制。射频反射可以测量呼吸和心跳的频率，然而，心脏信号的周期性（即其运行平均值）与情绪识别几乎没有关系，为了识别情绪，我们需要测量每个心跳长度(beats length)的微小变化。(调研一下引用源)

**从射频信号中提取单个心跳带来的挑战**:
1. 反射的RF信号由呼吸和心跳调制。呼吸的影响通常比心跳大几个数量级，并且往往掩盖单个心跳(individual beats)。过去为了从心率中剥离呼吸，在频域中运行多秒，从而放弃了测量心跳变化的能力。
2. RF信号中的心跳缺少表征ECG信号的尖峰，使得难以准确识别心跳边界。
3. 心搏间期的差别只有几十毫秒。因此，individual beats必须在几毫秒内分段(大概是指，以几毫秒的误差分段)。在没有用以识别心跳开始或结束的尖峰的情况下，很难达到这样的精度。

EQ-Radio’s key enabler is **a new algorithm** for extracting individual heartbeats and their differences from RF signals.

*step1*:first mitigates the impact of breathing.
原理：呼吸加速度比心跳小，这是因为呼吸通常是缓慢而稳定的，而心跳涉及肌肉的快速收缩（这发生在局部时间）。 因此，EQ-Radio对RF信号的加速度进行操作，以抑制呼吸信号并强调心跳。

*step2*:EQ-Radio needs to segment the RF reflection intoindividual heartbeats.
但不同于ECG,RF反射中心跳的形状是未知的，并且依赖于人的身体和相对于设备的具体姿势而变化。因此不能简单地寻找一个已知的形状进行分割，而需要 learn the beat shape 。我们将问题表述为一个联合优化，在两个子问题之间进行迭代：第一个子问题在给定特定分段的情况下学习一个心跳模板，第二个子问题找到与学习模板最相似性的分段。我们两个子问题之间进行反复迭代，直到我们收敛到**最好的心跳模板**和 **与模板最相似性的最优分割**。

*step3*:we note that our segmentation takes into account that beats can shrink and expand and hence vary in beat length.
因此，该算法找到合适的心搏（beats）分割，使连续心搏间期中的心跳信号的在形态学上最相似，同时允许beats信号的灵活变形（收缩或扩大）。

**系统架构**:
1. The first component is an **FMCW radio** that transmits RF signals and receives their reflections. The radio leverages the approach in [7] to zoom in on human reflections and ignore reflections from other objects in the scene.
2. the resulting RF signal is passed to the **beat extraction algorithm**.returns a series of signal segments that correspond to the individual heartbeats.
3. the heartbeats – along with the captured breathing patterns from RF reflections – are passed to an emotion **classification sub-system**, The emotion classification sub-system computes heartbeat-based and respiration-based features recommended in the literature [34, 14, 48] and uses an **SVM classifier** to differentiate among various emotional states.

**evaluate**:conducting user experiments with 30 subjects.依据该领域的文献 [34, 14, 48]来设计实验。要求被试者通过回忆相应的记忆（例如悲伤或愉快的记忆）来唤起特定的情绪。可以使用音乐或照片来辅助唤起。 在每个实验中，被试者报告他/她感受到的情绪，这种情绪持续的时间段。 在实验期间，同时使用EQ-Radio和商用ECG监视器来监视受试者，并且实验会录成视频，然后传递给基于图像的微软情感识别系统。

**结果**:EQ-Radio的情绪识别能力与最先进的基于ECG的系统是不相上下的：如果系统分别对被试者进行训练，情绪分类的准确性在EQ-Radio中为87％，在ECG-based系统中为88.2％。 如果对所有受试者使用同一个分类器，则EQ-Radio中为72.3％，基于ECG的系统为73.2％。对于相同的实验，基于图像的系统的准确度为39.5％。 这是因为当情绪在被试者的脸上不可见时，基于图像的系统表现不佳。

**原因**:EQ-Radio的性能优秀是由于它能准确地从RF信号提取心跳。 具体来说，若对心跳间隔的估计存在40-50毫秒的误差，会使情绪识别精度降低到44％（如我们在§7.3中的图12所示）。 而我们的算法实现3.2毫秒的IBI的平均误差，这比平均beats length的0.4％还小。
## conclusion
本文提出了一种依靠身体反射的无线信号来识别人的情绪的技术。 我们相信这是情绪识别新兴领域的重要一步。 随着人们对无线系统社区利用射频信号进行感知的兴趣越来越高，这项工作将射频感测的范围扩展到情感识别领域。 此外，这项工作为无线情感识别奠定了基础，我们预计随着无线传感技术的发展以及圈内人士在传感过程中采用更先进的机器学习机制，这些系统的准确性将会提高。
我们的算法从RF复原了整个人类的心跳，并且心跳呈现出非常丰富的形态。 我们预计这一结果将为在情绪识别以及在非侵入健康监测诊断的背景下理解心跳形态的研究铺平道路。
## contribution
## procedure
#### EQ-Radio OVERVIEW
EQ-Radio是一种纯粹依靠无线信号的情绪识别系统。 它发射射频信号并捕获人体的反射, 然后分析这些反射来推断人的情绪状态。 根据已知的唤醒价模型将人的情绪状态分为四种基本情绪：anger, sadness, joy, and pleasure (i.e., contentment)。 EQ-Radio的系统架构由三个以流水线方式运行的组件组成：如上所述，一个FMCW无线电，发射射频信号，并获取在人体的反射。 beats提取算法，将捕获的反射作为输入，并返回一系列对应于人员的单个心跳的信号片段。 情绪分类子系统，根据捕获的生理信号计算情绪相关特征，即人的呼吸模式和心跳，并使用这些特征来识别人的情绪状态。
#### CAPTURING THE RF SIGNAL
uses a radar technique called Frequency Modulated Carrier Waves (FMCW)，There is a significant literature on FMCW radios and their use for obtaining an RF signal that is modulated by breathing and heartbeats [7, 11, 49]. 

radio发射低功率信号并测量其反射时间，根据反射时间将不同物体/身体的射频反射分成buckets。 然后消除不随时间变化的静态物体的反射，并放大人体的反射。 它着眼于人处于准静态的时间段。 然后，它观察与travel distance有关的RF波的相位:φ(t) = 2πd(t)/λ.相位变化对应于呼吸引起的胸腔扩张和收缩引起的复合性位移，以及心跳引起的身体振动.相位的包络体现了吸气呼出过程的胸部位移，小凹陷是由于与脉动相关的微小的皮肤振动。 EQ-Radio对这个相位信号进行操作。
#### BEAT EXTRACTION ALGORITHM
一个人的情绪与她/他的心跳间隔的小变化有关; 因此，为了识别情绪，EQ-Radio需要从上述的RF相位信号中提取这些间隔。

提取心跳间隔的主要挑战是反射的RF信号中beats的形态是未知的。具体而言，这些beats会导致反射信号的距离变化，但测量的位移取决于很多因素，包括人体和其相对EQ-Radio天线的确切姿势。（相比之下，ECG信号的心跳的形态具有已知的期望形状，并且使用简单的峰值检测算法可以提取心跳间隔。）由于不知道RF中这些心跳的形态，所以无法确定心跳何时开始结束，因此无法获得每个beat的间隔。实质上，这变成了鸡和蛋的问题：如果我们知道心跳的形态，那将有助于我们分割信号;另一方面，如果我们有反射信号的分割，我们可以用它来恢复人体心跳的形态。

这个问题由于另外两个因素而加剧。首先，反射信号是嘈杂的; 第二，由于呼吸的胸部位移比心跳位移高几个数量级。 换句话说，我们在低信噪比（signal-to-interference-and-noise）机制下运作，其中“干扰”是由于呼吸引起的胸部位移。

为解决这些挑战，EQ-Radio首先处理RF信号以减轻呼吸的干扰。 然后制定并解决一个联合优化问题以恢复心搏间隔。 优化公式既不假设也不依赖于完美的分离效应。

###### Mitigating the Impact of Breathing
预处理步骤的目标是抑制呼吸信号并提高心跳信号的SINR。

RF信号的相位与吸气呼气过程和脉冲效应（pulse effect）的复合位移成正比。由于吸气 - 呼气过程引起的位移比心跳引起的微小振动大几个数量级，所以RF相位信号是 以呼吸为主，但呼吸加速小于心跳。因此，我们可以通过对与加速度（而不是位移）成比例的信号进行操作来抑制呼吸并强调心跳。

根据定义，加速度是位移的二阶导数。 因此，我们可以简单地对RF相位信号的二阶导数进行操作。 由于没有RF信号的解析表达式，所以必须使用数值方法来计算二阶导数。 我们使用下面的二阶微分器，因为它对噪声是robust的：

f′′0 =[4f0 + (f1 + f−1) − 2(f2 + f−2) − (f3 + f−3) ]/16h2, (1)where f ′′ 0 refers to the second derivative at a particular sample, fi refers to the value of the time series i samples away, and h is the time interval between consecutive samples.

在图3中，我们示出了具有相应加速度信号的示例RF相位信号。 图中显示，射频信号中呼吸比心跳更明显。 而在加速度信号中，存在对应于每个心跳周期的周期性模式（pattern），并且呼吸效应可以忽略不计。注意，尽管此时我们可以在加速度中观察心跳信号的周期性，但是由于信号噪声并且缺乏清晰的特征，仍然难以勾画出心搏间期的边界。

###### Heartbeat Segmentation
接下来，EQ-Radio需要将加速度信号分成单独的心跳。关键挑战是不知道心跳的形态来引导这个分割过程。 我们制定了一个优化问题，jointly恢复心跳的形态和分割。

这种优化的直觉是，连续的心跳应具有相同的形态; 因此，虽然由于beats length不同，心跳可能会拉伸或压缩，但它们应具有相同的整体形状（overall shape）， 这意味着我们需要找到一个分割，能最大限度地减少所产生的beats之间的形状差异，同时兼顾未知beats形状以及beats可能会压缩或拉伸。 我们的公式不使用贪婪算法寻求局部最优选择，而是在所有可能分割中寻求最优解：
###### Algorithm
略

#### EMOTION CLASSIFICATION
EQ-Radio从射频反射中恢复单个心跳（ individual heartbeats）后，使用心跳序列和呼吸信号来识别人的情绪。 下面描述EQ-Radio所采用的情感模型，及其特征提取和分类方法。
###### a. 2D Emotion Model
EQ-Radio采用一个二维情感模型，其坐标轴是价和兴奋; 该模型是过去文献中对人类情绪进行分类的最普遍的方法。 该模型分为四种基本情绪状态：Sadness（负价和负性唤醒），Anger（负价和正性唤醒），Pleasure（正价和负性唤醒）和Joy（正价和正性唤醒）。(?)
###### b. Feature Extraction
EQ-Radio从心跳序列和呼吸信号中提取特征。 关于从人类心跳中提取emotion-dependent 特征的文献很多[34,48,4]，过去的技术使用一些贴身的传感器。 这些特征可分为时域分析，频域分析，时频分析，Poincar'e图（Poincar´e plot），样本熵 Sample Entropy 和去趋势波动分析Detrend Fluctuation Analysis。 EQ-Radio从IBI序列中提取27个特征（见表1），这些特征是根据[34]中的结果选择的。 [34，4]中详细解释了这些功能。
EQ-Radio还应用了呼吸特征。 为了提取不规则的呼吸，EQ-Radio首先通过低通滤波后的峰值检测识别每个呼吸周期。 由于过去研究呼吸特征的工作推荐了时域特征[48]，EQ-Radio提取了时域特征（见表1第一行）。
###### c. Handling Dependence
对于同一情绪状态，展现生理特征因人而异，即便同一个人的这些特征在不同的日子也可能会有所不同。 这是由多种因素引起的，包括咖啡因的摄入，睡眠和一天的情绪基调。
为了提取独立于用户和独立于时间的“更好的”特征，EQ-Radio包含一个baseline情绪状态：中性（neural）。 这个idea主要是利用生理特征的变化而不是绝对值。 因此，对于每个特征，EQ-Radio通过从减去给定人给定日期的中性状态下计算的对应值，来校准计算出的特征。
###### d. Feature Selection and Classification
如前所述，文献提出了许多将IBI与情绪联系起来的特征。 使用所有这些特征和有限的数据进行训练可能会导致过度拟合。 选择一组与情感最相关的特征不仅可以减少训练所需的数据量，而且能提高对测试数据的分类准确性。
之前有关特征选择的研究[48,34]使用了包装方法(wrapper method)，将特征选择问题当作搜索问题。 然而，由于需要选择的数量是指数级巨大的，包装方法必须使用启发式方法来搜索相关特征的所有可能的子集。 而EQ-Radio使用另一类特征选择机制，即嵌入式方法[26]; 这种方法使我们能够在训练模型的同时学习哪些功能对模型的精确性有最大的贡献。 为此，EQ-Radio使用l1-SVM， [65]在训练SVM分类器时选择相关特征的子集。 表1分别以粗体和斜体显示选定的IBI和呼吸特征。 
## implementation && evaluation
我们描述了EQ-Radio的实现以及在提取个体心跳和识别人类情绪状态方面的经验表现。 所有的实验都得到了我们的IRB的批准。
#### Implementation
我们复制了由过去的无线生命体征监测工作设计的最先进的FMCW无线电[7]。 该器件产生一个信号，每4毫秒从5.46 GHz扫描到7.25 GHz，传输亚毫瓦功率。 参数如[7]中选择，使得传输系统符合FCC电子消费品法规。 FMCW无线电通过以太网连接到计算机。 接收到的信号被采样（数字化）并通过以太网传输到计算机。 EQ-Radio的算法在i7处理器和32GB RAM的Ubuntu 14.04计算机上实现。
#### Evaluation of Heartbeat Extraction
首先，我们想评估EQ-Radio分割算法在从身体反射的RF信号中提取心跳的准确性。
###### Experimental Setup
参与者：我们招募了30名参与者（10名女性）。 我们的subjects是在19〜77岁之间。 在实验过程中，subjects穿着不同面料的日常服装。
实验环境：我们在标准办公大楼的5个不同房间进行实验。 评估环境包含办公家具，包括书桌，椅子，沙发和电脑。 当其他用户在房间中时进行实验。 实验环境的变化和其他用户的存在对结果的影响可以忽略不计，因为§4中描述的FMCW无线电消除了来自静态物体（例如家具）的反射，并隔离了来自不同人的反射[7]。
度量：为了评估EQ-Radio的心跳提取算法，我们使用情感识别中常见的度量：
Inter-Beat-Interval (IBI): The IBI measures the accuracy in identifying the boundaries of each individual beat.
Root Mean Square of Successive Differences (RMSSD):
This metric focuses on differences between successive beats.RMSSD is typically used as a measure of the parasympathetic nervous activity that controls the heart [57]. We calculate RMSSD for IBI sequences in a window of 2 minutes.
Standard Deviation of NN Intervals (SDNN)：术语NN间隔指的是（IBI）。 因此，SDNN测量一个时间窗内beat length的标准偏差。 我们使用2分钟的时间窗。
baseline:我们使用商用ECG监测器获得上述指标的Ground Truth。 我们使用AD8232评估板和3导联心电图监护仪来获取ECG信号。通过将这两个设备连接到共享时钟来完成FMCW信号和ECG信号之间的同步。
###### Accuracy in comparison to ECG
我们与30名参与者进行实验，收集了超过13万次心脏跳动。 使用EQ-Radio和ECG设备同时监测每个受试者。 我们处理数据以提取上述三个指标。 我们首先比较由EQ-Radio估计的IBI与从ECG监测器获得的IBI。 图5（a）示出了散点图，其中x和y坐标分别是从EQ-Radio和ECG导出的IBI。 颜色表示特定区域中点的密度。 对角线上的点在EQ-Radio和ECG中具有相同的IBI，而与对角线的距离与误差成正比。 可以观察到，所有点都聚集在对角线周围，因此EQ-Radio可以精确地估计IBI，而不考虑它们的长度。(?)

我们对图5（b）中的误差进行了定量评估，图5（b）显示了每个心跳的EQ-Radio的IBI估计和基于ECG的IBI估计之间的差异的累积分布函数（CDF）。由于RF信号每4ms采样一次，CDF以4ms（The actual sampling rate of our receiver is 1MHz. However,because each FMCW sweep takes 4ms, we obtain one phase
measurement every 4ms. For a detailed explanation, please refer to [7]）的间隔跳变.5 CDF显示第97百分位误差为8ms。我们的结果进一步表明，EQ-Radio的平均IBI估计误差是3.2ms。由于我们的实验中平均IBI是740ms，EQ-Radio平均估计出拍频长度在其正确值的0.43％以内。

在图5（c）中，我们报告了情绪识别中常用的心跳变化度量的结果。该图显示了与基于接触的ECG传感器相比，从RF反射恢复SDNN和RMSSD时的误差的CDF。图表显示，这些指标中的每一个的中值误差都小于2％，即使是第90百分位误差也小于8％。这些情绪相关度量的高准确度，表明EQ-Radio的情绪识别准确性将与基于接触的技术一致。
###### Accuracy for different orientations & distances
在上面的实验中，对象距离EQ-Radio相对较近，距离3到4英尺，面对着设备。然而当对象远离或不面向设备时，也希望实现情绪识别。因此，我们将EQ-Radio的心跳分割准确度作为方向和距离的函数分别进行评估。首先，我们将距离固定为3英尺，并重复上述四个不同方向的实验：面向设备，背向设备，面向设备的左侧或右侧（垂直）。我们绘制了图6（a）中这四个方向的EQ-Radio的IBI估计值的中位数和标准偏差。该图显示，在所有方向上，中值误差保持在8ms以下（即beat length的1％）。正如预期的那样，当用户直接面对设备时，准确度是最高的。接下来，我们测试EQ-Radio的beat分割精度，作为它与参与者距离的函数。我们在受试者坐在与设备不同距离的椅子上进行实验。图6（b）显示了作为距离函数的IBI估计中的中值和标准偏差误差。甚至在10英尺处，中值误差小于8毫秒（即，拍长的1％）。
#### Evaluation of Emotion Recognition
In this section, we investigate whether EQ-Radio can accurately classify a person’s emotions based on RF reflections off her/his body. We also compare EQ-Radio’s performance with more traditional emotion classification methods that rely on ECG signals or images.
###### Experimental Setup
参与者：我们招募了12名参与者（6名女性）。其中6人（女性3人）有3〜7年的演艺经历。具有演艺经验的人在情绪管理方面更加熟练，这有助于收集高质量的情绪数据并提供一个参照组[48]。所有受试者都得到了补偿，所有的实验都得到了IRB的批准。

实验设计：获取高质量的情感分析数据是困难的，特别是在识别情感真实情绪方面[48]。因此，仔细设计实验至关重要。我们根据以前使用生理信号进行情绪识别的工作来设计我们的实验[34,48]。具体而言，在实验之前，受试者单独准备刺激内容（例如，个人记忆，音乐，照片和视频）;在实验过程中，受试者独自坐在5个会议室中之一，并使用准备好的刺激物来引出一定的情绪状态。某些情绪与微笑，哭泣，微笑等小动作联系在一起（我们注意到§5.1中描述的区分过滤器减轻了这种小的移动。 但是，它不能处理更大的身体动作像走路。 虽然我们使用的FMCW无线电可以隔离来自不同用户的信号，正如我们在§7.2中所展示的，为了更好地引发情绪状态，在本实验中没有其他用户在房间中）。实验结束后，受试者报告他/她感觉到这种情绪的时期。在同一时期收集的数据标有受试者报告的情绪。

在整个这些实验中，使用三个系统监测每个受试者：1）EQ-Radio，2）AD8232 ECG监视器，以及3）聚焦于受试者面部的摄像机。
Ground Truth：如上所述，引导受试者唤起特定的情绪并报告他们感觉到这种情绪的时期。受试者报告的情绪用于标记相应时期的数据。这些标记为分类提供了基础。
Baseline：我们将EQ-Radio的情绪分类和更传统的基于心电信号和图像分析情感识别方法进行比较。我们在相应的小节中描述这些系统的细节。
Metrics & Visualization：当在特定的数据点上进行测试时，分类器为每个被考虑的情绪状态输出一个分数。数据点被赋予对应于最高分的情绪。我们将被分配正确情感的测试数据的百分比作为分类准确度的度量。(?)

我们将分类的输出显示如下：回想一下，我们系统中的四种情绪可以表示为一个二维平面，其轴线是价和兴奋。每种情绪占据四个象限之一： Sadness (negative valence and negative arousal), Anger (negative valence and positive arousal), Pleasure (positive valence and negative arousal), and Joy (positive valence and positive arousal)。因此，我们可以通过在2D价唤醒空间中显示特定的测试数据来将分类结果可视化。如果这个点被正确的分类，它会落入正确的象限。
对于任何数据点，我们可以计算价和唤醒得分（也就是计算坐标）：Svalence = max（Sjoy，Spleasure） - max（Ssadness，Sanger）和Sarousal = max（Sjoy，Sanger） - max（Spleasure，Ssadness） Spleasure，Ssadness和Sanger是分类器为四种情绪输出的分类得分。例如，考虑具有以下分数的数据点：S joy = 1，Spleasure = 0，Ssadness = 0，Sanger = 0 -i.e。，这个数据点是纯粹快乐的一个单位。这样的数据点落在右上象限的对角线上。一个数据点有一个高喜悦得分，但其他情绪小分数仍然会落在欢乐象限，但不完全在对角线上。 （查看图8的例子。）
###### EQ-Radio’s emotion recognition accuracy
为了评估EQ-Radio的情绪分类准确性，我们从12个被试者收集400个两分钟的信号序列，每个情感100个序列。 我们训练两种类型的情感分类器：一个依赖于人的分类器和一个独立于人的分类器。 每个依赖于人的分类器都经过了来自特定主题的数据的训练和测试。 训练和测试是在互相排斥的数据点上使用leave-one-out交叉验证来完成的[18]。 至于独立分类器，则对11个被试者进行训练，并对剩下的被试者进行测试，并针对不同的被试者重复该过程。
我们首先报告依赖于人的分类结果。 使用价值和唤醒分数作为坐标。不同类型的分数表示数据的标签。 我们观察到，情绪聚集和隔离，表明这些情绪明显地编码在价和唤起，并且可以从EQ-Radio捕捉的特征解码。 我们也观察到这些点倾向于沿着对角线和反对角线聚集，这表明我们的分类器对预测具有高度的信心。 最后，图中还显示了每个受试者依赖于人的分类的准确性，总体平均准确率为87.0％


