---
title: TCP半连接队列和全连接队列溢出
date: 2022-06-04 19:38:46
tags:
  - TCP
---

转自: [TCP连接队列](https://www.cnblogs.com/sidesky/p/6844228.html)

# 问题描述

```
JAVA的client和server，使用socket通信。server使用NIO。
1.间歇性的出现client向server建立连接三次握手已经完成，但server的selector没有响应到这连接。
2.出问题的时间点，会同时有很多连接出现这个问题。
3.selector没有销毁重建，一直用的都是一个。
4.程序刚启动的时候必会出现一些，之后会间歇性出现。
```

<!--more-->

# 分析问题

- 第一步：client 发送 syn 到server 发起握手；
- 第二步：server 收到 syn后回复syn+ack给client；
- 第三步：client 收到syn+ack后，回复server一个ack表示收到了server的syn+ack（此时client的56911端口的连接已经是established）

从问题的描述来看，有点像TCP建连接的时候全连接队列（accept队列）满了，尤其是症状2、4. 为了证明是这个原因，马上通过 ss -s 去看队列的溢出统计数据：

```
667399 times the listen queue of a socket overflowed
```

反复看了几次之后发现这个overflowed 一直在增加，那么可以明确的是server上全连接队列一定溢出了

接着查看溢出后，OS怎么处理：

```
# cat /proc/sys/net/ipv4/tcp_abort_on_overflow
0
```

tcp_abort_on_overflow 为0表示如果三次握手第三步的时候全连接队列满了那么server扔掉client 发过来的ack（在server端认为连接还没建立起来）

为了证明客户端应用代码的异常跟全连接队列满有关系，我先把tcp_abort_on_overflow修改成 1，1表示第三步的时候如果全连接队列满了，server发送一个reset包给client，表示废掉这个握手过程和这个连接（本来在server端这个连接就还没建立起来）。

接着测试然后在客户端异常中可以看到很多connection reset by peer的错误，到此证明客户端错误是这个原因导致的。

于是开发同学翻看java 源代码发现socket 默认的backlog（这个值控制全连接队列的大小，后面再详述）是50，于是改大重新跑，经过12个小时以上的压测，这个错误一次都没出现过，同时 overflowed 也不再增加了。

到此问题解决，简单来说TCP三次握手后有个accept队列，进到这个队列才能从Listen变成accept，默认backlog 值是50，很容易就满了。满了之后握手第三步的时候server就忽略了client发过来的ack包（隔一段时间server重发握手第二步的syn+ack包给client），如果这个连接一直排不上队就异常了。



# TCP握手创建流程

![3d05f574867b70d1134e685e5f5ac137](3d05f574867b70d1134e685e5f5ac137.jpg)

在 TCP 三次握手的过程中，Linux 内核会维护两个队列，分别是：

- 半连接队列 (SYN Queue)
- 全连接队列 (Accept Queue)

正常的 TCP 三次握手过程：

1. Client 端向 Server 端发送 SYN 发起握手，Client 端进入 SYN_SENT 状态
2. Server 端收到 Client 端的 SYN 请求后，Server 端进入 SYN_RECV 状态，此时内核会将连接存储到半连接队列(SYN Queue)，并向 Client 端回复 SYN+ACK
3. Client 端收到 Server 端的 SYN+ACK 后，Client 端回复 ACK 并进入 ESTABLISHED 状态
4. Server 端收到 Client 端的 ACK 后，内核将连接从半连接队列(SYN Queue)中取出，添加到全连接队列(Accept Queue)，Server 端进入 ESTABLISHED 状态
5. Server 端应用进程调用 accept 函数时，将连接从全连接队列(Accept Queue)中取出

半连接队列和全连接队列都有长度大小限制，超过限制时内核会将连接 Drop 丢弃或者返回 RST 包。



# 相关指标查看

- ss 命令

  ```
  # -n 不解析服务名称 
  # -t 只显示 tcp sockets 
  # -l 显示正在监听(LISTEN)的 sockets 
   
  $ ss -lnt 
  State      Recv-Q Send-Q    Local Address:Port         Peer Address:Port 
  LISTEN     0      128       [::]:2380                  [::]:* 
  LISTEN     0      128       [::]:80                    [::]:* 
  LISTEN     0      128       [::]:8080                  [::]:* 
  LISTEN     0      128       [::]:8090                  [::]:* 
   
  $ ss -nt 
  State      Recv-Q Send-Q    Local Address:Port         Peer Address:Port 
  ESTAB      0      0         [::ffff:33.9.95.134]:80                   [::ffff:33.51.103.59]:47452 
  ESTAB      0      536       [::ffff:33.9.95.134]:80                  [::ffff:33.43.108.144]:37656 
  ESTAB      0      0         [::ffff:33.9.95.134]:80                   [::ffff:33.51.103.59]:38130 
  ESTAB      0      536       [::ffff:33.9.95.134]:80                   [::ffff:33.51.103.59]:38280 
  ESTAB      0      0         [::ffff:33.9.95.134]:80                   [:: 
  ```

  LISTEN状态Socket

  - Recv-Q：当前全连接队列的大小，即已完成三次握手等待应用程序 accept() 的 TCP 链接
  - Send-Q：全连接队列的最大长度，即全连接队列的大小

  非LISTEN状态Socket

  - Recv-Q：已收到但未被应用程序读取的字节数
  - Send-Q：已发送但未收到确认的字节数

- netstat命令

  通过 netstat -s 命令可以查看 TCP 半连接队列、全连接队列的溢出情况

  ```
  $ netstat -s | grep -i "listen" 
  189088 times the listen queue of a socket overflowed 
  30140232 SYNs to LISTEN sockets dropped 
  ```

  上面输出的数值是累计值，分别表示有多少 TCP socket 链接因为全连接队列、半连接队列满了而被丢弃

  - 189088 times the listen queue of a socket overflowed 代表有 189088 次全连接队列溢出
  - 30140232 SYNs to LISTEN sockets dropped 代表有 30140232 次半连接队列溢出

  在排查线上问题时，如果一段时间内相关数值一直在上升，则表明半连接队列、全连接队列有溢出情况



其他内容待补充, 参考文首链接
