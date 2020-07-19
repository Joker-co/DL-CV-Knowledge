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
         * 优点：1. 不需要通过设置正负样本采样比例来控制样本平衡，针对性更强；
         * 缺点：1. 过度训练hard examples，忽略简单样本的训练，可能导致模型对简单样本检测能力的下降；
      * 【S-OHEM-基于loss分布采样的在线困难样本挖掘】
         * 由于损失函数由多部分组成（如分类损失+定位损失），仅根据topN损失函数挖掘难分类样本是不可靠的，因此需要根据损失函数分布进行精细挖掘；
         * 优点：1. 困难样本的挖掘更加精细；
         * 缺点：1. 不同阶段对不同损失函数的侧重不同，不同数据集不同任务对不同损失函数的侧重也不同，通过超参数设置不同损失函数的分布，引入了额外的超参数；
      * 【A-Fast-RCNN-基于对抗生成网络的方式来生成困难样本】
         * 通过GAN网络，针对特征图生成具有遮挡与形变（本文主要讨论旋转）的训练样本，采用了两个GAN网络：ASDN-生成遮挡样本，ASTN-生成形变样本，并将ASDN的输出作为ASTN的输入，生成更加鲁棒的训练样本；
         * 优点：1. 生成数据集中不存在的困难样本，扩大数据源，避免对简单样本的忽略；
         * 缺点：2. GAN网络训练困难；2. 生成的样本与训练数据的真实性还有差距，性能近似于S-OHEM；
