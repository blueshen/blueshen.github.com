---
title: 空间搜索-地球投影
date: 2020-01-16 22:03:21
tags: [ spatail search, projection, solr ]
categories: geo
---

![世界地图-墨卡托](/images/geo-projection/1920px-Mercator_projection_SW.jpg)

大比例尺制图中实际用到的投影有27种+之多，其中最重要的有：墨卡托(Mercator)投影(85%)，Lambert等角正割圆锥投影(5%)，Albers等积正割圆锥投影，等距圆锥投影, 最为常用的是横轴墨卡托投影。

具体来说，不同的地理区域常用的地图投影方法也不同。

投影种类繁多，命名也极其繁杂，有很多其实是同一类投影，单是叫法很容易混淆。下面进行一个清晰的讲解，让大家了解一下各种投影的区别。

### 投影方式

#### 投影性质划分

按投影后，在地图上的表示不同，分为**等积投影**、**等角投影**

![](/images/geo-projection/v2-43e383472900684ad1007545feb3b2e9_hd.png)

所谓等积投影，就是不同地理位置上投影后的面积是一样的，面积上比例是一样的。

等角投影，主要是指投影后，角度(经度或者纬度)是不变的。

<!--more-->

#### 投影形状划分

根据投影后的形状不同，主要分为：圆锥投影、圆柱投影、方位投影

![常见地球投影](/images/geo-projection/v2-e119d04c5e575be78dc43b558e37333f_hd.png)

在形状上投影后，展开为2D平面，如下图所示。

![img](/images/geo-projection/v2-ca03abf5291d596c7b91ca381ba95d38_hd.png)

#### 投影方位划分

按照不同的轴线都可以进行投影, 主要有以下几种：**正轴投影**、**横轴投影**、**斜轴投影**等等。

![地球投影轴](/images/geo-projection/v2-b924e67efbc78cd825ea92a6c15b1208_hd.png)

### 

### 投影命名

综合来说，依据不同投影性质，投影形状，投影方案，可以唯一确定一种投影方式。

如下图所示：

![img](/images/geo-projection/v2-e01226a3db8034b5d06e165ec8635a42_hd.png)

通过不同的方式组合，投影出来的地图也是不一样的，这些都是标准的称呼，即学术上的称呼。由于不同的人或者机构率先投入使用，所以就有了很多别名。比如墨卡托投影就是一个别名，实际上就是**等角正轴圆柱投影**。

### 墨卡托投影

**墨卡托投影就是将三维的地球表示在一个二维平面上的方法之一**，也是应用得最广泛的方法。我们平时看到的谷歌地图，百度地图，腾讯地图。都是使用的墨卡托投影。

墨卡托投影的过程其实非常简单，就是将地球投影到一个圆柱再将圆柱展开成平面。

![img](/images/geo-projection/v2-9ecc4cb58b44586ae97b54d5a1e2ca48_hd.jpg)



从球心出发射出一条直线，它与球的交点投影后的位置就是这条线与圆柱的交点。

当然，中间的计算过程中会做一些取舍。下面是一个动图，用于理解这个投影过程。

![墨卡托投影动态图](/images/geo/墨卡托投影.gif)



---

参考文档：

https://www.zhihu.com/question/21161865

https://www.zhihu.com/question/21161865/answer/21052254

https://www.zhihu.com/question/21161865/answer/531923615

https://zhuanlan.zhihu.com/p/24981976

https://en.wikipedia.org/wiki/List_of_map_projections

http://mathworld.wolfram.com/topics/MapProjections.html