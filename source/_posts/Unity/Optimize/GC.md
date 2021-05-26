---
title: Unity GC
date: 2020-05-08 11:41:32
top: 529
categories:
- Unity优化
tags:
- Unity优化
---

# 内存管理方式

* 1. **手动管理**:类似C/C++一样使用malloc/free或者new/delete来为对象分配释放内存.优点:速度快,没有额外的开销.缺点:必须清楚的知道每一个对象的使用情况,否则就容易发生内存泄露,内存野指针,空指针,内存偏移出错等等.

* 2. **半自动管理**:引用计数(Reference Count),对象创建出来后,维护一个针对该对象(可以是任何东西)的技术,使用该对象对该计数加 1,需要释放时,再减一,当计数为 0 时,销毁该对象.优点是:把创建和释放时,无需手动,并且在实际使用过程中处理,速度也快.缺点是存在循环引用的问题.例子:Unity 中的PhysX.

* 3. **全自动管理**:追踪式 GC 器(Tracing Garbage Collector), Unity使用的 GC 器是一种叫做标记/清除(Mark/Sweep)的算法,它的思路是当程序需要进行垃圾回收时,从根(GC Root)出发标记所有可达对象,然后回收没有标记的对象.                
Unity 中使用的是 Boehm-Demers-Weiser 的 GC 器:
        * Stop The World:即当发生 GC 时,程序的所有线程都必须停止工作,等 GC 完成才能继续,Unity 不支持多线程 GC,即使 Unity2019后使用的增量式 GC,在回收时也要停掉所有线程.
        * 不分代:.NET 和 Java 会把托管堆分成多个代(Generation),新生代的内存空间非常小,而且一般来说,GC 主要会集中在新生代上,这让每一次 GC 的速度也非常快,但是 unity 的 GC 是完全不分代的,即只要发生 GC,就会对整个托管堆进行 GC(Full GC).
        * 不压缩:不会对堆内存进行碎片整理.
                <!-- ![Google插件3](https://github.com/xinzhuzi/Record/tree/master/source/_posts/Unity/Optimize/GC/1.png) -->
                ![内存碎片](1.png)
        GC 会造成托管堆出现很多这样的空白"间隙",这些间隙并不会合并,当申请一个新对象时,如果没有任何一个间隙大于这个新对象大小,堆内存就会增加.


# 影响 GC 性能的主要因素
        * 影响GC速度的因素主要有两个: