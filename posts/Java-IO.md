---
title: Java IO
date: 2022-05-31 08:37:56
tags: 
  - Java
  - IO
---

# BIO -> NIO

>  BIO： Blocking IO 阻塞IO，按字节进行处理
>
> NIO：No-Blocking IO（New-IO），非阻塞。按缓冲区处理

<!--more-->



# NIO

3个组件：

- Selector：检查Channel状态变化
- Channel：运输数据通道
- Buffer：数据存储

零拷贝：

- mmap（内核缓冲区与用户缓冲区的共享）

- sendfile（系统底层函数支持）



# 操作系统支持

select，poll，epoll





