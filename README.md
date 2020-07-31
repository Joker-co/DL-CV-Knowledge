# 深度学习/计算机视觉基础知识

## 梯度消失

* 产生原因：对于传统的激活函数如sigmoid，tanh，在输入的较大与较小区域存在梯度饱和区，当输入进入饱和区，梯度趋于0；输入经过多层网络后，在不经过处理的情况下，会导致分布趋于激活函数的饱和区，因此反向传播时，在激活函数处梯度趋于0，产生梯度消失，距离输出越远的层参数受到的训练越微小；

* 解决方法：
    * 激活函数角度，使用ReLU系激活函数，避免一定程度饱和区的影响；
    * 数据分布角度，使用BN层等normalization方法，将激活函数的输入进行分布的归一化处理，使其位于激活函数的非饱和区；
    * 参数初始化角度，使用Xavier，Msra初始化以保证数据在参数处理前后的方差保持一致；
    * 使用ResNet的残差模块或denseNet等，通过短接分支，保证了信息的完整性，简化问题为学习输入与目标的差，缓解ReLU的神经元坏死问题；
    * 预训练+微调

## 梯度爆炸

* 产生原因：本质是由于连乘，对激活函数求导后梯度过大，直接原因可能是参数初始化过大；

* 解决方法：
    * 梯度裁剪，设置阈值，截取过大的梯度；
    * 正则化，控制参数大小；

## 目标检测中的样本不平衡（长尾）

* 解决方法：
   * 数据角度：均衡数据增广
   * 算法角度：
      * 【**OHEM-在线困难样本挖掘**】
         * 选取分类与框检测loss较高的样本，进行再次训练；实际操作中，采用两个共享参数的ROI Network，一个仅进行前向传播，另一个进行前向与反向传播，一张图像首先通过前向传播网络计算样本loss，选取loss较高的样本送入另一网络进行前向与反向传播；
         * **优点**：1. 不需要通过设置正负样本采样比例来控制样本平衡，针对性更强；
         * **缺点**：1. 过度训练hard examples，忽略简单样本的训练，可能导致模型对简单样本检测能力的下降；
      * 【**S-OHEM-基于loss分布采样的在线困难样本挖掘**】
         * 由于损失函数由多部分组成（如分类损失+定位损失），仅根据topN损失函数挖掘难分类样本是不可靠的，因此需要根据损失函数分布进行精细挖掘；
         * **优点**：1. 困难样本的挖掘更加精细；
         * **缺点**：1. 不同阶段对不同损失函数的侧重不同，不同数据集不同任务对不同损失函数的侧重也不同，通过超参数设置不同损失函数的分布，引入了额外的超参数；
      * 【**A-Fast-RCNN-基于对抗生成网络的方式来生成困难样本**】
         * 通过GAN网络，针对特征图生成具有遮挡与形变（本文主要讨论旋转）的训练样本，采用了两个GAN网络：ASDN-生成遮挡样本，ASTN-生成形变样本，并将ASDN的输出作为ASTN的输入，生成更加鲁棒的训练样本；
         * **优点**：1. 生成数据集中不存在的困难样本，扩大数据源，避免对简单样本的忽略；
         * **缺点**：2. GAN网络训练困难；2. 生成的样本与训练数据的真实性还有差距，性能近似于S-OHEM；
      * 【**Focal Loss**】
         * 基于交叉熵损失函数，添加权重(1-p)的指数幂以侧重对hard example的训练，包含两个超参数，权重前系数与指数，指数不可过大，否则会导致损失函数过小无法训练，权重前系数更加侧重控制样本平衡，取0~1；

## ROC & PR曲线

* **PR曲线**：以recall为横坐标，precision为纵坐标
   * 对于检测任务，首先按照置信度阈值筛选出判断为正样本的检测框，其次依据IoU阈值判定是否检测正确，Precision = TP/(TP+FP)；Recall = TP/(TP+FN)；将检测框按照置信度由高至低排序，依次遍历计算Precision与Recall，绘制曲线，PR曲线下面积为AP值；
   * 对于分类任务，首先按照置信度阈值筛选出判断为正样本的图像，依据label判定是否检测正确，其后类似；
* **ROC曲线**：以正阳率为横坐标，假阳率为纵坐标，TPR = TP/P，FPR = FP/N，绘制方法：将检测结果依据置信度由高至低排序，依次遍历计算正阳率与假阳率，ROC曲线下面积为AUC，衡量模型性能；
* **小结**：
   * ROC曲线对正负样本数量变化不敏感，可用于衡量模型在多个数据集上的表现；PR曲线对正负样本数量变化敏感，适用于不同模型在同一数据集上的表现；
   * 对于二分类任务，ROC曲线一般位于y=x曲线上方，若位于该曲线下方，只需翻转分类结果即可获得性能更好的分类器；

## LR & Softmax & SVM

* **LR**：包括二类分类与多类分类版本
   * **二类分类**：直接由样本求得类别概率，损失函数对权重求导得：x(p-y) - y为样本label{0,1}；从似然函数角度讲，等价于求解极大似然函数；
   * **多类分类**：类别有各自的权重；
* **Softmax**：
   * **多类分类**：对于模型的多类输出，首先通过softmax处理，获得归一化的各类置信度，损失函数虽然只涉及对应类别的置信度，但是置信度分母包含所有类别的输出，因此损失函数对对应类权重求导得：x(p-1)；损失函数对其他类权重求导得：xp；
* **SVM**：对于二类分类，SVM选择一个超平面将两类样本划分开，保证两类样本集中距离超平面最近的样本点与超平面保持最大距离；
   * **线性可分支持向量机**：分离超平面为：wx+b=0；决策函数为：sign(wx+b)；基于函数间隔的最优化问题；
   * **线性支持向量机**：在线性可分支持向量机的基础上，添加松弛变量与惩罚系数，准则为软间隔最大化，针对线性不可分的数据集，为硬间隔最大化不可区分的样本点使用松弛变量，并在需要最小化的目标函数中添加惩罚函数加权的松弛变量和，约束条件中针对不同的样本也添加了对应的松弛变量；**目标函数等价于合页损失函数+正则化**；
   * **非线性支持向量机**：对上述的约束最优化问题可以采用拉格朗日乘子法求解，由此可以自然引入核函数，SVM的决策函数可以使用核函数形式表示，即为内积形式的运算，等同于将低维线性不可分的输入空间通过非线性变换映射到高维线性可分的特征空间，并且不需要设计实际的变换函数，只需关注表示内积运算的核函数；

## 激活函数：

* **Sigmoid**：1/(1+exp(-z)；其导数为：sigmoid\*(1-sigmoid);
* **Tanh**：(exp(x)-exp(-x))/(exp(x)+exp(-x))；**Sigmoid与Tanh之间的关系**：tanh(x)=2sigmoid(2x)-1；
* **Sigmoid与Tanh对比**：
   * sigmoid不关于原点对称，会偏移输入数据分布使其更易偏向饱和区；tanh关于原点对称；
   * 在激活区，tanh的梯度大于sigmoid，因此收敛速度会更快；
   * tanh的饱和区大于sigmoid，更容易产生梯度消失；

## SENet-SE-Module：

* **S**：传统的卷积，关注局部特征且输出混杂空间特征与通道特征，不能单独关注通道特征，S模块对每个通道的特征进行global ave pooling做全局编码，也可使用更复杂的编码方式；
* **E**：S得到的特征没有编码通道间的关系，E模块编码通道间的关系，由两个全连接层构成的bottleneck模块，两个全连接层shape分别为(C/r)*C，C*(C/r)，r为超参数用于减少网络参数量，第一个全连接层的激活函数用ReLU，第二个全连接层的激活函数用sigmoid，使用sigmoid的原因为：SE模块获得的通道attention不是one-hot型，允许多个channel的特征贡献，因此使用sigmoid作为激活函数，同时计算成本相对低；
