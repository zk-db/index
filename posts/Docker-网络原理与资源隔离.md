# Docker 网络原理与资源隔离
Docker底层通过`namespace`和`cgroup`实现. 通过`namespace`实现资源隔离. 通过cgroup实现资源限制.

<!--more-->

# 网络原理

- Bridge(桥接)

  容器的默认网络模式，docker在安装时会创建一个名为docker0的Linux bridge，在不指定--network的情况下，创建的容器都会默认挂到docker0上面。bridge模式为容器创建独立的网络栈，保证容器内的进程使用独立的网络环境，使容器之间，容器和docker host之间实现网络隔离。

- Host

  使用宿主机的网卡进行网络访问, 无需转发,网络性能比较好

- Container

  A容器使用B容器的共享网络(??)

- None

  这样创建出来的容器完全没有网络

- User-defined

  用户自定义模式主要可选的有三种网络驱动：bridge、overlay、macvlan。bridge驱动用于创建类似于前面提到的bridge网络;overlay和macvlan驱动用于创建跨主机的网络、IP等。  

linux网络基础参考:[虚拟网络](https://www.cnblogs.com/jmilkfan-fanguiju/p/12789756.html)

# 资源隔离

- namespace

  参考namespace: [linux namespace](https://www.cnblogs.com/sparkdev/p/9365405.html)

- cgroup

  参考cgroup: [linux cgroup](https://tech.meituan.com/2015/03/31/cgroups.html)
