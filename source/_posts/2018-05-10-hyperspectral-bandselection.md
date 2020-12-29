---
title: 基于图论表示的高光谱波段选择方法（GRBS)论文笔记
date: 2018-05-10 12:23:27
categories:
	- 高光谱
tags: 
	- 高光谱
	- 降维
	- 波段选择
	- 论文笔记
mathjax: true
---
## 1. 为什么要进行波段选择

高光谱图像一般通过连续的一百个或数百个窄观测波段，采集目标物在可见光近红外波段的反射能量信息。因此数据特点就是具备高谱段维度。这一点既是高光谱图像的优势，使高光谱图像能够根据连续的光谱曲线对地物有极强的区分能力；但与此同时在某些应用场景中也会带来很多的问题，例如图像分类问题中，多波段可能会导致“维数灾难”，也被叫做*Hughes Peonomena*。此外由于高光谱图像波段的连续性，使得相邻波段间的相关系数很高

<!-- more -->

## 2. 波段选择的方法 

根据论文*A robust and efficient band selection method using graph representation for hyperspectral imagery*[^1]中所述，主要选择方法可以包括使用监督的方法和非监督的方法，因为作者希望波段选择能够全自动进行，因此重点关注非监督方法。而非监督方法又可以分为如下几种：

- **波段优先方法（Band prioritization)**

  首先定义每个波段的代价函数或者标准（criteria），然后根据这个数值进行排序并选择，一般来说这个代价越高，波段权重更高。这一类方法计算快，但是区分性差，很容易选择出连续的波段。这类算饭的典型代表为*information divergence band selection*

- **组合优化方法（Combinatorial Optimization）**

  这一类方法将抑制相关性也纳入算法中，典型算法包括`optimum index factor (OIF)` ， `maximum ellipsoid volume (MEV)`， `orthogonal subspace projection based band selection (OSP-BS)` 和`constrained band selection (CBS)` ，这些算法通常计算复杂度高，而且通常倾向于选择outliers，而不是exemplars。下面的插图[^1]很形象的说明这个问题。具体到波段选择结果上，也就是很多选出来的波段其实是噪声波段。

{% asset_img outlier_vs_exemplars.png Outlier vs Exemplars%}

- **基于聚类的方法**

  基本思想为将波段按照相似度进行聚类，然后选择靠近聚类中心的波段作为特征波段。这一类方法对于outlier有很好的鲁棒性，但是受噪声影响大，而且计算复杂度高。这个和作者之前使用谱聚类方法结合波段互信息进行分类的实验过程非常吻合，使用全波原始波段时候，从200+的波段中选出的前10个特征波段全部是噪声波段，导致后面只有把这些波段去掉才能获得较好谱聚类结果，同时使用CPU进行计算时非常慢，只用通过采用GPU加速的方法才能够在几分钟的时间内得到结果。

## 3. GRBS（Graph Representation based Band Selection）方法

在Sun Kang et.al. 的论文[^1]中，针对上面的不同算法的优劣势，提出了一个基于图论的波段选择方法，从实现细节来看，该方法从波段优先方法出发，结合了数据科学中的特征选择（Feature Selection）算法，原理相对非常简单直接，计算复杂度低，同时论文提出该方法对于噪声和outlier有很好的鲁棒性，能够在标准数据集的上提取结果在分类问题中有良好的表现。具体的流程如下：

- **邻接矩阵构建**

该方法是以图论为基础的方法，将每个波段看做论文中的顶点$V$，将波段间的相似度看做边$E$，这样可以构建一个全连接图$G=(\mathbf{V},\mathbf{E})$，最简单的$\mathbf{E}$的计算公式如下：

{% raw %}
$$
E := \left\{ 
\begin{aligned}
E_{ij} & = exp(-\frac{||\boldsymbol{v}_i - \boldsymbol{v}_j||^2}{2\sigma^2}) \\
E_{ii} & = 0 \\
\end{aligned}
\right.
$$
{% endraw %}

这样整个边的值就可以构成一个邻接矩阵，而根据这个邻接矩阵可以算出每个波段的**度**（*degree*），可以用下面公式表达：
$$
d_i=\sum\limits_{j=1}^{n}E_{ij}
$$
其中$n$是波段的数量，也就是说这个**度**表示该波段其其他所有波段相似度之和，这个值约小表示这个波段越独立，越大表示这个波段越集群。因此，最简单直接的考虑就是挑选这个度最小的前$k$个波段作为最终选择的波段，但是需要注意，相似度越高的波段，这个**度**约接近，也就是可能会选出非常相似的$k$个波段，这显然不是我们想要的结果。因此需要一个方法来抑制相似波段的选择。

因此，论文中对于一个子波段集合，定义了下面的准则函数，来计算这个子波段集合的是否具有高的表达所有原始波段的能力以及子波段集合内部的相似度是否较低：
$$
J(\mathbf{B})=\frac{\sum\limits_{i\in \mathbf{B}}d_i}{\sum\limits_{i\in\mathbf{B}}\sum\limits_{j\in\mathbf{B}}E_{ij}}
$$
因此最终目标就是从原始的$n$个波段中选择$k$个波段，使得$J(\mathbf{B})$最小化。

- **相似度计算优化**

上面所述的相似度计算采用了最简单的欧式距离，这种计算会导致对于有噪声的谱段非常敏感，使得噪声谱段和其他所有波段的相似度均较差，因此论文对于相似度计算进行了正则化处理，使其对噪声不在敏感。

将原始公式展开可得：

{% raw %}
$$
\mathop{E_{ij}}\limits_{i \ne j}=exp(-\frac{||\mathbf{v_i}-\mathbf{v_j}||^2}{2 {\sigma}^2})=exp(-\frac{\mathbf{v_i}^\mathbf{T} \mathbf{v_i} + \mathbf{v_j}^\mathbf{T} \mathbf{v_j} - 2 \mathbf{v_i}^\mathbf{T} \mathbf{v_j} }{2 \sigma ^2})
$$
{% endraw %}

上下除以向量的大小，可以得到下面结果：
$$
\mathop{E_{ij}}\limits_{i \ne j}=exp(-\frac{1-\rho_{ij}}{\sigma^2})
$$
公式中$\rho_{ij}=\frac{\mathbf{v_i^T v_j}}{||\mathbf{v_i}||||\mathbf{v_j}||}$，是第$i$个波段和第$j$个波段间的相关系数。经过这样的正则化计算，可以有效降低噪声的影响。同时这个正则化还能够处理不同的动态范围数据，可以保证使用相同的参数处理所有数据。

- **特征波段子集选择**

最后就是根据优化后的相似度计算函数计算$J(\mathbf{B})$，这样就将问题转换为从$n$个波段中选择$k$个波段，使得$J(\mathbf{B})$最小化，这个问题本身是NP难题，但是可以使用数据科学中的特征选择算法来进行选择，方法可以参考这篇博客（[特征选择常用算法综述](http://www.cnblogs.com/heaad/archive/2011/01/02/1924088.html)）[^2]，本文选择比较简单的SFS和SBS方法。思路就是基本的贪心算法，从全集一个一个选择或者一个一个删除，使得得到的子集最小化，这样计算效率非常高，整体计算时间非常短，可以控制在秒级。

## 参考文献

[^1]: Sun, K., Geng, X., Chen, J., Ji, L., Tang, H., Zhao, Y., & Xu, M. (2016). A robust and efficient band selection method using graph representation for hyperspectral imagery. International Journal of Remote Sensing, 37(20), 4874–4889. https://doi.org/10.1080/01431161.2016.1225173
[^2]: [特征选择常用算法综述](http://www.cnblogs.com/heaad/archive/2011/01/02/1924088.html)