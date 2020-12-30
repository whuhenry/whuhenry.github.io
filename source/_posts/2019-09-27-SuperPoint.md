---
title: '论文笔记4- SuperPoint:使用自监督的方法进行兴趣点检测和描述的方法'
categories: 论文笔记
tags:
  - 摄影测量
  - 论文笔记
  - 特征点
typora-root-url: 2019-09-27-SuperPoint
abbrlink: a3923764
date: 2019-09-27 15:24:56
---

## 1. 基本信息

> [1] Detone, D., Malisiewicz, T., & Rabinovich, A. (2018). SuperPoint: Self-supervised interest point detection and description. *IEEE Computer Society Conference on Computer Vision and Pattern Recognition Workshops*, *2018*-*June*, 337–349. https://doi.org/10.1109/CVPRW.2018.00060

发表在CVPR2018上的论文，是MagicLeap公司的Detone提出的一种基于全卷机网络的一体化自监督特征点提取和描述子生成方法。

<!-- more -->

## 2. 论文中重要的内容记录点

- 自监督方法来训练特征点检测和描述全卷积网络，得到的特征点和描述子适用于MVS问题中
- 能够在一次forward运算中得到特征点位置以及描述子
- 特征点是图像中不随光线和视角变化而变化（stable and repeatable）的点
- 特征点（interest point）的定义是ill-posed，也就是没有一个明确的定义，因此使用强监督的方法，像训练人面部特征点提取网络那样训练一般的特征点提取网络是几乎不可能的
- 通过合成场景数据训练初始特征点提取网络，合成图像的特征点位置是没有异议的（应该是出了特征点位置，其他位置的图像均为均一颜色图像）
- 最终的Loss函数是左右相片的detector的loss和descriptor的loss的和，这样就能够同时训练detector和descriptor
- MagicPoint在合成场景测试集中的表现远超传统的特征点描述子，对于背景噪声的干扰有很好的鲁棒性，但是对与真实场景，则于传统方法有一定差距，因此使用真实场景数据集对MagicPoint进行进一步训练，并结合Homographic adaptation进行。
- Homographic Adaptation：通过多个(这个个数是网络的超参数，通过测试发现大概100左右就能够得到较好的性能，再增加性能变化不大，计算时间急剧增大)H矩阵的变换图像，对变换后图像提取特征点，最后将所有变换的特征点取平均得到最终的特征点（ **这个取平均的思路放到高光谱图像中不知道是否也能成功??** ）