---
layout: post
title:  "python进阶：全局解释锁（GIL）"
date:   2017-02-04 13:31:01 +0800
categories: python
tag: python进阶
---

* content
{:toc}

### 什么是python的全局解释锁（GIL）？

- 我们所说的Python全局解释锁（GIL）简单来说就是一个互斥体（或者说锁），这样的机制只允许一个线程来控制Python解释器。这就意味着在任何一个时间点只有一个线程处于执行状态。
- GIL对执行单线程任务的程序员们来说并没什么显著影响，但是它成为了计算密集型（CPU-bound）和多线程任务的性能瓶颈。
- 由于GIL即使在拥有多个CPU核的多线程框架下都只允许一次运行一个线程，所以在Python众多功能中其声誉可谓是“臭名昭著”。
在这篇文章中，你将了解到GIL是如何影响到你的Python程序性能的以及如何减轻它对代码带来的影响。

### GIL解决了Python中的什么问题？

Python利用引用计数来进行内存管理，这就意味着在Python中创建的对象都有一个引用计数变量来追踪指向该对象的引用数量。当数量为0时，该对象占用的内存即被释放。
我们来通过一个简单的代码演示引用计数是如何工作的：
```python
>>> import sys
>>> a = []
>>> b = a
>>> sys.getrefcount(a)
3
```
在上述例子中，空列表对象[ ]的引用计数为3。该列表对象被a、b和传递给sys.getrefcount( )的参数引用。
回到GIL本身：
	问题在于，这个引用计数变量需要在两个线程同时增加或减少时从竞争条件中得到保护。如果发生了这种情况，可能会导致泄露的内存永远不会被释放，抑或更严重的是当一个对象的引用仍然存在的情况下错误地释放内存。这可能会导致Python程序崩溃或带来各种诡异的bug。
	通过对跨线程分享的数据结构添加锁定以至于数据不会不一致地被修改，这样做可以很好的保证引用计数变量的安全。
	但是对每一个对象或者对象组添加锁意味着会存在多个锁这也就导致了另外一个问题——死锁（只有当存在多个锁时才会发生）。而另一个副作用是由于重复获取和释放锁而导致的性能下降。
	GIL是解释器本身的一个单一锁，它增加的一条规则表明任何Python字节码的执行都需要获取解释锁。这有效地防止了死锁（因为只存在一个锁）并且不会带来太多的性能开销。但是这的确使每一个计算密集型任务变成了单线程。
	GIL虽然也被其他语言解释器使用（如Ruby），但是这不是解决这个问题的唯一办法。一些编程语言通过使用除引用计数以外的方法（如垃圾收集）来避免GIL对线程安全内存管理的请求。
	从另一方面来看，这也意味着这些语言通常需要添加其他性能提升功能（如JIT编译器）来弥补GIL单线程性能优势的损失。

### 为什么选取GIL作为解决方案？
	那么为什么在Python中使用了这样一种看似绊脚石的技术呢？这是Python开发人员的一个错误决定么？
	正如Larry Hasting所说，GIL的设计决定是Python如今受到火热追捧的重要原因之一。
	当操作系统还没有线程的概念的时候Python就一直存在着。Python设计的初衷是易于使用以便更快捷地开发，这也使得越来越多的程序员开始使用Python。
	人们针对于C库中那些被Python所需的功能写了许多扩展，为了防止不一致变化，这些C扩展需要线程安全内存管理，而这些正是GIL所提供的。
	GIL是非常容易实现而且很容易添加到Python中。因为只需要管理一个锁所以对于单线程任务来说带来了性能提升。
	非线程安全的C库变得更容易集成，而这些C扩展则成为Python被不同社区所接受的原因之一。
	正如您所看到的，GIL是CPython开发者在早期Python生涯中面对困难问题的一种实用解决方案。

### 对多线程Python程序的影响

	当你留意一些典型的Python程序或任何计算机程序时你会发现一个程序针对计算密集型和I/O密集型任务之间的性能表现是有所差异的。
	计算密集型任务是那些促使CPU达到极限的任务。这其中包括了进行数学计算的程序，如矩阵相乘、搜索、图像处理等。
	I/O密集型任务是一些需要花费时间来等待来自用户、文件、数据库、网络等的输入输出的任务。I/O密集型任务有时需要等待非常久直到他们从数据源获取到他们所需要的内容为止。这是因为在准备好输入输出之前数据源本身需要先进行自身处理。举例来说，一个用户考虑在输入提示中输入什么或者在其自己进程中运行的数据库查询。

 让我们先来看一个执行倒计时的简单的计算密集型程序：
```python
import time
from threading import Thread

COUNT = 50000000

def countdown(n):
    while n > 0:
        n -= 1

start = time.time()
countdown(COUNT)
end = time.time()

print "Time taken is second -" , end - start
```
执行结果：
```python
Time taken is second - 1.61981391907
```

接下来我对代码做出微调，使用两个线程并行处理来完成倒计时：
```python
import time
from threading import Thread

COUNT = 500000000

def countdown(n):
    while n > 0:
        n -= 1

t1 = Thread(target=countdown,args=(COUNT//2,))
t2= Thread(target=countdown,args=(COUNT//2,))


start = time.time()
t1.start()
t2.start()
t1.join()
t2.join()
end = time.time()

print "Time taken is second -" , end - start
```
执行结果：
```python
Time taken is second - 2.18046617508
```
正如你所看到的，两个版本的完成时间相差无几。在多线程版本中GIL阻止了计算密集型任务线程并行执行。
GIL对I/O密集型任务多线程程序的性能没有太大的影响，因为在等待I/O时锁可以在多线程之间共享。
但是对于一个线程是完全计算密集型的任务来说（例如，利用线程进行部分图像处理）不仅会由于锁而变成单线程任务而且还会明显的增加执行时间。正如上例中多线程与完全单线程相比的结果。
这种执行时间的增加是由于锁带来的获取和释放开销。

### 为什么GIL还没有被删除？

	Python的开发者收到了许许多多关于这方面的抱怨，但是像Python这样极受欢迎的语言无法做出去除GIL这样的巨变同时还不造成向后不兼容问题。
	GIL显然是可以被删除的，而且在过去这项任务也被开发者和研究人员多次完成。但是所有的尝试打破了在很大程度上取决于由GIL提供解决方案的C扩展市场。
	当然，还有许多其他解决方案可以解决GIL问题，但是其中一些以牺牲单线程和多线程I/O密集型任务的性能表现为代价，而另外一些解决方法又过于复杂。毕竟新版本发布后你不会希望你的Python跑得慢了些。
	BDFL of Python的创始人Guido van Rossum在2007年09月的文章《It isn’t Easy to remove the GIL》中向社区做出回答：
	“如果单线程任务和多线程I/O密集型任务的性能表现不会下降，那么我十分希望Py3k中能出现一组修补程序。”
	当然了，此后的每一次尝试都没有满足这个条件。
	为什么在Python 3 中GIL没有被移除？
	Python3中的确有机会使得许多功能从零开始，并且在这个过程中打破了那些需要更改和更新的C扩展并且将其移植到Python 3中。这也是为什么Python 3的早期版本被社区采纳的较慢的原因。

### 但是为什么GIL没有被删除？
	删除GIL会使得Python 3在处理单线程任务方面比Python 2慢，可以想像会产生什么结果。你不能否认GIL带来的单线程性能优势，这也就是为什么Python 3中仍然还有GIL。
	但是Python 3的确对现有GIL做了重大改进。
	我们仅仅讨论了GIL对“仅计算密集型任务”和“仅I/O密集型任务”的影响，但是对于那些一部分线程是计算密集型一部分线程是I/O密集型的程序来说会怎么样呢？
	在这样的程序中，Python的GIL通过不让I/O密集型线程从计算密集型线程获取GIL而使I/O密集型线程陷入瘫痪。
	这是因为Python中内嵌了一种机制，这个机制在固定连续使用时间后强迫线程释放GIL，并且如果没人获取这个GIL，那么同一线程可以继续使用。
	这个机制面临的问题是大多数计算密集型线程会在别的线程获取GIL之前再次获取GIL。这个研究工作由David Beazley进行，并且你可以在这里得到可视化资源。
	Antoine Pitrou于2009年在Python3.2中解决了这个问题，他添加了一种机制来查看其他线程请求GIL的访问数量，当数量下降时不允许当前线程在其他线程有机会运行之前重新获取GIL。

### 如何处理Python中的GIL？

如果GIL给你带来困扰，你可尝试一下方法：
多进程vs多线程：最流行的方法是应用多进程方法，在这个方法中你使用多个进程而不是多个线程。每一个Python进程都有自己的Python解释器和内存空间，因此GIL不会成为问题。Python拥有一个multiprocessing模块可以帮助我们轻松创建多进程：

```python

import time
from threading import Thread
from multiprocessing import Pool
COUNT = 500000000

def countdown(n):
    while n > 0:
        n -= 1

if __name__ ==  "__main__":
    pool = Pool(processes=5)
    start = time.time()
    t1 = pool.apply_async(countdown,[COUNT//5])
    t2 = pool.apply_async(countdown, [COUNT // 5])
    t3 = pool.apply_async(countdown, [COUNT // 5])
    t4 = pool.apply_async(countdown, [COUNT // 5])
    t5 = pool.apply_async(countdown, [COUNT // 5])
    pool.close()
    pool.join()
    end = time.time()
    print "Time taken is second -" , end - start
```
运行结果：
```python
Time taken is second - 3.81723690033
```
相比于多线程版本，性能有所提升。

但是时间并没有下降到我们之前版本的1/5，这是因为进程管理有自己的开销。多进程比多线程更“重”，因此请记住，这可能成为规模瓶颈。
替代Python解释器：Python中有多个解释器实现办法，分别用C,Java,C#和Python编写的CPython，JPython，IronPython和PyPy是最受欢迎的。GIL只存在于传统的Python实现方法如CPython中。如果你的程序及其库文件可以通过别的实现方式实现，那么你也可以尝试一下。
等等看吧：许多用户利用GIL提升了单线程任务性能表现。当然多线程程序员们也不必为此烦恼，因为Python社区内的一些聪明大脑们正在致力于从CPython中删除GIL。其中一种尝试为Giletomy。
Python GIL经常被认为是一个神秘而困难的话题。但是请记住作为一名Python支持者，只有当您正在编写C扩展或者您的程序中有计算密集型的多线程任务时才会被GIL影响。
在这种情况下，这篇文章应该给了你需要的一切去了解GIL是什么以及如何在自己的项目中处理它。如果您希望了解GIL的低层次内部运行，我建议您观看David Beazley的Understanding the Python GIL。