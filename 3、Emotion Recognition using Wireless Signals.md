# Emotion Recognition using Wireless Signals
## supplement
心率变异性是指逐次心搏间期的微小差异，它产生于自主神经系统对心脏窦房结的调制，使得心搏间期一般存在几十毫秒的差异和波动。
## abbr.
- ECG:Electrocardiograph 心电图
- IBI：inter-beat-intervals 心搏间期
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
1. The first component is an FMCW radio that transmits RF signals and receives their reflections. The radio leverages the approach in [7] to **zoom in** on human reflections and **ignore** reflections from other objects in the scene.
2. the resulting RF signal is **passed to** the beat extraction algorithm.**returns** a series of signal segments that correspond to the individual heartbeats
3.the heartbeats – along with the captured breathing patterns from RF reflections – are passed to an emotion **classification sub-system**, The emotion classification sub-system computes heartbeat-based and respiration-based features recommended in the literature [34, 14, 48] and uses an **SVM classifier** to differentiate among various emotional states.

**evaluate**:conducting user experiments with 30 subjects.依据该领域的文献 [34, 14, 48]来设计实验。要求被试者通过回忆相应的记忆（例如悲伤或愉快的记忆）来唤起特定的情绪。可以使用音乐或照片来辅助唤起。 在每个实验中，被试者报告他/她感受到的情绪，这种情绪持续的时间段。 在实验期间，同时使用EQ-Radio和商用ECG监视器来监视受试者，并且实验会录成视频，然后传递给基于图像的微软情感识别系统。

**结果**:EQ-Radio的情绪识别能力与最先进的基于ECG的系统是不相上下的：如果系统分别对被试者进行训练，情绪分类的准确性在EQ-Radio中为87％，在ECG-based系统中为88.2％。 如果对所有受试者使用同一个分类器，则EQ-Radio中为72.3％，基于ECG的系统为73.2％。对于相同的实验，基于图像的系统的准确度为39.5％。 这是因为当情绪在被试者的脸上不可见时，基于图像的系统表现不佳。

**原因**:EQ-Radio的性能优秀是由于它能准确地从RF信号提取心跳。 具体来说，若对心跳间隔的估计存在40-50毫秒的误差，会使情绪识别精度降低到44％（如我们在§7.3中的图12所示）。 而我们的算法实现3.2毫秒的IBI的平均误差，这比平均beats length的0.4％还小。
## conclusion
