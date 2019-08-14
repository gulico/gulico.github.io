---
title: 近似最近邻算法SSG学习笔记（二）伪码注释
date: 2019-08-14 16:49:05
tags: [搜索]
categories: [算法]
---

以下为出现在文章《Satellite System Graph: Towards the Efﬁciency Up-Boundary of Graph-Based Approximate Nearest Neighbor Search(卫星系图：接近基于图的近似最近邻搜索效率的上界)》中的两个算法，下面做简单注释。

# 算法1：贪婪搜索Search-on-Graph(G, p, q, l)

```
/**
* 功能：在图G上从起始点p开始搜索，得到k个距离查询点q最近的点
* 参数：图G 
*      起始点p  
*      查询点q 
*      候选池大小l   
* 返回：k个距离查询点q最近的点 
**/

Require: graph G, start node p, query point q, candidate pool size l 
Ensure: k nearest neighbors of q 

i = 0, candidate pool S = ∅      //最初候选池S为空，候选池中元素索引序号i = 0
S.add(p)                                   //起始点加入候选池S

while i < l do                            //当候选池中点的索引序号i小于l
	i = the index of the ﬁrst unchecked node in S  //i = 候选池中首个未查询的点的索引序号
	mark pi as checked                                               //标记pi为查询过的点
	
	for all neighbor n of pi in G do                           //将pi的所有邻居加入候选池
		S.add(n)
	end for 
	sort S in ascending order of the distance to q //将候选池中的点，以与q点的距离为依据升序排列
	
	If S.size() > l, S.resize(l)                                    //若S的大小大于l，则更新S的大小
end while 
return the ﬁrst k nodes in S

```

# 算法2：建立SSG图索引SSGIndexing(D, l, r, s, α) 

```
/**
* 算法2：SSGIndexing(D, l, r, s, α) 
* 功能：建立SSG索引
* 传入参数：数据集D，
*		  候选集大小L
*		  最大度r
*		  导航点的数量s
*		  边之间的最小角度α
* 返回：SSG图G
**/

Require: dataset D, candidate set size l, maximum outdegree r, number of navigating points s, minimum angle α.

Ensure: an SSG G. 

/**
* 1.建立近似kNN图：GkNN
**/

Build an approximate kNN graph Gknn. 
G = ∅. 


for all node i in Gknn do //遍历Gknn图中的所有点i
	P = ∅. //候选集P为空

/**
* 2.候选集P从对应点的邻居和邻居的邻居中产生
**/
	for all neighbor n of node i do //遍历i的所有邻居n，加入候选集P
		P.add(n). 
		for all neighbor n0 of node n do //遍历n的所有邻居n0，加入候选集P
			P.add(n0). 
		end for 
		remove the redundant nodes in P. //移除P中多余的边
		if P.size() ≥ l then //若P的大小大于l，则跳出循环
			break. 
		end if 
	end for 

/**
* 3.从候选集P中挑选符合SSG的i的邻居
**/
	sort P in the ascending order of the distance to i. //对候选集中的点，按照与i的距离升序排序。

	select neighbors from P according to the deﬁnition of SSG. //根据SSG选编策略，从P中选择i的邻居
end for 

/**
* 4.加强图的连通性
**/

Random select s points from the datasets as N. //随机选择s个导航点N
for all point n in N do //遍历导航点
	Strengthen the connectivity of the graph with 
	DFS-expanding from s. //利用深度优先扩展算法，加强图的连通性
end for 23: return G.

```

