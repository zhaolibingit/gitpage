---
layout: post
title:  "大数据：python numpy 生成正态分布数据"
date:   2019-03-24 13:31:01 +0800
categories: 大数据
tag: 数据分析
---

* content
{:toc}

#### np.random.normal()正态分布

高斯分布的概率密度函数为：

![](http://blogdata.zhaolibin.com/FvvVpz6Gk11YHBeCNIMoMFv6u4cl)

numpy中

numpy.random.normal(loc=0.0, scale=1.0, size=None)

#### 参数的意义为：
```
loc:float	概率分布的均值，对应着整个分布的中心center
scale:float	概率分布的标准差，对应于分布的宽度，scale越大越矮胖，scale越小，越瘦高
size:int or tuple of ints	输出的shape，默认为None，只输出一个值
```
我们更经常会用到np.random.randn(size)所谓标准正太分布（μ=0, σ=1），对应于np.random.normal(loc=0, scale=1, size)

#### numpy.random.randn

numpy.random.randn(d0, d1, …, dn)是从标准正态分布中返回一个或多个样本值。
- randn函数返回一个或一组样本，具有标准正态分布。
- dn表格每个维度
- 返回值为指定维度的array


#### 生成0到100正态分布的整数
	实现方法：
		1.生成0-1的正态分布数据
		2.乘100取绝对值
