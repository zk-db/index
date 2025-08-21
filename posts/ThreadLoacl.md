---
title: ThreadLoacl
date: 2022-05-31 11:33:47
tags:
  - java
  - ThreadLocal
---

# 使用

```java
private static final ThreadLocal<T> res1 = new NamedThreadLocal<>("resource1");
private static final ThreadLocal<T> res2 = new NamedThreadLocal<>("resource2");
```

<!--more-->

# 原理

ThreadLocal是一个壳子，真正的存储结构是ThreadLocal里有ThreadLocalMap这么个内部类。

ThreadLocalMap的引用是在Thread上定义的

ThreadLocal本身并不存储值，它只是作为key（弱引用）来让线程从ThreadLocalMap获取value

![image-20220531114220950](image-20220531114220950.png)



# 问题

1. 为什么不用Thread作为key存取数据？这样更直观

   thread作为key则所有线程数据在一个大Map中

   - 不方便维护，一个大map。
   - 一个线程多个值需要存取，只能在value上做文章。
   - Map不能无限膨胀

2. 内存泄漏问题是什么？什么时候会发生？如何避免？

   ThreadLocalMap的Key是一个弱引用指向ThreadLocal实例。此时正常情况有两个引用指向ThreadLocal实例，一个强引用和一个弱引用。如果强引用被回收掉，ThreadLocal对象只有一个弱引用，也会被回收掉。此时ThreadLocalMap中key为null。到那时value并未被回收，如果在线程池下使用ThreadLocal并且对于key无新操作（因为ThreadLocal在每次操作时会自动清理key为null的value）就会存在大量value未回收的情况。

   ![640](640.jpeg)

