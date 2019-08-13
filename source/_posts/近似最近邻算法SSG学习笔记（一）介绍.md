---
title: 近似最近邻算法SSG学习笔记（一）介绍
date: 2019-08-12 20:29:06
tags: [搜索]
categories: [算法]
---

最近阅读了论文《Satellite System Graph: Towards the Efﬁciency Up-Boundary of Graph-Based Approximate Nearest Neighbor Search(卫星系图：接近基于图的近似最近邻搜索效率的上界)》

接下来做简单的介绍：

# 背景

高维空间中的近似最近邻搜索Approximate nearest neighborn searching（ANNS）在数据库、信息检索、数据挖掘和机器学习中是必不少的。

## ANNS近似最近邻搜索问题

### ANNS的主要思想

搜索可能是近邻的数据项而不再只局限于返回最可能的项目，在牺牲可接受范围内的精度的情况下提高检索效率。

### ANNS的分类

- 基于树的方法（如图a）
  随机KD树、R树
  
- 基于散列的方法（如图b）
  局部敏感哈希（LSH）、光谱散列
  
- 基于量化的方法
  产品量化、复合量化
  
- 基于图的方法（如图c）
  NSW、HNSW、FANNG、NSG
  
  ![ANNS](近似最近邻算法SSG学习笔记（一）介绍/ANNS.png)

其中NSG与SSG都是由同一位作者提出的，两种图的主要思想和构建方法都非常类似，而且SSG的很多改进点都由NSG的缺点引出的。所以，了解NSG有利于SSG的理解。那么下面就来介绍以下NSG等相关知识。

## 相关研究

### 单调搜索网络MSNET与单调相对领域图MRNG

- **单调路径Monotonic Path**
  在给定空间的有限点集S中，p、q为在集合中的两点，且G为定义在S上的图。其中

$$
v_1,v_2...v_k(v_1=p,v_k=q)
$$

​		为从点p到q的路径上的点，且∀i = 1,...,k−1,δ(,q) > δ(+1,q)随着i的增大， 与q的距离越来越近。

- **单调搜索网络Monotonic Search Networks (MSNET)**
  在给定空间的有限点集S中，任意两点p、q之间至少存在一条单调路径。
  **特性：**
  强连通图，确保图的连接性
  不需要回溯，路径中每一步都更加接近终点q

- **单调搜索网络MSNET与单调相对领域图MRNG**
  相对邻域图 Relative Neighborhood Graph(RNG)
  在给定空间的有限点集S中，当且仅当任意两点p、q的之间的区域 中不包含集合S中的任何点。
  其中，
  $$
  lune_{pq}= B(p,δ(p,q)) ∩ B(q,δ(p,q))
  $$
  
- **单调相对邻域图Monotonic Relative Neighborhood Graph (MRNG)**
  **定义：**在给定空间的有限点集S中，满足：
  任意两点p、q的区域中不包含集合S中的任何点。（RNG的定义）
  或者，若区域中存在的点r，但不属于MRNG
  **定理：**
  在给定空间的有限点集S中，在 S 上定 义的 MRNG 是 MSNET 。

- 例子：

  图a为RNG，图b为MRNG
  图a中，pq之间的路径存在绕行；图b中，从任何节点到另一节点至少存在一个单调路径。

  ![RNG和MRNG对比图](近似最近邻算法SSG学习笔记（一）介绍/rng&mrng.png)

### NSG存在的一些缺陷

- **MSNET过于稀疏，导致搜索性能较差**
  基于ANN的搜索复杂度主要取决于结点的出度、起点到终点的路径长度
  (a)(b)(c)分别为MSNET图族的三种不同的图，且(a)->(c)：稀疏度降低
  (a)(b)(c)图分别需要7、5、8次计算。
  **结论：**并不是图越稀疏、或者越稠密效率越高，而是存在“最优稀疏度

  ![稀疏度对MSNET图族搜索性能的影响](近似最近邻算法SSG学习笔记（一）介绍/msnet_family.png)

- **规模较大的实际应用中，nsg索引的复杂度依然很高，限制了它的扩展和搜索性能**

  