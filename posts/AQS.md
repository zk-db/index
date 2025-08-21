# AQS

AbstractQueueSynchroinzer。提供了一个锁框架。内部有一个Integer类型state和一个双向队列。封装了线程锁请求入FIFO双向队列入队出队以及线程挂起唤醒等过程，实现线程安全。

# **两种模式**

独占：State作为一个独占资源

共享：State的32位拆分为多个资源。

# **公平非公平**

公平锁：通过FIFO队列，新锁请求放入队尾

非公平锁：锁请求先尝试获取锁，获取失败再入队列

# **加锁执行流程**（独占非公平锁为例）

- 调用`lock()`方法
- CAS尝试修改State获取锁
- 获取锁成功
- 获取失败，加入锁队列，前驱节点设置为`Signal`状态（有后继节点需要唤醒），挂起当前线程
- 等待前驱节点获取锁唤醒执行后，唤醒后继线程获取锁继续执行

# **解锁执行流程**

- 调用`unlock()`方法
- CAS尝试修改State
- 修改成功释放锁
- 唤醒下一个节点

# Condition

```java
// 初始化
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();

// await
try {
  lock.lock();
  condition.await();
} catch (InterruptedException e) {
  e.printStackTrace();
} finally {
  lock.unlock();
}

// lock
try {
  lock.lock();
  condition.signal();
} finally {
  lock.unlock();
}
```

Condition内部维护条件队列（单向链表）。当调用`awit()`时，加入条件队列中。调用`signal()`时将头节点转移到AQS队列尾部，等待唤醒执行。`signalAll()`会将所有条件队列的节点计入AQS队列。



# Synchronized VS AQS

- Synchronized 无需手动解锁，AQS需要
- Synchronized膨胀为重量级锁后无法回退，AQS会自适应调整
- Synchronized 的wait/notifyAll没有AQS Condition灵活
- AQS可中断锁
- AQS可以实现公平锁

create date: 2022-05-31 09:59:15