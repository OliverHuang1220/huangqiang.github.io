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

### One-Stage论文解读
```
< A Fast and Accurate One-Stage Approach to Visual Grounding >ICCV2019
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
Ground module将多模态特征作为输入，输出与Quary相关的目标图像区域。与YOLOv3相同，同过聚类方法产生密集的Anchors,每个尺度的特征图分别有三种不同大小的Anchors,  共有（8 × 8 + 16 × 16 + 32 × 32）×3=4032个Anchors。将YOLOv3中尾部的Sigmoid替换为Softmax函数，并使用二分类CE loss计算偏差。Pre与Gt的IOU大于0.5则将标签置1。对4032个Anchors通过YOLOv3的回归分支对Bbox进行修正。最终，得到相关目标区域。
Experiments
作者详细分析了Two-stage与one-stage两种方法在不同数据集上表现的差异。
作者认为，Two-stage在Fkickr30数据集和Referit数据集上表现差是因为在第一阶段产生的候选区域质量差导致的，而One-stage可以避免这个问题。
在第一阶段产生的候选区域如果与GT的IOU大于0.5，自认为此候选区域Hit到GT。此实验中，取了N=200个候选区域。从表中1，2行可以看出，使用Mask RCNN的检测结果作为候选区域以及Mask RCNN的RPN层输出的Proposals作为候选区域的Hit Rate明显低于通过聚类产生Anchors的Hit Rate高，这也是导致本文方法在此两种数据集上优越表现得原因。
作者在补充材料中，添加了在RefCOCO数据集上的实验。
从表2中看出，本文方法与Two-stage方法的表现相差不大，甚至要低一点。这是因为Two-stage的检测模型都是以COCO数据集为预训练模型，而RefCOCO数据集是COCO数据集的一个子集。  Two-stage方法与RefCOCO数据集有很强的相关性。从表3中看出，Mask RCNN的检测结果作为候选区域以及Mask RCNN的RPN层输出的Proposals作为候选区域的Hit Rate非常高，从而也更加证明了本文方法在多个数据集上的优越表现与良好的泛化性。
```
