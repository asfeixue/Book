# ReentrantLock

## 基本介绍
可重入互斥锁与使用同步方法，同步语句的隐式监视锁有同样的基本行为和语义，但是具有扩展性。

ReentrantLock由最后成功锁定并且未释放的线程持有。一个线程执行lock时，如果锁不被另外一个线程持有，获取成功则会返回。如果当前线程已经持有锁，方法会立即返回。如下方法可以用于检查：isHeldByCurrentThread()，与 getHoldCount()。

这个class的构造方法接受一个公平参数。当设置为true，在竞争之下，锁偏好授权等待时长最大的线程。否则锁不保证任何访问顺序。程序在多线程下使用公平锁相比使用默认设置会整体表现出低吞吐量(表现为更慢；通常慢很多)，但是获取lock的时差会减小并且避免饥饿等待。但是请注意，锁的公平性不代表线程调度的公平性。因此，使用公平锁的多线程之一可以在其它活动线程不处理且当前线程未获取锁时多次获取锁。另请注意tryLock方法不符合公平性设置，即便其它线程在等待，只要锁可用就可以成功获取。

建议的做法是始终使用try块来锁定，最常见的做法是在之前/之后的结构中，比如：
```
class X {
   private final ReentrantLock lock = new ReentrantLock();
   // ...

   public void m() {
     lock.lock();  // block until condition holds
     try {
       // ... method body
     } finally {
       lock.unlock()
     }
   }
 }
```

除了实现Lock接口之外，类还定义了方法isLocked与getLockQueueLength，以及一些相关联的对于仪表和监控有用处的保护访问方法。

此类的序列化和内置的锁行为方式相同：无论其序列化时处于什么状态，反序列化时锁处于未锁定的状态。

锁支持一个线程最大2147483647次锁递归。尝试超出此限制会从锁定方法中抛出Error异常。

## ReentrantLock与synchronized比较
- 相同：ReentrantLock提供了synchronized类似的功能和内存语义。
- 不同：
1. ReentrantLock提供了时间锁等候，可中断锁等候，锁公平等，因此更有扩展性。在多条件变量和高度竞争场合，ReentrantLock更合适，ReentrantLock还提供了Condition，对线程的等待和唤醒等操作更加灵活。
2. 高竞争度的条件下，ReentrantLock性能相比synchronized好一点。
3. 因为JVM对synchronized的优化，低烈度的竞争synchronized性能会比ReentrantLock好一些。
4. ReentrantLock提供可轮询的锁请求，获取失败可以下次重试，相对不容易产生死锁一些。synchronized一旦进入锁请求要么成功，要么当前线程持续阻塞，更容易产生死锁一点。
5. synchronized可以在dump中看到，有明确关键字可以提取出来。ReentrantLock是一个普通对象，工具等无法直观识别。

## 基本功能
1. 支持可轮询的锁请求
2. 支持可定时的锁请求
3. 支持可中断的锁请求

## 使用注意
1. lock必须在finally中释放。
2. 用synchronized管理锁请求和释放时，JVM在生成线程转储时可以包含锁定信息。Lock类只是普通的对象，JVM不知道哪个线程持有Lock对象。
