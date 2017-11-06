# Recognizing Keystrokes Using WiFi Devices

## abstract
## introduction
## conclusion

## procedure
#### Step 6:classification
获取了基于DWT的按键shape feature之后，WiKey用他们来建一个用于分类的训练模型。由于WiKey需要比较不同按键的shape feature，我们选取了一个著名的比较算法：DTW（动态时间规整），通过执行optimal alignment来算出波形之间的距离。在这样的度量标准下，WiKey使用来自所有TX-RX天线对的那些特征训练一个kNN分类器的集合,WiKey从集合中的每个分类器获得decision(决策)，并使用多数投票来获得最终结果。

A.Dynamic Time Warping
DTW是一种基于动态编程的解决方案，用于获得任意两个波形之间的最小距离校准。DTW可以处理不同长度的波形，并通过最小化两者之间的距离，实现一个波形与另一个波形的非线性映射。
B.Classifier Training
C.Behavioral Clustering of User Data



