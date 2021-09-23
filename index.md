# Welcome to Oliver's GitHub Pages

## Visual Grounding

### 任务描述
基于自然语言描述的查询文本在图像中框选出最佳匹配的图像区域。

### 应用场景
1、可应用于基于文本的对话式问答中，嵌入在移动端或者PC中并作为人机交互的一种方式。
2、提供了图文互搜的一种新方法。

### One-Stage当前存在的问题
1、	由于采用的是one-stage检测网络框架，在检测精度上受到到限制，在面对复杂背景、小目标时表现较差。  
2、	针对图像空间特征的提取，空间特征的建模不足，对视觉特征的补充还不够。当前的想法是场景图可以对图像进行良好的空间特征提取，但是需要目标的区域特征作为图节点的特征，  这种方法可以只能用在 Two-stage的方法中，因为One-stage方法是在最后才会产生候选区域。  
3、	对文本中的属性词、关系词的无法准确理解，这也是造成在最终指代相关目标时错误的原因。对于同类别的不同属性、不同相关信息的目标时，错误率很高。  
4、	在图像与文本特征融合过程中，没有充分考虑各自的上下文信息，以及两者之间信息之间的交互，以一种动态的方式来进行融合特征，而不是简单的拼接或者元素相乘。  
5、	对于图像与文本的融合后的多模态特征包含很多冗余特征，关键在于怎么再次使用文本特征对融合后的多模态特征进行优化，从而突出相关目标并抑制无关目标。  
### 工作方向
1、对描述中的属性词、关系词的建模，增强模型理解
2、长的描述通常需要更大范围的感受野，从而获得更丰富的上下文信息
### 论文解读
```
《A Fast and Accurate One-Stage Approach to Visual Grounding》ICCV2019
Introduction
使用YOLOv3目标检测框架，在DarNet53-FPN产生三种尺度得特征图后，再分别融合文本特征得到多模态特征图F1、F2、F3，在三种特征图F上产生聚类好得Anchor,然后对所有产生得Anchors回归，  产生最终得Refer Bbox。
Approach
本文共有5个部分。Visual encoder,Text encoder、Spatial encoder、Fusion module、Ground module.
Visual encoder
使用DarkNet53作backbone并结合FPN产生8×8×1024、16×16×512、32×32×256三种大小特征图。经过1×1卷积核降维后通道数均为512.
Text encoder
使用Bert作为语言编码器提取文本特征，整个Quary经过Bert编码后，输出768D大小的向量，再经过两个全连接层降维后变为512D大小的向量。
Spatial encoder
使用空间特征对视觉特征做补充，经过空间特征编码产生三种8×8×8、16×16×8、32×32×8大小特征图分别对应三种尺度大小的特征图。核心思想是将大小为H×W大小的特征图放到一个左上角为（0，0）  右下角为（1，1）坐标系中。特征图中的任意一个点（i，j）的8种特征定义为：
Fusion module
Fusion module分别对三种尺度下的三种特征进行融合，以8×8尺度下为例，Fusion module将8×8×512的Visual feat、8×8×512的Text feat、8×8×8的
Spatial feat进行通道维度拼接得到8×8×1032的多模态特征，其他两种尺度特征同理。将多模态特征再经过1×1的卷积降维到512D得到8×8×512大小的最终多模态特征，并输入到Ground module.
Ground module
Ground module将多模态特征作为输入，输出与Quary相关的目标图像区域。与YOLOv3相同，同过聚类方法产生密集的Anchors,每个尺度的特征图分别有三种不同大小的Anchors,  共有（8 × 8 + 16 × 16 + 32 × 32）×3=4032个Anchors。将YOLOv3中尾部的Sigmoid替换为Softmax函数，并使用二分类CE loss计算偏差。Pre与Gt的IOU大于0.5则将标签置1。对4032个Anchors通过YOLOv3的回归分支对Bbox进行修正。  最终，得到相关目标区域。
Experiments
作者详细分析了Two-stage与one-stage两种方法在不同数据集上表现的差异。
作者认为，Two-stage在Fkickr30数据集和Referit数据集上表现差是因为在第一阶段产生的候选区域质量差导致的，而One-stage可以避免这个问题。
在第一阶段产生的候选区域如果与GT的IOU大于0.5，自认为此候选区域Hit到GT。此实验中，取了N=200个候选区域。从表中1，2行可以看出，  使用Mask RCNN的检测结果作为候选区域以及Mask-RCNN的RPN层输出的Proposals作为候选区域的Hit Rate明显低于通过聚类产生Anchors的Hit Rate高，这也是导致本文方法在此两种数据集上优越表现得原因。
作者在补充材料中，添加了在RefCOCO数据集上的实验。
从表2中看出，本文方法与Two-stage方法的表现相差不大，甚至要低一点。这是因为Two-stage的检测模型都是以COCO数据集为预训练模型，而RefCOCO数据集是COCO数据集的一个子集。  Two-stage方法与RefCOCO数据集有很强的相关性。从表3中看出，Mask RCNN的检测结果作为候选区域以及Mask RCNN的RPN层输出的Proposals作为候选区域的Hit Rate非常高，从而也更加证明了本文方法在多个数据集上的优越表现与良好的泛化性。
```
```
<Improving One-stage Visual Grounding by Recursive Sub-query Construction>ECCV2020
Introduction
本文在FAOA论文所提出的One-Stage方法基础上进行了改进，作者认为现在的one-stage方法忽略了对Quary的建模，对整个Quary进行编码得到一个编码向量，  这种方法在处理长句子或者负载句子时表现很差。这是因为这种建模方法会增加Quary的模糊性使得模型会关注一些不重要的单词而忽略重要单词。因此，作者提出了一种递归结构，每次递归产生一个sub-quary，并利用此sub-quary去调整提取的图像特征(此过程也是多模态融合)，并使用最后一次递归后调整的text-visual特征已经包含完整的目标相关信息并作为最终的多模态特征，进行Bbox的预测。
Approach
递归结构主要由Sub-query Learner和Sub-query Modulation两部分组成。Sub-query Learner是在每次递归时产生sub-quary。  Sub-query Modulation是利用sub-quary对上一次递归产生的text-visual进行调整。
Sub-query Learner
由语言编码器对长度为n的Quary进行编码得到S={s1 s2 …sn},并且每次递归都会产生不同的一组长度为n的单词分数向量α(k)={a1,a2…an}（可以理解对整句话中每个单词的注意力程度）。  每次递归除了输入单词特征S,上一次递归产生的text-visual特征Vk-1也输入到Sub-query Learner中从而避免过度聚焦某一个特定单词。α(k)计算方式如下.  
Wa0, ba0, Wa1, ba1是网络可学习参数，α(k)也就是产生的sub-quary
h(k)包含了先前单词历史信息,计算方式如下，其中第k次递归时的hk包含了先前k-1次的单词分数信息，也就是累加前k-1次的单词得分α。
Sub-query Learner在每一次递归中应该关注不同的单词且在递归结束后，大部分单词应该被学习过，因此作者加入了两个正则项来保证这一准则。
其中，Ldiv避免一个单词被关注多次从而强制加入多样性，Lcover使得模型可以查看所有单词从而提高覆盖范围。
Sub-query Modulation
在每一轮递归中，Sub-query Learner通过输出α(k)的方式产生sub-quary,然后生成sub-quary的特征q(k)送入到Sub-query Modulation。Sub-query Modulation使用  
q(k)去调整text-visual特征v(k-1)。
```
```
《MAttNet: Modular Attention Network for Referring Expression Comprehension》
Introduction
MAttNet为Two-Stage方法，即首先利用检测网络提取出图像上存在的objects的region，再从这些objests中选出与expressions描述最详尽的object作为referent Bbox。  以往的Two-Stage方法中对于language主要是整体处理，而不关心各个成分的作用，作者将句子分割成不同的module（subject, location, relationship)，并根据不同的module对object feature进行不同的处理。  Subject处理类别名称，颜色和其他属性，location模块处理绝对位置和（某些）相对位置，relationship模块处理其他object与subject-object的关系。
Approach
Language Attention Network
Language Attention Network主要有两种作用，首先是采用soft-parse的方法对Expression进行解析，从而得到句子的subject、loca、relat三个部分。  另外还可以计算出Expression中三个部分的贡献程度(作者考虑了不是完整主谓宾的Expression)。
Subject Module
Subject Module同样有两种作用，首先是对subject的属性预测，帮助模型理解候选目标的外观特征表示，属性可以很好的区分同一类别的不同object。  让visual feature和属性+类别描述提前建立一种联系，可以让后面计算score时的module学习起来更加容易。作者使用template-parser来获取训练集中expressions中的颜色等通用属性单词并删除低频单词。  在训练时，只有Expression中的属性单词才会参与预测。第二点，Subject Module可以在phrase的引导下来对subject的相关区域进行关注，使用phase attention模块来增强vision feature，增加不同object分数的区分度，作者将其称为’in-box’注意力。Matching Function则是分别计算候选区域与Expression中三个模块的相似度。
Location Module
作者发现有41%和36%的Expression包含明确的位置单词在Refcoco和Refcocog数据集中。因此作者对候选框的绝对位置信息编码成5D向量，  作者不仅考虑了候选框的绝对位置，还选取了5个同类别的五个object，来把相对位置也考虑进来；因为这种关系描述一般是“second left person”，所以在relationship模块是没办法考虑到这种关系的，两种位置组合再一起与Expression中的loc部分计算相似度
Experiments
文章中的各个点的对比基本都有涉及到,但relationship module对模型的准确率提升很小，基本都在1%以下。
我个人认为，提升较小的原因是数据集较为简单，大部分expression仅凭subject及localization就可以确定object；如果有较为复杂的数据集，应该可以验证这个看法。
作者relationship模块设计不是很完美，仅仅考虑了距离最近的五个object中最重要的那个。相当于已经假设关系中涉及的object都在自己的附近（数据集中大部分确实如此）。
```
