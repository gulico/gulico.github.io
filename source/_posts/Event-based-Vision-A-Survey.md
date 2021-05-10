---
title: 基于事件的视觉研究综述(SNN部分阅读笔记)
date: 2020-07-27 09:49:00
tags: [DVS,脉冲神经网络]
categories: [类脑计算]
mathjax: true
---

title: *Event-based Vision: A Survey*

author: Guillermo Gallego, Tobi Delbr¨uck, Garrick Orchard, Chiara Bartolozzi, Brian Taba, Andrea Censi, Stefan Leutenegger, Andrew Davison, J¨org Conradt, Kostas Daniilidis, Davide Scaramuzza

year: 2019

journal: arXiv

**原理：**异步测量每个像素的亮度变化，并输出事件流，这些事件流对亮度变化的时间，位置和符号进行编码 。

**优势：**较高的时间分辨率（微秒级），非常高的动态范围（140 dB对60 dB），低功耗和高像素带宽（kHz级） 减少运动模糊。 因此，在传统摄像机的挑战性场景中，例如低延迟，高速和高动态范围，事件摄像机在机器人技术和计算机视觉方面具有巨大的潜力。

**概括：**重点介绍了为解锁事件摄像机的卓越性能而开发的应用程序和算法。 我们会从事件摄像头的工作原理，可用的实际传感器及其使用的任务（从低视力（特征检测和跟踪，光学流等）到高视力（重建， 细分，识别）。 我们还将讨论开发用于处理事件的技术，包括基于学习的技术，以及针对这些新型传感器的专用处理器，例如脉冲神经网络。 此外，我们着重指出仍需解决的挑战，以及寻找更有效，更受生物启发的机器感知世界并与世界互动的机会。

## 事件相机的原理(PRINCIPLE OF OPERATION OF EVENT CAMERAS)

介绍了事件摄像机，其工作原理和优点，以及它们作为新型视觉传感器所面临的挑战。

### Dynamic Vision Sensor (DVS)：

动态视觉传感器（DVS）[2]，[31]，[32]，[ 33]，[34]，对于每个像素，异步且独立地响应场景中的亮度变化。

其输出取决于场景中运动以及亮度变化的数量。当变化超过阈值时，摄像机发送事件，该事件从芯片发送，带有x，y位置，时间t和变化的1位极性p（即，亮度增加（“ ON”）） 或减小（“ OFF”）。

**通常使用地址事件表示（address-event representation, AER）**读数

### 时间相机的类型：

1. The DVS event camera: 通过处理DVS事件（即**亮度变化**）就可以解决许多应用，但很明显，有些应用还需要某种形式的静态输出（即“绝对”亮度）。 为了解决这个缺点，已经开发了同时输出动态和静态信息的照相机的几种发展。

2. 基于异步时间的图像传感器（Asynchronous Time Based Image Sensor, ATIS）[3]，[47]具有包含DVS子像素的像素，该子像素触发另一个子像素以读出**绝对强度**。

   **缺点：**ATIS具有的缺点是像素至少是DVS像素面积的两倍。 同样，在黑暗的场景中，两个强度事件之间的时间可能会很长，并且强度的读取可能会被新事件中断（[49]提出了解决此问题的方法）。

3. 动态和主动像素视觉传感器（widely-used Dynamic and Active Pixel Vision Sensor, DAVIS）在同一像素中将传统的主动像素传感器（APS）[51]与DVS结合在一起。 与ATIS相比，优点是像素尺寸小得多，仅使DVS像素面积增加了约5％。

   **缺点：**APS读出具有有限的动态范围（55db），并且像标准相机一样，如果像素不改变，它是冗余的。

### 事件相机的优势

1. 高时间分辨率：事件摄像机可以捕获非常快的运动，而不会遭受基于帧的摄像机典型的运动模糊。
2. 低延迟：每个像素独立工作，无需等待帧的全局曝光时间：一旦检测到变化，便立即进行传输。 
3. 低功耗：因为事件摄像机仅传输亮度变化，从而删除了冗余数据，所以功率仅用于处理变化的像素。
4. 高动态范围（HDR）：DVS像素可以适应非常暗和非常亮的刺激。

### 挑战

设计新颖方法（**算法和硬件**）以处理所获取的数据并从中提取信息以释放相机优势的挑战——时空，光度和随机性质：

1. **处理不同的时空输出**：事件摄像机的输出与标准摄像机的输出根本不同：事件是异步的且空间稀疏，而图像则是同步且密集的。 因此，为图像序列设计的基于帧的视觉算法不能直接应用于事件数据。
2. **应对不同的光度传感**：与标准相机提供的灰度信息相反，每个事件都包含二进制（增加/减少）亮度变化信息。 亮度变化不仅取决于场景的亮度，还取决于场景和相机之间当前和过去的运动。
3. **应对噪声和动态影响**：由于光子中固有的散粒噪声和晶体管电路噪声，所有视觉传感器都存在噪声，并且它们也存在非理想性。对于事件摄像机，这种情况尤其如此，在事件摄像机中，对时间对比度进行量化的过程很复杂，并且尚未完全表征。

### 事件产生模型

事件摄像机[2]具有独立的像素响应。 

改变其对数光电流$L \doteq \log (I)$（“亮度”）。 事件$e_k \doteq（x_k，t_k，p_k）$在像素$x_k \doteq（x_k，y_k）^⊤$且在时间t k触发，因为自该像素的最后一次事件以来亮度增加:
$$
\Delta L\left(\mathbf{x}_{k}, t_{k}\right) \doteq L\left(\mathbf{x}_{k}, t_{k}\right)-L\left(\mathbf{x}_{k}, t_{k}-\Delta t_{k}\right)
$$
达到时间对比度阈值±C
$$
\Delta L\left(\mathbf{x}_{k}, t_{k}\right)=p_{k} C
$$

- 其中C> 0，
- ∆t k是同一像素上一次事件发生后经过的时间，
- 极性p k∈{+1，-1}是亮度变化的符号

## 事件相机的结果如何处理(EVENT PROCESSING)

通常用于从事件摄像机输出中提取信息的几种方法，并讨论了某些方法背后的生物学灵感。

### 事件处理方法分类

根据同时处理的事件数量，可以区分两类算法：

1. **逐个事件操作**的方法，其中系统状态（估计的未知数）可以在事件到达时发生变化单个事件，从而实现最小的延迟，以及
2. **对事件组或事件包进行操作**的方法，这些方法会引入一些延迟。 如果不考虑等待时间，基于事件组（即时间窗口）的方法在每个事件到达时仍可以提供状态更新（如果窗口滑动一个事件）

根据事件的处理方式，我们可以区分

1. 基于模型的方法和
2. 非模型（即数据驱动的机器学习）方法。

假设事件是在优化框架中处理的，则另一分类涉及所使用的目标或损失函数的类型：基于几何，时间和光度的（例如，事件极性或事件活动的函数）。

### 事件表示方式

- **独立事件 Individual events**：$e_k =（x_k，t_k，p_k）$用于逐事件处理方法，例如概率过滤器和Spiking神经网络[7], [24], [61], [96], [97].
- **事件包 Event packet**：事件包$\mathcal{E} \doteq\left\{e_{k}\right\}_{k=1}^{N_{e}}$在一个时空邻域中被一起处理以产生输出。 此表示保留了精确的时间戳和极性信息。 选择合适的分组大小Ne对于满足算法的假设（例如，分组跨度期间的恒定运动速度）是至关重要的，该假设随任务而变化。 例子是[18]，[19]，[98]，[99]。
- **事件框架/图像或2D直方图：**时空邻域中的事件以简单的方式（例如通过对事件进行计数，累积极性等）以像素方式转换为图像（2D网格），该图像可以馈送到基于图像的计算机视觉算法中。
- **时间表面 Time surface (TS)**：S是2D地图，其中每个像素都存储一个时间值（例如，该像素[79]，[103]上一次事件的时间戳）。 为了使TS对噪声的敏感度降低，可以通过在时空窗口中过滤事件来计算每个像素值[107]。 更多示例包括[21]，[108]，[109]，[110]。
- **体素网格Voxel Grid**：是事件的时空（3D）直方图，其中每个体素代表特定的像素和时间间隔。 通过避免将事件折叠在2D网格上（图3），该表示可以更好地保留事件的时间信息。 
- **3D点集**：时空邻域中的事件被视为3D空间中的点$（x_k，y_k，t_k）∈R^3$。 因此，时间维度变成了几何维度。 它是一种稀疏表示，用于基于点的几何处理方法，例如平面拟合[21]或PointNet [116]。
- **图像平面上的点集Point sets on image plane**:事件被视为图像平面上不断发展的2D点集。 它是基于均值漂移或ICP [117]，[118]，[119]，[120]，[121]的早期形状跟踪方法中的一种流行表示，其中事件提供了跟踪边缘图案所需的唯一数据。
- （*有意思）**运动补偿事件图像 Motion-compensated event image** [122]，[123]：不仅依赖于事件而且依赖于运动假设。 运动补偿的思想是，当边缘在图像平面上移动时，它会触发所遍历像素的事件； 可以通过将事件扭曲到参考时间并最大化它们的对齐方式来估计边缘的运动，从而生成扭曲事件（IWE）的清晰图像（即直方图）[123]。 从某种意义上说，运动补偿揭示了事件流中边缘的隐藏（“运动不变”）映射。 图像对于诸如特征跟踪[63]，[125]的进一步处理可能是有用的。 有点集[126]，[127]和时间表面[128]，[129]的运动补偿版本。
- （*有意思）**重建图像 Reconstructed images**：通过图像重建（第4.6节）获得的亮度图像可以解释为比事件帧或TS更具运动不变性的表示形式，并可以用于推断[8]以产生一流的结果。

#### 关于极性

可以通过两种方式来考虑事件极性：分别处理正事件和负事件并合并结果（例如，使用TS [103]），或以通用表示形式将它们一起处理（例如，亮度增量图像[63]），其中极性为 通常在相邻事件之间汇总。

事件极性取决于运动方向，因此对于应该独立于运动的任务（例如对象识别）是一件很麻烦的事（为减轻这种情况，应使用来自多个运动方向的训练数据）。 对于运动估计任务，极性可能有用，尤其是在检测方向的突然变化时。

#### 将事件转换到网格的通用框架

[111]它还研究了传递到人工神经网络（ANN）的表示形式的选择如何影响任务性能，因此建议自动学习使这种性能最大化的表示形式。

### 事件处理方法

事件处理系统包括几个阶段：

1. 预处理（输入适应），
2. 核心处理（特征提取和分析），
3. 后处理（输出创建）。

第3.1节中的**事件表示可以发生在不同的阶段**：

- 例如，在[122]中，事件包用于预处理，而运动补偿的事件图像是核心处理阶段的内部表示。 

- 在其他情况下，以上表示只能在预处理中使用：在[22]中，事件被转换为事件图像和时间表面，然后由ANN处理。

**事件表示方法的选择：**

- 在神经网络硬件上实现的SNN（第3.3节）上逐一处理事件（第5.1节）也是很自然的，以寻求**更有效和低延迟**的解决方案。 逐事件方法的主要指数是过滤器（确定性或概率性）和SNN。 

### 事件处理方法分类

#### 逐个事件处理方法

许多方法在训练过程中使用事件包（对帧进行深度学习），然后将训练后的网络转换为SNN，逐个事件处理数据[133]，[134]，[135]，[136]，[137]。 逐事件的无模型方法主要用于对对象[15]，[103]，[133]，[134]或动作[16]，[17]，[138]进行分类，并且已针对嵌入式应用程序 [133]，通常使用自定义SNN硬件[15]，[17]（第5.1节）。 经过深度学习训练的SNN通常比依赖无监督学习进行特征提取的SNN具有更高的准确性，但是人们越来越希望找到有效的方法来直接在SNN [138]，[139]和嵌入式设备中实施监督学习。

#### 事件包处理方法

……

#### 生物启发的视觉处理

生物学原理和计算原语驱动事件摄像机像素和某些事件处理算法（和硬件）的设计，例如Spiking Neural Networks（SNN）。

- **SNN处理事件**
  - 概述
    - 人工神经元，例如Leaky-Integrate and Fire或Adaptive Exponential，是受哺乳动物视觉皮层中发现的神经元启发的计算原语。 它们是人工SNN的基本构建块。 神经元从视觉空间的一小部分区域（接受场）接收输入尖峰（“事件”），当状态超过阈值时，神经元会修改其内部状态（膜电位）并产生输出尖峰（动作电位）。
    - 信息沿着层次结构传播，从事件摄像机像素到SNN的第一层，再到更高（更深）的层。 
    - **多数第一层接受场均基于Difference of Gaussians（对中心环绕对比有选择），Gabor滤镜（对定向边缘有选择）及其组合。** 
  - 优势：对于传统视觉方法的优势在于更低的延迟和更高的效率。
  - 技术任务：
    - 受生物启发的模型已用于一些低级别的视觉任务。 例如，可以通过使用时空定向的滤波器[79]，[130]，[153]来模拟基于事件的光学流，这些滤波器模仿初级视觉皮层[154]，[155]中的接收场的工作原理。 基于[157]的生物学提议，已经使用相同类型的定向过滤器来实现基于脉冲的选择性注意模型[156]。 
    - 来自双眼视觉的生物启发模型，例如复发性侧向连接性和兴奋性抑制性神经连接[158]，已用于解决基于事件的立体对应问题[40]，[159]，[160]，[161] ，[162]或控制人形机器人上的双眼聚散[163]。 
    - （hmax）视觉皮层还启发了在[164]中提出的分层特征提取模型，该模型已在SNN中实现并用于对象识别。 这样的网络的性能越好，它们从峰值的准确定时中提取信息的效果就越好[165]。 早期的网络是手工制作的（例如，Gabor过滤器）[52]，但是最近的努力使网络可以通过大脑启发性学习（例如，穗时间依赖性可塑性（STDP））建立接受域，从而获得更好的识别率[132]。 这项研究通过在深度网络中使用更多受计算启发的监督学习类型（例如反向传播）来有效实现脉冲深度卷积网络的方法进行了补充[139]，[166]，[167]，[168]，[ 169]。
    - 为了构建小型，高效和反应性的计算系统，昆虫视力也是基于事件的处理的灵感来源。 为此，已经基于由DVS输出驱动的神经元模型开发了用于小型机器人中快速有效的避障和目标捕获的系统[170]，[171]，[172]，这些神经元模型对迫在眉睫的物体做出反应并触发逃生。

## 相关算法和应用(ALGORITHMS / APPLICATIONS)

事件相机的应用，从

- 低级（特征检测，跟踪和光学流估计）到
- 高级视觉任务（讨论与场景的3D结构有关的任务，例如深度估计，视觉测距（VO）和历史相关主题），
- 以及一些旨在释放其潜力的算法（运动分割，识别以及将感知与控制耦合）。 
- 还指出了未来研究的机会和在每个主题上的公开挑战。 

### 特征检测与跟踪

图像平面上的特征检测和跟踪是许多视觉任务（如视觉里程表，对象分割和场景理解）的基本构建块。 事件摄像机使异步跟踪成为可能，以适应场景的动态并且具有低延迟，高动态范围和低功耗（第2.2节）。 因此，它们允许跟踪标准相机各帧之间的“盲”时间。 为此，开发的方法需要处理视觉信号的独特时空和光度特性：事件仅异步报告亮度变化（第2.3节）。

#### **挑战**：

1. 由于事件表示亮度变化，而亮度变化取决于运动方向，因此使用事件摄像机进行特征检测和跟踪的主要挑战之一是克服由这种运动依赖性引起的场景外观变化（图5）。 跟踪要求在不同时间（即数据关联）在**事件之间建立对应关系**（或根据事件建立的特征），这由于外观的不同而难以实现。 
2. 第二个主要挑战包括处理**传感器噪声**和由摄像机运动引起的可能的**事件杂波**。

### 光流估计

光学流估计是在不了解场景几何形状或运动的情况下计算图像平面上物体速度的问题。

### 3D重建。 单眼和立体声

事件摄像机的深度估计是一个广泛的领域。 可以根据考虑的场景以及确定问题假设的摄像机设置或运动对其进行划分。

### 姿势估计和SLAM

用事件摄像机解决同时定位和制图（SLAM）问题一直很困难，因为为常规摄像机开发的大多数方法和概念（特征检测，匹配，迭代图像对齐等）都不适用或不可用。 事件与图片根本不同。 因此，面临的挑战是设计新的SLAM技术，该技术能够释放相机的优势（第2.3和2.2节），以显示它们在解决当前基于帧的相机的困难场景方面的有用性。

### 视觉惯性里程表（VIO）

### 影像重建

事件表示亮度变化，因此，在理想条件下（无噪声场景，完美的传感器响应等），事件的积分会产生“绝对”亮度。 这是直观的，因为事件只是对场景中视觉内容进行编码的每像素非冗余（即“压缩”）方式。 事件集成，或更一般而言，图像重建（图9）可以解释为“解压缩”事件流中编码的可视数据。

### 运动分割

静止事件摄像机所观察到的运动对象的分割很简单，因为事件仅可归因于对象的运动（假设恒定照明）[117]，[119]，[176]。 在移动摄像机的场景中会遇到挑战，因为事件是在图像平面上的任何地方触发的[13]，[128]，[151]（图10），由运动对象和静态场景（其视在运动被诱导）产生 通过相机的自我运动），目标是**推断每个事件的这种因果关系**。 但是，每个事件携带的信息很少，因此执行上述每个事件的分类是一项挑战。

### 识别

用于事件摄像机的识别算法已经变得越来越复杂，从简单形状的模板匹配到使用手工特征上的传统机器学习或现代深度学习方法对任意边缘图案进行分类。 这种发展**旨在赋予识别系统更多的可表达性（即近似能力）和对数据失真的鲁棒性**。

#### 算法

- 早期基于事件的传感器的研究始于使用静态传感器跟踪运动对象。 事件驱动的对象形状模型的位置更新用于**检测和跟踪具有已知简单形状的对象**，例如斑点[6]，圆[52]，[232]或线[53]。 通过与预定义的**模板进行匹配**，还可以检测出简单的形状，从而无需描述对象的几何形状。 此模板匹配方法是在早期硬件中使用卷积实现的。
- 对于更复杂的对象，可以使用模板来匹配低级特征，而不是整个对象，之后，可以使用分类器根据观察到的特征的分布做出决策[103]。 通常使用最近的邻居分类器，并在特征空间中计算距离。 
- 可以通过增加特征不变性来提高精度，这可以使用**层次模型**来实现，在分层模型中，每一层的特征复杂度都会增加。 有了很好的功能选择，切换任务时只需要对最终分类器进行重新培训。 这导致选择使用哪些功能的问题。 
- 早期的作品中使用了**手工制作的定向特征**，但是通过从数据本身中学习这些特征可以获得更好的结果。 在最简单的情况下，每个模板都可以从一个单独的样本中获得，但是**这样的模板对样本数据中的噪声很敏感**[15]。 
- 人们可能会遵循一种生成方法，学习能够准确重构输入的功能，如[134]中的深度信仰网络（DBN）所做的那样。 
- 最近的工作通过无监督学习，**聚类事件数据并使用每个聚类的中心作为特征来获得特征**[103]。 在推断过程中，每个事件都与其最接近的特征相关联，分类器根据观察到的特征的分布进行操作。 
- 随着基于框架的计算机视觉中深度学习的兴起，许多人寻求利用深度学习工具进行基于事件的识别，并使用反向传播来学习特征。 这种方法的优点是不需要在输出端使用单独的分类器，但缺点是需要更多的标记数据进行训练。
  - 大多数基于学习的方法将事件/峰值转换为（密集）张量，这是基于图像的分层模型（例如ANN）的便捷表示形式（图11）。 每个张量元素的值可以通过不同的方式进行计算（第3.1节）。 
  - 简单的方法使用时间表面或事件直方图框架。 一种更可靠的方法是使用具有指数衰减[103]或平均时间戳[107]的时间表面。 
  - 也可以使用图像重建方法（第4.6节）。 
  - 一些识别方法依赖于在推理过程中将脉冲转换为帧[146]，[229]，
  - 而其他人则将训练后的ANN转换为可以直接对事件数据进行操作的SNN [133]。 
  - 除了识别[22]，[102]，类似的想法也可以应用于其他任务。 随着神经形态硬件的发展（第5.1节），人们越来越有兴趣直接在SNN中学习[139]，甚至直接在神经形态硬件本身中学习[140]。

#### 任务

- 早期，检测**简单形状**（例如圆形）的

- 很快，较复杂形状（例如卡点子）的分类[133]，大写字母[15]和面孔[103]，[229]。 
- 自始至终，一个流行的任务是对**手写数字**进行分类。 受其在基于帧的计算机视觉中扮演的角色的启发，从原始的MNIST数据集[57]，[233]中生成了一些基于事件的MNIST数据集。 这些数据集仍然是算法开发的良好测试，目前许多算法在任务[107]，[138]，[139]，[234]，[235]，[236]上达到了98％以上的准确性，但很少有人提出数字识别是基于事件的视觉的优势。 
- 更多困难的任务要么涉及更多困难的对象，例如Caltech-101和Caltech-256数据集（计算机视觉仍然认为这两个数据集很容易），
- 要么涉及更多困难的场景，例如从运动中的车辆上识别[107] 。 到目前为止，几乎没有什么作品可以解决这些任务，而那些通常只能依靠生成事件框架并使用传统的深度学习框架对其进行处理的工作。

**任务的要点**：识别的关键挑战是事件摄像机对场景中的相对运动做出响应（第2.3节），因此要求对象或摄像机都在移动。 因此，事件相机不太可能成为识别静态或慢速移动物体的强大选择，尽管在这些应用中**几乎没有结合使用基于帧和事件相机的优势**。 对象的基于事件的外观高度依赖于上述相对运动（图5），因此可以严格控制摄像机的运动来帮助识别[233]。

- 由于摄像机响应动态信号，因此显而易见的应用包括通过**对象的移动方式识别对象**[237]，或**识别诸如手势或动作之类的动态场景[**16]，[17]。 与静态对象识别相比，这些任务通常更具挑战性，因为它们包括时间维度，但这**正是事件摄像机擅长的地方**。

#### 机会（应该发挥与帧相机相比事件相机的优势）

事件相机具有许多诱人的特性，但是如果要与现代的基于帧的方法竞争，基于事件的识别还有很长的路要走。 

尽管比较基于事件和基于帧的方法很重要，但必须记住每个传感器都有自己的优势。 对于基于帧的传感器，理想的采集方案包括传感器和静态对象，这对于事件摄像机而言是最糟糕的方案。

为了发现基于事件的识别并被广泛采用，需要找到能够发挥其优势的应用程序。 这种应用不太可能与成熟的计算机视觉识别任务相似，这些任务可以发挥基于帧的传感器的优势。 取而代之的是，此类应用可能涉及资源受限的动态序列识别，或来自移动平台机载的识别。 在此类应用中寻找并证明基于事件的传感器的使用仍然是一个开放的挑战。

### 神经形态控制

生物学表明，基于事件的范例原则上不仅适用于感知和推断，而且适用于控制。

## 基于事件的系统和应用(EVENT-BASED SYSTEMS AND APPLICATIONS)

神经形态处理器和嵌入式系统。



## 相关资源(RESOURCES)

可用于事件摄影机的软件，数据集和模拟器，以及其他信息源。 

收集该领域的相关资源：链接到信息（论文，视频，组织，公司和研讨会）以及驱动程序，代码，数据集，模拟器和其他必要信息 基于事件的愿景中的工具。

### 软件

- jAER [267] 11是一个基于Java的环境，用于事件传感器和事件处理和处理，如降噪，特征提取，光学流，使用IMU，CNN和RNN推理进行旋转等。
- libcaer 12是一个最小的C库，用于访问，配置和从iniVation和aiCTX神经形态传感器和处理器获取数据。 它支持DVS和DAVIS摄像机以及Dynap-SE神经形态处理器。
- 在UZH-ETH Zurich的Robotics and Perception Group中开发的ROS DVS软件包13基于libcaer。 它为DVS和DAVIS提供C ++驱动程序。
- 事件驱动的YARP项目14 [270]包含用于处理安装在iCub人形机器人上的神经形态传感器（例如DVS）的库，以及用于处理事件数据的算法。 它基于另一个机器人平台（YARP）中间件。
- pyAER 15是Inst开发的围绕libcaer的python包装器。 神经信息学（UZH-ETH）的研究可能会在快速实验中流行。
- 其他开源软件实用程序和处理算法（在Python，CUDA，Matlab等中）在基于事件的视觉研究小组的页面上遍布网络[9]。 专有软件包括公司开发的开发套件（SDK）。

### 数据集和模拟器

在[9]中列出了其中的几个，按任务排序。 从广义上讲，它们可以分为针对 

- 运动估计（回归）任务：在第一组中，有用于光学流，SLAM，对象跟踪，分割等的数据集。
- 识别（分类）任务：第二组包括用于对象和动作识别的数据集。

#### 针对识别的数据集

与传统的计算机视觉系统相比，用于识别的数据集目前规模有限。 它们由甲板上的卡片（4类），面孔（7类），手写数字（36类），动态场景，汽车等中的手势（岩石，纸张，剪刀）组成。流行的基于帧的计算机视觉的神经形态版本 通过使用类似扫视运动[233]，[273]获得了MNIST和Caltech101等数据集。 在实际场景中获取了较新的数据集[17]，[107]，[274]，[275]（不是从基于帧的数据生成的）。 这些数据集已在[15]，[103]，[107]，[133]，[146]，[147]等中用于基准基于事件的识别算法。



## (DISCUSSION)探讨

## (CONCLUSION)总结