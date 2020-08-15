# Helmet
 基于yolov3的车间安全帽佩戴检测系统

## 项目背景

&emsp;&emsp;这个项目为参加第十一届中国大学生服务外包创新创业大赛时所选的文思海辉的命题。  

&emsp;&emsp;关于安全帽佩戴的检测方法，目前主要分为两类。一类是基于传统图像特征提取检测的方法进行识别，一类是基于人工智能的方法进行识别。在经过项目小组的一系列讨论之下，最终选择依托pytorch与yolov3来完成这个项目。  

### yolov3算法介绍

#### 算法原理

&emsp;&emsp;YOLOv3是一个全卷积网络，共包含252层。不运用任何模式的池化，而是使用步长为 2 的卷积层进行降采样，这有助于避免池化带来的梯度负面问题。作为全卷积网络，YOLOv3对输入图像尺寸大小的要求并不严格。其训练大致流程是：利用特征提取网络从原始输入图像截取获得特定大小a*a的特征图；接着将其分成对应大小的a*a个网格单元，中心坐标落在哪个网络格中，就由该网络格来预测该目标；再通过区域生成网络RPN中每一个网络单元预测一定数量的边界框，其中只有和 ground truth 的交并比最大的边界框才能用来预测该目标。  
&emsp;&emsp;同时，如果想要在练习期间提升整体训练速度，可以选择在GPU上进行批处理。但必须事先在网络中固定所有输入图像的宽度和高度，才能将多个图像统一放进一个大批次中。

#### 网络结构 
&emsp;&emsp;YOLOv3使用DarkNet-53作为主干网络。它的整个结构里没有组合以及完全连接的层。在前向传播过程中，它通过更改卷积内核的步长来实现张量的尺寸大小的更改。并且在最终输出时，v3将打印尺寸大小为原始输入图像的1/32倍的特征图。  
![Yolov3网络结构](https://github.com/Rocky-17/Blog_illustration/blob/master/Helmet/YOLOv3_Structure.png?raw=true)  

&emsp;&emsp;首先，需要将原始输入图像统一为固定尺寸大小。然后，将其放入DarkNet-53网络中，进行特征提取处理。其中，DBL是v3的基本组件，具体组成可见图中左下角；Res1、2、8则表示对应的block里有1、2、8个unit，在这里还借鉴了ResNet的残差结构。在经过特征提取和5次下采样后，生成特征图。最后，再经过DBL和卷积层的配合，输出Y1，图中的其他两个输出Y2、Y3分别是在它们前者的基础上再通过一次处理进行输出，即将Darknet中间层的输出和上一输出尺度的某一层的上采样进行张量拼接。v3共输出了这3种不同的特征图谱。高度统一为255，基础边长比分别为13、26、52。并使用特征金字塔网络。利用多尺度进行检测。同时还要求每个单元网格推测3个界限框，每个界限框须包含五个基本参数，针对指定数据集coco的80个类别。最终有：3*(4+1+80)=255。  
&emsp;&emsp;相对于YOLOv2，YOLOv3使用逻辑回归预测框。需要先估计目标物体的所有位置可能性，再在先验锚中找出得分最高的那个prior。即对框包围的部分有一个整体的目标性评分。这种在操作在预测之前的逻辑回归方法可以去掉不必要锚框，减少多余的计算量。

## 模型训练与测试

### 环境 

|||
|-|-|
|操作系统|Ubuntu 16.04.5 LTS|
|CPU|Intel(R) Xeon(R) CPU E5-2620 v4 @ 2.10GHz|
|GPU|Nvidia 1080Ti 11G|
|Python|3.7.4|
|深度学习框架|PyTorch 1.2.0|
|CUDA|10.2.89|

### 数据

&emsp;&emsp;数据除大赛组委会所给的图片以外，自己在网络上截取了一部分。  
&emsp;&emsp;最终得到数据集共13764张图片，其中所有人佩戴有安全帽的图片为5129张，所有人均未佩戴安全帽的图片7811张，二者兼有的824张。  
&emsp;&emsp;按8:2的比例分配训练集（11011张）与测试集（2753张）

### 训练

&emsp;&emsp;训练教程参见[官方教程](https://github.com/ultralytics/yolov3/wiki/Train-Custom-Data)

### 训练结果

&emsp;&emsp;经200epoch的训练后，准确率和召回率都达到了92%左右，受制于数据采集环境过于单调，应该算是达到了一个还不错的目标。
![results](https://github.com/Rocky-17/Blog_illustration/blob/master/Helmet/results.png?raw=true)