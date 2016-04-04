---
title: python dict几种遍历方式性能简单比较
date: 2016-04-04 17:21:52
categories:
- Python
- Dict
tags:
- 性能
---

Python中对dict的遍历有很多种方法，本文中对几种方法性能进行比较简单的对比，给大家一个参考

## 测试环境

	Windows10 + Python2.7.8

## 测试代码

首先看我们的测试代码

<!--more-->

dict_test.py

	# -*- coding: utf-8 -*-
	
	l = [(x,x) for x in xrange(10000000)]
	d = dict(l)  
	
	from time import clock  
	
	time_list = []
	time_list.append(clock())
	
	#print d
	#该方法内部实现还不清楚，效率很高，感觉和dict.iterkeys()类似，待考证
	for i in d:
	    t = i + d[i]
	    #print "%d, %d" % (i, d[i])
	time_list.append(clock())
	
	#print d.keys()
	#调用内置方法dict.keys()，该方法将字典中的键以列表的形式返回
	for i in d.keys():
	   t = i + d[i]
	time_list.append(clock())
	
	#print d.iterkeys()
	#调用内置方法dict.iterkeys()，该方法返回针对键的迭代器
	for i in d.iterkeys():
	    t = i + d[i]
	time_list.append(clock())
	
	#print d.items()
	#dict.items()，items方法将所有字典项以列表方式返回，这些列表项中的每一项都来自于（键、值），但是在项返回时并没有特殊顺序
	for k,v in d.items():
	    t = k + v
	time_list.append(clock())
	
	#print d.iteritems()
	#dict.iteritems()，iteritems作用大致一样，但是会返回一个迭代器对象而不是列表
	for k,v in d.iteritems():
	    t = k + v
	time_list.append(clock())
	#补充 items()返回的是一个列表，所以当dict很大时会消耗大量内存。在python3中，items()进行了优化，也只返回迭代器，所以取消iteritems方法
	
	#print zip(d.iterkeys(),d.itervalues())
	#zip函数接受任意多个（包括0个和1个）序列作为参数，返回一个tuple列表
	#关于zip函数的详细说明可以参考 http://www.cnblogs.com/frydsh/archive/2012/07/10/2585370.html
	for k,v in zip(d.iterkeys(),d.itervalues()):
	    t = k + v
	time_list.append(clock())
	
	#打印各个方法所花时间
	i = 0
	while i < time_list.__len__() - 1:
	    print "%d: %s" % (i, time_list[i + 1] - time_list[i])
	    i = i + 1


## 运行结果

运行四次

	第一次
	0: 2.55650656968
	1: 2.81541793721
	2: 2.60263192552
	3: 13.3213878899
	4: 2.88246335262
	5: 4.9362081227
	
	第二次
	0: 2.59641417876
	1: 2.66579489621
	2: 2.57118659521
	3: 14.0514050106
	4: 3.30343528077
	5: 5.3133750038

	第三次
	0: 2.54319498334
	1: 2.71084397889
	2: 2.50744050815
	3: 13.2553875455
	4: 2.92418555176
	5: 4.9001545927

	第四次
	0: 2.61268754242
	1: 2.72744719433
	2: 2.73589164328
	3: 13.613833514
	4: 3.02585041181
	5: 5.39626910881

## 结论

	for k,v in dict.items() >> for k,v in zip(d.iterkeys(),d.itervalues()) > for k,v in d.iteritems() > for i in d.keys() > for i in d.iterkeys() = for i in d

理解这几种遍历方式的差异的关键就在于理解dict.items()与dict.iteritems()的区别，dict.keys()和dict.iterkeys()也是类似

**items()返回的是一个列表，所以当dict很大时会消耗大量内存，iteritems作用大致一样，但是会返回一个迭代器对象而不是列表**。在python3中，items()进行了优化，也只返回迭代器，所以取消iteritems方法