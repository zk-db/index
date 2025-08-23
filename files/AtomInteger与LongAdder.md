---
title: AtomInteger与LongAdder
date: 2022-05-31 08:59:49
tags:
  - AtomInteger
  - LongAdder
  - 线程安全
  - CAS
---

# CAS

Compare And Swap：通过原子的进行比较交换操作保证数据安全。

具体流程：每次更新前先判断当前值是否相等，如果相等就进行替换。否则自旋等待锁释放。

存在问题：ABA问题，就是存在另外线程将值改回之前的值，此时另一个不该获取锁的线程可以拿到锁

如何避免：通过对信息加版本号，例如：使用AtomicStampedReference作为版本信息保证不会回退旧数据

<!--more-->

# AtomInteger

底层通过CAS，能够原子的进行累加，保证线程安全。

缺点：只有一个共享资源，高并发下大部分线程自旋等待。性能不佳

改进：使用`LongAdder`



# LongAdder

底层同样采用CAS，但是内部类似分段锁机制通过一个Cell数据将资源划分开，降低失败次数并减小自旋性能损失。所以在高并发情况下推荐使用`LongAdder`
