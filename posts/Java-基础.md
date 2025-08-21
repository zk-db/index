---
title: Java 基础
date: 2022-05-30 10:29:11
tags: 
  - 面试 
  - Java 
  - 基础
---

# 语言特性

- 面向对象（封装、继承、多态）
- 跨平台
- 多线程支持
- 编译与解释执行（class文件编译与JIT即时编译（编译为机器码），JVM执行时解释执行）

<!--more-->

# 基础

1. 基础数据类型

   | 基本类型  | 位数 | 字节 | 默认值  | 取值范围                                   |
   | --------- | ---- | ---- | ------- | ------------------------------------------ |
   | `byte`    | 8    | 1    | 0       | -128 ~ 127                                 |
   | `short`   | 16   | 2    | 0       | -32768 ~ 32767                             |
   | `int`     | 32   | 4    | 0       | -2147483648 ~ 2147483647                   |
   | `long`    | 64   | 8    | 0L      | -9223372036854775808 ~ 9223372036854775807 |
   | `char`    | 16   | 2    | 'u0000' | 0 ~ 65535                                  |
   | `float`   | 32   | 4    | 0f      | 1.4E-45 ~ 3.4028235E38                     |
   | `double`  | 64   | 8    | 0d      | 4.9E-324 ~ 1.7976931348623157E308          |
   | `boolean` | 1    |      | false   | true、false                                |

   注意：boolean长度为明确。取决于JVM实现，逻辑上是占用1位。

2. 自动拆装箱

   装箱其实就是调用了 包装类的`valueOf()`方法，拆箱其实就是调用了 `xxxValue()`方法。

   1. **装箱**：将基本类型用它们对应的引用类型包装起来；
   2. **拆箱**：将包装类型转换为基本数据类型；

3. 局部变量与成员变量

   局部变量通常在栈中，随方法调用结束而回收。无默认值需设置；

   成员变量通常在堆中，随对象创建而存在。有默认值。

4. 集合

   List，Map，Set以及线程安全相关集合

   集合操作参考：https://github.com/Snailclimb/JavaGuide/blob/main/docs/java/collection/java-collection-precautions-for-use.md

5. 并发

6. 异常

   共同祖先：java.lang.Throwable

   重要子类：Exception与Error

   **`Exception`** ：程序本身可以处理的异常，可以通过 `catch` 来进行捕获。`Exception` 又可以分为 Checked Exception (受检查异常，必须处理) 和 Unchecked Exception (不受检查异常，可以不处理)。

   - Checked Excpetion:
     - IOException
     - ClassNotFoundException
     - SQLException
     - FileNotFoundException
   - Unchecked Exception:
     - ArithmeticException
     - ClassCastException
     - NullPointException
     - IllegalThreadStateException
     - IndexOutOfBoundsException

   **`Error`**：`Error` 属于程序无法处理的错误 ，~~我们没办法通过 `catch` 来进行捕获~~不建议通过`catch`捕获 。例如 Java 虚拟机运行错误（`Virtual MachineError`）、虚拟机内存不够错误(`OutOfMemoryError`)、类定义错误（`NoClassDefFoundError`）等 。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。

   - OutOfMemoryError
   - StackOverFlowError
   - AssertionError
   - VritualMachineError

   注意：不要再finally中使用return！在try中return返回值会放在一个本地变量中，后续执行到finally中的return，会覆盖并返回。

   JVM官方文档明确提到：

   If the `try` clause executes a *return*, the compiled code does the following:

   1. Saves the return value (if any) in a local variable.
   2. Executes a *jsr* to the code for the `finally` clause.
   3. Upon return from the `finally` clause, returns the value saved in the local variable.

   异常使用规范：异常信息有意义；日志打印异常与抛出异常不要并存。

7. 泛型

   一套工具适配多种类型

   提供编译时检查避免错误

   泛型擦出：运行时泛型会去掉

8. 反射

   优点：灵活

   缺点：性能问题，参考：https://stackoverflow.com/questions/1392351/java-reflection-why-is-it-so-slow

   应用场景：各种框架配置与调用都有用到反射。

9. IO

   操作系统相关的知识：为了保证操作系统的稳定性和安全性，一个进程的地址空间划分为 **用户空间（User space）** 和 **内核空间（Kernel space ）** 。

   **从应用程序的视角来看的话，我们的应用程序对操作系统的内核发起 IO 调用（系统调用），操作系统负责的内核执行具体的 IO 操作。也就是说，我们的应用程序实际上只是发起了 IO 操作的调用而已，具体 IO 的执行是由操作系统的内核来完成的。**

   UNIX 系统下， IO 模型一共有 5 种： **同步阻塞 I/O**、**同步非阻塞 I/O**、**I/O 多路复用**、**信号驱动 I/O** 和**异步 I/O**。

   - BIO

     - 同步阻塞 I/O

       阻塞每次读写，等到内核态数据拷贝完成返回。

     - 同步非阻塞 I/O

       多次读取不阻塞，内核态数据拷贝时时阻塞。

   - NIO

     - select 

       维护很多socket连接（创建链接时维护socket集合），通过询问内核是否已准备好数据，如果准备好再发器read（读取过程依旧是阻塞的），内核轮询socket集合，找到准备完成的socket进行数据读取（可能只有部分socket活跃，每次轮训耗时长且无意义）。每次需要将socket集合传递给内核，有一定开销。进程被唤醒拿到相应还需要遍历确定哪个socket收到数据，共需要两次遍历。select有最大文件描述符数量限制。

       ```c
       int select(int nfds, fd_set *readfds, fd_set *writefds,
                  fd_set *exceptfds, struct timeval *timeout);
       ```

       

     - pool 同select，但是无最大文件描述符限制

       ```c
       int poll(struct pollfd fds[], nfds_t nfds, int timeout)；
       ```

       

     - epool

       基于事件

       ```c
       struct epitem {
         struct rb_node  rbn;      
         struct list_head  rdllink; 
         struct epitem  *next;      
         struct epoll_filefd  ffd;  
         int  nwait;                 
         struct list_head  pwqlist;  
         struct eventpoll  *ep;      
         struct list_head  fllink;   
         struct epoll_event  event;  
       };
       
       struct eventpoll {
         spin_lock_t       lock; 
         struct mutex      mtx;  
         wait_queue_head_t     wq; 
         wait_queue_head_t   poll_wait; 
         struct list_head    rdllist;   //就绪链表
         struct rb_root      rbr;      //红黑树根节点 
         struct epitem      *ovflist;
       };
       
       //用户数据载体
       typedef union epoll_data {
          void    *ptr;
          int      fd;
          uint32_t u32;
          uint64_t u64;
       } epoll_data_t;
       //fd装载入内核的载体
        struct epoll_event {
            uint32_t     events;    /* Epoll events */
            epoll_data_t data;      /* User data variable */
        };
        //三板斧api
       int epoll_create(int size); 
       int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);  
       int epoll_wait(int epfd, struct epoll_event *events,
                        int maxevents, int timeout);
       ```

       

   - AIO

     拿回数据异步

   参考链接：https://www.jianshu.com/p/722819425dbd

   

   10. JVM

       可以先看看这个：https://github.com/Snailclimb/JavaGuide/blob/main/docs/java/jvm/jvm-intro.md

       

       - 内存分配

       堆：eden区，2个suvivor区，old区。对象创建会在eden创建，经过一次回收会搬到suvivor区，后续每经过一次回收对象年龄加一，直到到达老年代阈值（默认15岁），对象进入老年代。老年代阈值会根据配置动态计算。

       栈：本地方法栈、虚拟机栈以及程序计数器

       方法区：加载的类信息以及运行时常量池以及JIT编译代码缓存。字符串常量池与静态变量均在堆中。

       参考：https://github.com/Snailclimb/JavaGuide/blob/main/docs/java/jvm/memory-area.md

       

       - 对象创建流程

         1. 类加载

            检查是否已加载过，如果没有进行类加载

         2. 申请内存

            类加载完成即可确定需要的内存，随后便申请内存分配

            分配策略：

            - 指针碰撞

              维护一个指针，区分已使用空间与未使用空间。

            - 空闲列表

              维护一个列表，记录哪些内存块可用

            并发分配内存：

            - CAS+失败重试：通过CAS保证操作原子性
            - TLAB（Thread Local Allocation Buffer：本地分配缓存区，在线程初始化时申请Eden一部分空间）：分配内存时先在TLAB分配，到达一定量后通过CAS刷新到堆上。缺点：空间较小大对象无法适配。

         3. 类初始化

            - 初始化零值

              设置初始值，对象头不调整

            - 设置对象头

              **虚拟机要对对象进行必要的设置**，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

            - init方法执行

       - 对象内存布局

         对象头，实例数据，对齐填充

         **Hotspot 虚拟机的对象头包括两部分信息**，**第一部分用于存储对象自身的运行时数据**（哈希码、GC 分代年龄、锁状态标志等等），**另一部分是类型指针**，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

         **实例数据部分是对象真正存储的有效信息**，也是在程序中所定义的各种类型的字段内容。

         **对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用。** 因为 Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍，换句话说就是对象的大小必须是 8 字节的整数倍。而对象头部分正好是 8 字节的倍数（1 倍或 2 倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

         

       - 对象访问方式

         句柄和直接指针

         - 句柄指向句柄池数据（堆），再由句柄池指向实例数据（堆）和对象类型数据（栈）

         - 直接指针直接执行对象实例数据（堆），其对象头存在指向对象类型数据（栈）的指针

         优缺点：句柄在对象变化过程后用户的引用无需变化，而直接指针需要变化。但是直接指针少一次指针定位开销，速度更快。

       - 类加载模型

         双亲委托

         如果自定义类加载器：继承`ClassLoader`，重写`findClass()`方法，如果需要打破双亲委托，则需要重写`loadClass()`方法

       - JVM参数与优化

         参考：https://github.com/Snailclimb/JavaGuide/blob/main/docs/java/jvm/jvm-parameters-intro.md

# 面试问题整理

1. 字符型常量与字符串常量的区别？

   字符常量相当于一个整形ASCII值，可以参与运算；字符串是内存地址。字符串常量2个字节；字符串若干个字节

2. 重写遵循规范

   1. 方法名，形参需要相同
   2. 返回值类型以及声明异常均小于或等于父类方法
   3. 访问权限需大于或等于父类方法

3. 可变长参数

   可以传入不等长参数：

   ```java
   public static void method(String... args) {
      //......
   }
   ```

   需要注意的：如果方法重载，固定参数方法优先级会高于变长参数方法

   

   

4. 基本数据类型与包装类的区别？

   1. 包装类型默认值null
   2. 包装类型可用于泛型
   3. 基本数据类型局部变量存储与Java虚拟机栈局部变量表中，基本数据类型的成员变量存在堆中，包装类型都在堆中

5. 构造方否可以override？

   不能override，可以overload

6. java 9为何将String底层由 char[] 改成了 byte[] ?

   新版的 String 其实支持两个编码方案： Latin-1 和 UTF-16。如果字符串中包含的汉字没有超过 Latin-1 可表示范围内的字符，那就会使用 Latin-1 作为编码方案。Latin-1 编码方案下，`byte` 占一个字节(8 位)，`char` 占用 2 个字节（16），`byte` 相较 `char` 节省一半的内存空间。

7. String采用运算符“+”拼接为何耗费内存？

   内部采用StringBuillder.append() 方法实现，拼接完后调用toString() 得到Stirng对象。这样就会存在大量的StringBuilder对象。

8. 字符串常量池什么作用？

   创建字符串后会在堆中创建具体对象，然后在常量池中创建对应的引用。访问字符串是直接返回常量池中的引用即可。字节码命令“ldc” 可以判断字符串常量池是否保存对应的字符串对象引用。

9. intern方法作用

   native方法：如果字符串常量池不包含字符串饮用添加。然后返回字符串引用。

10. 常量折叠

    对于源代码中存在的可以确定的static final修饰的基础变量数据类型以及String，会对其进行计算并作为常量嵌入最终生成代码中。这事javac编译器做的优化

11. finally代码一定执行吗？

    不一定！虚拟机异常终止便不会执行了。比如调用：System.exit(1); 还有所在线程死亡，CPU停止执行。

12. Java只有值传递吗？

    是的，无论是基础类型还是引用类型。如果要修改对象值，通过引用修改即可。

13. 通过类加载器与Class.forName()差异

    Class.forName() 可以指定是否初始化Class，类加载器不会初始化。如果不出实话静态代码不会得到执行。

14. 精度丢失与BigDecimal

    由于float，double等表示小数时由于二进制存储原理，对于一些小数没有精确的2进制表示形式所以存在精度丢失。BigDecimal实现利用了BigInteger，并加入小数位的概念实现避免了精读丢失。

    

    





