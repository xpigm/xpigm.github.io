---
layout: post
title:  equals 和 hashCode
categories: Java
tags: []
no-post-nav: true
comments: true
---

操作使用hash算法的容器时，需要定义equals和hashCode函数，这两个函数共同作用于hash容器的查询操作。

## equals 规范

equals继承自 `Object` 类的 `equals` 函数，默认情况下这个函数会对比两个对象的内存地址，因此只有在比较同一个对象的时候，才会返回 true 。在一些情况下，我们希望放宽这一限制，只对对象内部某几个属性进行比较。

一个合适的 `equals` 函数需要满足这五个特征：

- **自反性**（Reflexive）：对于非 `null` 的 `x` 来说，`x.equals(x)` 必须返回 `true` ；
- **对称性**（Symmetric）：对于非 `null` 的 `x` 和 `y` 来说，如果 `x.equals(y)` 为 `true` ，则 `y.equals(x)` 也必须为 `true` ；
- **传递性**（Transitive）：对于非 `null` 的 `x` 、 `y` 和 `z` 来说，如果 `x.equals(y)` 为 `true` ， `y.equals(z)` 也为 `true` ，那么 `x.equals(z)` 也必须为 `true` ；
- **一致性**（Consistent）：对于非 `null` 的 `x` 和 `y` 来说，只要 `x` 和 `y` 状态不变，则 `x.equals(y)` 总是一致地返回 `true` 或者 `false` ；
- 任何非 `null` 对象与 `null` 比较时，应当返回 `false` 。

## hashCode的意义

hashCode被称作哈希函数，用来加速哈希表。

在HashMap中，查询一个值首先要进行计算哈希值，然后使用哈希值定位数组下标。如果能保证没有冲突，那就有了一个完美的哈希函数，但是这种情况只是特例。

#### 哈希冲突

由于hashCode并不总是唯一的，并且哈希表的大小是有限的，因此哈希冲突是无法避免的。但是一个优秀的哈希算法能够尽可能的将数据均匀分布，以降低哈希冲突的概率。

在HashMap中，采用了拉链式的结构处理哈希冲突，不直接保存值，而是保存值的List，然后对List中的值使用equals方法进行线性查询（在JDK1.8引入了红黑树优化过长的链表）。

#### 重写hashCode

设计hashCode函数的一个重要因素就是：无论何时，对同一个对象调用 hashCode() 都应该生成相同的值。哈希值允许是重复的，但是通过hashCode() 和 equals() ，必须能够确定一个对象的身份。因此，在重写 equals 函数时，同时也需要重写 hashCode  函数。



## 调优HashMap

#### 基本概念

- 容量：哈希表中桶（Bucket）数量；
- 初始容量：表被创建时，桶的初始个数；
- Size：表中键值对个数
- 负载因子（Load factor）:  为 `size\over capacity` ，当表内容量达到了负载因子（默认为0.75f）的阈值，表就会自动扩充为原始容量的两倍，这个过程叫做 `rehash` 。

#### 优化思路

影响哈希表性能的两个重要因素分别是**初始容量**和**负载因子**。

一般情况下，默认负载因子(0.75)在时间和空间成本之间提供了良好的平衡。较高的值会减小空间开销，但是会增加查找成本。初始容量的设置应考虑预期的条目数及负载因子的值，以尽量减少 `rehash` 的次数，但是较高的容量同时会影响迭代性能。
