# Non-Local Deep Features for Salient Object Detection
# 用于显著目标检测得非局部深度特征
### 摘要
- 显著性检测旨在突出图像中最相关的对象。当突出目标在杂乱的背景上被描绘出来时，使用传统模型的方法会遇到困难，而深度神经网络则会受到过程复杂性和评估速度慢的影响。本文提出了一种简化的卷积神经网络，它通过一个多分辨率的4×5网格结构将局部和全局信息结合起来。我们没有像通常的情况那样用CRF或超混合来增强空间一致性，而是实现了一个受Mumford-Shah函数启发的损失函数，该函数惩罚边界上的错误。我们在MSRA-B数据集上训练了我们的模型，并在六个不同的显著性基准数据集上进行了测试。结果表明，该方法在计算量减少18～100倍的同时，接近实时，具有较高的显著性检测性能。
### 1 引言
- 显著性检测的目的是模仿人类的视觉系统，自然地从图像的其余部分分离一个场景的主要对象。突出检测的几个应用包括图像和视频压缩[14]，上下文感知图像重定向[25]，场景解析[50]，图像缩放[3]，目标检测[44]和分割[33]。
- 一个突出的物体通常被定义为一个区域，其视觉特征不同于图像的其余部分，其形状遵循一些先验标准[5]。传统的有限元方法通常是局部像素化或区域化提取有限元图像，并与全局特征进行比较。这种比较的结果称为显著性得分，存储在显著性映射中。最近，深度学习进入了显著性检测领域，并迅速确立了自己的事实基准。与传统的无监督方法相比，它们最大的优点是可以使用结合局部和深层特征的简单优化函数进行端对端训练。
- 一些方法使用了直接卷积神经网络(CNN)模型[36]，而另一些方法提出了针对显著性检测问题的模型[25,26,29,43,50]。为了实现最先进的性能，性能最好的CNN模型需要一些重要的步骤，如生成对象提议、应用后处理、通过使用超像素来实现平滑或定义复杂的网络架构，同时使预测远远慢于实时。因此，仍然存在简化模型体系结构和加速计算的机会
- 在本文中，我们证明了通过简化的非局部深度特征(NLDF)模型可以实现最先进CNN模型的总体目标(增强预测显著性地图的空间相干性，并在优化中同时使用局部和全局特征)。受Mumford-Shah (MS)功能[35]的启发，贝叶斯损失加强了空间相干性。损失以交叉熵项和边界项的和表示。与传统的MS功能实现不同，我们使用由深度网络学习到的非本地特性，而不是原始的RGB颜色。此外，与直接最小化边界长度(如无监督的MS实现所做的那样)不同，我们使用预测和ground truth边界像素来最小化联合损失的交集。这个边界罚项被证明对我们的模型的性能有显著的贡献。
- 我们的model s网络由卷积和反卷积块组成，在一个4 - 5网格(见图1)中，网格的每一列提取分辨率特定的特征。在每个分辨率轴上还使用局部对比度处理块，以提升局部对比度强的特征。得到的局部和全局特征被合并到一个分数处理块中，以输入分辨率的一半给出最终输出。
- 由于我们的方法不依赖超像素，它是完全卷积的，因此获得了类中最好的求值速度。NLDF模型在0.08秒内评估输入图像，与其他最先进的深度学习方法相比，速度增益为18到100倍
### 2 相关工作
- 我们的方法不同于这些方法，因为它使用一个单一的完全卷积的CNN。它使用一系列的多尺度卷积和反褶积块组织在一个新的5 4网格。我们的CNN模型确保输出具有正确的大小，同时捕获本地和全球上下文以及不同分辨率的特性。受Mumford-Shah模型[35]的启发，我们适应了机器学习的背景，通过损失函数来加强空间一致性。
### 3 提出的方法
#### 3.1
- 突出区域检测和图像分割通常归结为对非凸能量函数的优化，该函数由一个数据项和一个正则项组成。

#### 3.2 网络框架
- 这里，我们提供了一个深度卷积网络架构，其目标是学习鉴别显著性特征(我们的模型如图1所示)。正如在第2节中提到的，良好的显著性特征必须考虑到图像的局部和全局上下文，并结合不同分辨率的细节。为了实现这个目标，我们实现了一个新颖的网格状CNN网络，包含5列4行。在这里，每一列都针对特定于给定输入规模的特征提取。模型的输入I(左边)是一个352×352图像，输出(右边)是一个176×176 saliency map，我们使用双线性插值将其调整为352×352。
- 我们模型的第一行包含五个源自VGG-16[39]（CONV-1到CONV5）的卷积块。如表1所示，这些卷积块包含步长2的最大池操作，该操作将其特征映射{X1，…，X5}降采样为因子2，例如{176×176，88×88，…，11×11}。第一行的最后一个和最右边的卷积块计算特定于图像全局上下文的特征XG。
- 第二行和第三行是一组10个卷积块，第2行是CONV-6到CONV-10，第3行是Contrast-1到Contrast-5。这些块的目的是计算每个分辨率的特征（Xi）和对比特征（xci）。对比度特征捕捉每个特征相对于其局部邻域的差异，这些区域要么比它们的邻居亮，要么暗。
- 最后一行是一组反褶积层，用于从11 11(右下)一直到176 176(左下)的特性映射。这些非池层是一种结合每个比例计算的特征图(Xi,XC i)的方法。左下的块构造了最终的局部特征映射XL。分数块有2个卷积层和一个softmax，通过融合局部(XL)和全局(XG)特征来计算显著性概率。表1给出了我们模型的更多细节。
##### 3.2.1 非局部特征提取
###### 多尺度局部特征
- 如图1第二行所示，卷积块CONV-6到CONV-10连接到vgg16到CONV-1到CONV-5处理块。这些卷积层的目标是学习多尺度局部feature map {X1,X2，…，X5}。每个卷积块有一个大小为3 3和128通道的内核。
###### 对比特征
- 突出性是前景对象的独特品质，使其从其周围的背景中脱颖而出。显著性特征在前景对象和背景中必须是单一形式的，但同时在前景和背景区域之间是不同的。为了捕获这种对比信息，我们在每个局部特征Xi上添加了一个对比特征。每个对比特征xci的计算方法是用其局部平均值减去Xi。平均池的内核大小为3×3 
- 需要注意的是，这种对比度特征在精神上与Achanta等人的[2]相似，后者计算像素的RGB颜色与图像的全局平均颜色之间的差值。它甚至更接近Liu和Gleicher的[28]，从一个高斯图像金字塔计算对比特征。然而，我们的方法是不同的，因为我们的特性是学习的，而不是预定义的。
###### 反卷积特征
- 由于最终输出的大小是176 176，我们使用一系列反褶积层来增加预先计算的feature map Xi和XC i的大小。我们没有按照Long等人[31]提出的{2,4,8,16}的比例增加特征图，得到粗糙的特征图，而是采用了如图1第三行所示的逐级向上采样过程。在每一个UNPOOL处理块上，我们将之前的特征图的样本提高2倍。结合局部特征Xi、局部对比特征Xc i和之前块s未合并特征Ui+1的信息，得到未合并feature map Ui
###### 局部特征映射
- XL的特征通道数等于X1和U2的总和。注意，我们尝试使用另一个unpolo操作将XL的大小从176×176增加到352×352，但是发现这个操作将计算时间增加了一倍，而没有显著提高精度。
###### 捕获全局上下文：
- 检测图像中的显著对象需要模型在为单个小区域分配显著性之前捕获图像的全局上下文。为此，我们在CONV-5块之后添加了三个卷积层来计算全局特征XG。前两个卷积层的核大小为5，最后一个卷积层的核大小为3。所有三层都有128个功能频道。
#### 3.3 交叉熵损失
- 最终显著性图是使用两个线性运算符（WL，bL）和（WG，bG）计算局部特征XL和全局特征XG的线性组合。使用softmax函数计算每个像素是否显著的概率。
#### 3.4 IoU边界损失
- 由于Dice损失或IoU边界损失在医学图像分割中的重要应用[53，40，34]，我们提出的方法近似了对等式边界长度的惩罚。 （1）使用IoU边界损耗项。 为了计算边界损失，我们使用Sobel运算符紧随其后的tanh激活，来估计显着图梯度幅度（以及边界像素）。 tanh激活将显着图的梯度幅度投影到[0，1]的概率范围。 给定显着图Cˆ j的梯度幅度和区域j的真实显着图C的梯度幅度，可以计算Dice或IoU边界损耗
