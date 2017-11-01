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
