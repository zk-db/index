# CountDownLatch

内部采用AQS实现，初始化时设置State。

`countDown()`方法调用时调用`tryRelease()`将state减一并通过CAS更新state；

`await()`主线程调用时，会判断state是否为0，是0的话直接退出，否则加入AQS队列等待唤醒。

<!--more-->

# CyclicBarrier

借助 ReentrantLock的Condition等待唤醒实现。

在构建CyclicBarrier时，传入的值会赋值给CyclicBarrier内部维护count变量，也会赋值给parties变量（这是可以复用的关键）

每次调用await时，会将count -1 ，操作count值是直接使用ReentrantLock来保证线程安全性；

如果count不为0，则添加则condition队列中

如果count等于0时，则把节点从condition队列添加至AQS的队列中进行全部唤醒，并且将parties的值重新赋值为count的值（实现复用）



# 总结

CountDownlatch基于AQS实现，会将构造CountDownLatch的入参传递至state，countDown()就是在利用CAS将state减-1，await()实际就是让头节点一直在等待state为0时，释放等待的线程

CyclicBarrier则利用ReentrantLock和Condition，自身维护了count和parties变量。每次调用await将count-1，并将线程加入到condition队列上。等到count为0时，则将condition队列的节点移交至AQS队列，并全部释放。

date: 2022-05-31 14:46:31
