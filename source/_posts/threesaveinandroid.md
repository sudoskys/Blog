---
title: 安卓的三级缓存
date: 2021-10-30 03:16:53
tags: Android
---

## 为什么使用三级缓存？

-减少接口使用量，节省加载所需流量，减少不必要的网络请求，提升应用速度。

## 什么是三级缓存

举个离子

初次加载网络图片以及我们再次使用此资源的策略。

应用加载图片时，先去网络下载图片，然后依次存入内存，存储。

当我们再一次需要用到刚才下载的网络资源时，就不需要再重复网络请求，直接可以从内存和存储中找，策略是 内存-->存储-->网络再次请求

本地缓存可以存在SD卡中，内存缓存一般存在数组或集合中。

而数组和集合的生命周期依赖于它存在的activity中，因此当程序退出，一般情况下数组和集合中的资源会被释放。

注意：网络缓存不是缓存

### 概念普知

> 实例和对象：<br />          对象是类的一个实例，创建对象的过程也叫类的实例化。对象是以类为模板来创建的。这样在安卓的底部就会用堆来存储对象的实例，栈来存储类的对象。引用是指某些对象的实例化需要其它的对象实例，比如ImageView的实例化就需要Context对象，就是表示ImageView对于Context持有引用(ImageView holds a reference to Context)。
>

> 垃圾回收机制（GC）：<br />          对虚拟机可用内存空间，即堆空间中的对象进行识别，如果对象正在被引用，那么称其为存活对象，反之，如果对象不再被引用，则为垃圾对象，可以回收其占据的空间，用于再分配。更细致来讲就是对于GC来说，当程序员创建对象时，GC就开始监控这个对象的地址、大小以及使用情况。通常GC采用有向图的方式记录并管理堆中的所有对象，通过这种方式确定哪些对象时“可达”，哪些对象时“不可达”。当对象不可达的时候，即对象不再被引用的时候，就会被垃圾回收。该机制对虚拟机中的内存进行标记，并确定哪些内存需要回收，根据一定的回收策略，自动的回收内存，永不停息（Nerver Stop）的保证虚拟机中的内存空间，防止出现内存泄露和溢出问题。
>

> 内存泄露：<br />          当不再需要某个实例后，但是这个对象却仍然被引用，这个情况就叫做内存泄露(Memory Leak)。安卓虚拟机为每一个应用分配一定的内存空间，当内存泄露到达一定的程度就会造成内存溢出。
>

> 内存的引用：<br />          内存的引用级别包括，强引用、软引用、弱引用、虚引用。强引用是默认的引用方式, 即使内存溢出,也不会回收。软引用（softReference）, 内存不够时, 会考虑回收。 弱引用 （WeakReference）内存不够时, 更会考虑回收。虚引用（PhantomReference） 内存不够时, 最优先考虑回收! 一般我们常用到的引用就是强引用，比如引用创建一个成员变量里面的引用。对于GC来说， SoftReference的强度明显低于 SrongReference。SoftReference修饰的引用，其告诉GC：我是一个 软引用，当内存不足的时候，我指向的这个内存是可以给你释放掉的。一般对于这种占用内存资源比较大的，又不是必要的变量；或者一些占用大量内存资源的一些缓存的变量，就需要考虑 SoftReference。对于GC来说， WeakReference 的强度又明显低于 SoftReference 。 WeakReference 修饰的引用，其告诉GC：我是一个弱引用，对于你的要求我没有话说，我指向的这个内存是可以给你释放掉的。虚引用其实和上面讲到的各种引用不是一回事的，他主要是为跟踪一个对象何时被GC回收。在android里面也是有用到的：FileCleaner.java 。这些避免内存溢出的引用方式在Android 2.3+的版本上已经不再起太大作用, 因为垃圾回收器会频繁回收非强引用的对象, Android官方建议使用LRUCache。所以当我们用软引用进行内存缓存时会发现内存中的资源会被系统频繁回收。最终是从本地进行读数据。
>
> 类似listview类似的列表加载，对于不用的bitmap，LruCache会根据LRU算法清理掉。
>

这样我们就能很好的理解要三级缓存了。

所以推荐加载图片用Glide。

## 参考/摘录

[Java类、对象和实例的理解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/31539682)

[Android 中图片的三级缓存及Glide缓存机制_无与伦比的猪-CSDN博客_android glide三级缓存](https://blog.csdn.net/qq_33565218/article/details/98970980?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-2.baidujsUnder6&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-2.baidujsUnder6)

[关于Android中的三级缓存 - huang502 - 博客园 (cnblogs.com)](https://www.cnblogs.com/huangjie123/p/6130665.html)
