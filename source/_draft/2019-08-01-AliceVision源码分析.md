---
title: AliceVision源码分析——深度图（depthMap）生成模块初探
date: 2019-08-01 17:55:58
categories: MVS学习笔记
tags:
	- AliceVision
	- 三维重建
	- 源码分析
	- Depth Map
typora-root-url: 2019-08-01-AliceVision源码分析
mathjax: true
---

{% note info %}
Alicevision是一个开源的多视三维重建库，包含了三维重建的整个流程，本文初步分析其深度图生成部分源码实现方法，学习在实际项目中如何应用Stereo Match的方法。
{% endnote %}

<!-- more -->

## 1. AliceVision简介

Alicevision是一个开源的多视三维重建库，使用C++和CUDA开发，主页地址为https://alicevision.org/，除了基础的代码库和示例程序外，官方还提供了一个具有GUI界面的软件Meshroom，方便用户使用。Alicevision库包含了三维重建的全流程，包括特征点提取和匹配、SFM、深度图估计、多视深度图融合（Meshing）以及纹理映射。主要由欧洲的一些研究人员进行开发，总的历史可以最早追溯到2009年，与著名的CMPMVS和openMVG库均有联系，其自动重建结果能够媲美很多商业化软件，例如Agisoft Photoscan，因此还是很有借鉴意义的。

- **AliceVision组织提供的开源库和软件**  $\downarrow$

![AliceVision组织提供的开源库和软件](1564654133128.png)

## 2. 深度图（DepthMap）生成模块初探

AliceVision的深度图生成有独立的模块`depthMap`，按照官方文档的说法，该模块使用SemiGlobal Matching (SGM) 的来进行密集匹配和深度图生成。SGM方法是著名的密集匹配方法，匹配效果好且具备易于并行化处理的优点，因此在基于深度学习的密集匹配方法出现前是最流行的密集匹配方法之一，但是原论文是在