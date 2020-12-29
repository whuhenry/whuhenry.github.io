---
title: 论文笔记 - Scanning Photogrammetry for Measuring Large Targets in Close Range
date: 2019-04-29 14:11:33
categories: 论文笔记
tags:
	- 近景摄影测量
	- 旋转扫描
	- 大目标
	- 论文笔记
typora-root-url: 2019-04-29-Scanning-Photogrammetry-for-Measuring-Large-Targets-in-Close-Range
---

##  1.  近景旋转面阵图像拼接和相对定向方法

武汉大学遥感院黄山博士发表在Remote Sensing上的论文 [^1] 和毕业论文[^2]中提出的方法，在同一摄站，使用框幅式相机结合长焦镜头，延水平和垂直旋转扫描成像，获得大面积高分辨率图像。


{% asset_img scanpic_seperate.png This is an example image %}

<!-- more -->

![scanImageSynthetic](1556439504179.png)

其合成大幅高分辨率图像的思想是将同一摄站的图像重投影到指定的平面上，生成一幅更大的摄站框幅图像，原理示意图如下所示。

![modelofSynthetic](1556439690086.png)

基于该思想，其构建了整体的数据处理解决方案，**主要思想**是：

	1. 首先根据辅助信息和GCP生成Synthetic Image；
 	2. 然后利用合成图像构建自由网平差；
 	3. 利用合成图像的自由网平差结果对原始图像的匹配结果和进行优化；
 	4. 构建原始图像自由网平差；
 	5. 最后使用绝对定向完成模型计算。

在构建原始图像自由网平差的时候，其加入旋转信息作为辅助信息，利用类似旋转相机成像模型的新的成像模型，构建误差方程，以改进一般相对定向方法对于纯旋转图像（或近似纯旋转图像）无法求解的问题。提高了定位精度。

在最后的精度评定实验上，其利用拍摄了武汉大学行政楼前平台图像，并在场景中布置了标志点，使用全站仪测量获取标志点位置信息。如下图所示。

![whu](1556440499569.png)

![1556440580723](1556440580723.png)

利用本文提出的方法，在40~150m的距离上，能够得到2mm左右的测量精度（不超过2个GSD），而且是利用少量的定向点得到的结果，检查点精度稍差，但也能达到3~5mm左右的测量精度。总体测量精度高。

![1556440736395](1556440736395.png)

该文献对于实验场地的选择与构建，精度的评定，以及实验的描述等方面有较强的借鉴意义。

## 参考文献

[^1]: HUANG S, ZHANG Z, KE T, 等. Scanning Photogrammetry for Measuring Large Targets in Close Range[J]. Remote Sensing, 2015, 7(8): 10042–10077.
[^2]: 黄山. 大型目标的扫描近景摄影测量方法研究[D]. 武汉大学, 2016.