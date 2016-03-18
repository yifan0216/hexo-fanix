title: whole-page-presentation
date: 2016-03-18 21:01:06
tags:
---

# Beyond Ranking: Optimizing Whole-Page Presentation



## 背景

搜索结果不再是简单的超链接集合，而是复杂的**异构信息**聚合页。搜索结果不仅在内容上有差异，**位置、图像、字体**等展现样式也有变化。所以文章提出了一个框架，从数据中学习，自动对聚合信息排版，使得用户对**整体页面**的**呈现结果**体验最佳。



## 问题形式化

1. 页面内容$\mathbf{x}^T=(\mathbf{x}_1^T,\dots, \mathbf{x}_k^T)$，$\mathbf{x_i}^T$包括user，query与item信息。
2. 页面呈现$\mathbf{p}$，包含$\mathbf{x}$如何展现的信息，如字体，颜色，位置，行间距等。
3. 结果页面$(\mathbf{x},\mathbf{p})$
4. 用户反馈$\mathbf{y}$，可以有点击量，点击位置，停留时间，第一次点击时间等。
5. 用户满意度$s$，通过用户反馈$\mathbf{y}$计算得到的标量$s=g(\mathbf{y})$，$g(.)$人为给出。
6. 通过数据学习得到打分函数$\mathbf{F}$对用户满意度$s$进行预估，因此给定页面内容$\mathbf{x}$与打分函数$\mathbf{F}$，可以通过最大化$\mathbf{F}$寻找最优的$\mathbf{p}$

$$\mathbf{p}^*=argmax_{\mathbf{p} \in \mathbf{P}}\mathbf{F}(\mathbf{x}, \mathbf{p})$$

$\mathbf{P}$为业务约束的呈现结果集。



## 打分函数$\mathbf{F}$与最优页面呈现$\mathbf{p}$的求解

### 问题简化

1. 页面评分函数$g(\mathbf{y})=\sum_i c_iy_i$,$y_i$为$item_i$点击行为打分
2. $\mathbf{p} \in \{0, 1\}^{k\times k}$
3. 简化为排序问题，对ctr进行优化

### 求解过程

1. Quandratic Feature Model

   a.	对$\mathbf{x}$与$\mathbf{p}$每个维度特征两两交叉，形成新特征，使用线性回归求解。

   b.	$\mathbf{F}(\mathbf{x}, \mathbf{y})=\mathbf{c}^{\top}\mathbf{U}\mathbf{x}+\mathbf{a}^{\top}\mathbf{p}$ 寻找最优解$\mathbf{p}$时，$\mathbf{c}^{\top}\mathbf{U}\mathbf{x}$为常量，$\mathbf{a}^{\top}$中包含了$\mathbf{F}$参数信息与页面内容$\mathbf{x}$，寻找最优解即是寻找一个排序，使得所有位置打分之和最大

2. gbdt

   无解析解，具体查论文。



##效果评估

### 模拟数据

1. 数据生成过程：随机生成结果页面$(\mathbf{x},\mathbf{p})$($\mathbf{p}$为7x7的布尔矩阵)，基于top position bias、top-left position bias、top-bottom bias、item-specific bias假设生成模拟样本。

2. 使用线性交叉模型学习打分函数

3. 随机生成页面内容$\mathbf{x}$，根据打分函数求得最大值与对应的最优位置，根据位置与得分画出热度图，验证打分函数确实学到了模拟样本的分布。

   ![一维排序，top position-bias](figs/fig1.png)

   ![二维排序 top-left position bias](figs/fig2.png)

   ![2维排序 top-bottom position bias](figs/fig3.png)

   ![2维排序 item-specific bias](figs/fig4.png)

### 真实数据

1. 使用2013年垂直搜索收集的真实数据，随机生成排序，收集用户反馈信息。
2. baseline对比方法 ，logit-rank与gbdt-rank
3. 离线指标预估线上指标

   $$\bar{s}=\frac{1}{N}\sum_i^N\frac{g(\mathbf{y}_i)1_{\{\mathbf{p}^*_i==\mathbf{p}_i\}}}{Pr(\mathbf{p}_i)}$$

4. 评估结果

   ![满意度对比](figs/fig5.png)

   ![CTR对比](figs/fig6.png)

5. 结论

   Pres算法在新闻类比较结构化的内容上效果与Rank算法差异不大，但是在本地化推荐结果页面上效果要优于Rank算法，因为Rank算法基于Top Position-Bias的假设，在搜索内容异构的情况下这一条假设不成立，然而Pres算法不需要任何假设。

## 移动场景的应用

​	屏幕大小与用户使用习惯，甚至可以对呈现样式做个性化。
