# Condition

## 基本介绍
Condition因子将对象监视器方法(wait, notify and notifyAll)转换成不同的对象，并通过将他们与任意锁实现组合提供每个给定对象有多个等待集合的效果。Lock可以取代使用synchronized的方法和语句，Condition也可以取代Object的监控器方法。

Condition(也称之为条件队列或条件变量)提供一种方式，让一个线程执行挂起等待，直到另外一个线程通知它condition的某些状态当前是true。因为对共享信息的访问会在多个线程中发生，其必须被保护，所有某种形式的锁会和condition建立关联。等待Condition的关键属性在于它原子的释放锁并且挂起当前线程，类似Object.wait。

一个Condition实例实质上是绑定到具体的lock上。要从特定Lock实例上获取Condition实例，使用newCondition()方法。

例如，假如我们有一个支持put和take的游街缓冲区。如果在一个空的缓存区上尝试执行take，线程会阻塞直到一个项目可用；如果在一个满的缓存区执行put，线程会阻塞直到空间可用。我们希望将阻塞的put线程与take线程移入一个单独的等待集合中，以便我们可以使用当项目或者空间可用时仅通知一个线程的优化。可以通过使用2个Condition实例来实现。
```
class BoundedBuffer {
   final Lock lock = new ReentrantLock();
   final Condition notFull  = lock.newCondition(); 
   final Condition notEmpty = lock.newCondition(); 

   final Object[] items = new Object[100];
   int putptr, takeptr, count;

   public void put(Object x) throws InterruptedException {
     lock.lock();
     try {
       while (count == items.length)
         notFull.await();
       items[putptr] = x;
       if (++putptr == items.length) putptr = 0;
       ++count;
       notEmpty.signal();
     } finally {
       lock.unlock();
     }
   }

   public Object take() throws InterruptedException {
     lock.lock();
     try {
       while (count == 0)
         notEmpty.await();
       Object x = items[takeptr];
       if (++takeptr == items.length) takeptr = 0;
       --count;
       notFull.signal();
       return x;
     } finally {
       lock.unlock();
     }
   }
 }
```

Condition实现可以提供和Object监视器方法不同的行为和语义，例如通知保证排序，执行通知时不需要锁定。如果一个实现提供这种专门的语义，那么实现必须记录这些特殊的语义。

注意Condition实例仅仅是普通对象，可以在synchronized语句中作为目标，并且可以调用自己的监视器等待与通知方法。获取Condition实例的监视器锁或者使用其监视器方法，与获取该Condition的关联的lock或者使用等待和信令方法没有指定关系。为了避免混淆，建议不要通过这种方式获取Condition，除非自定义实现。

### 实现注意
当等待Condition时，一般来说，做为对底层平台语义的让步，“虚假唤醒”允许发生。对大多数应用程序几乎无影响，Condition始终在循环中等待，测试正在等待的状态。实现可以移除虚假唤醒的可能性，但是建议程序员总是假定会发生，始终等待循环。

条件等待的3种形式(中断，不可中断，超时)在一些平台上的易用性和性能特征可能不同。特别的，可能很难提供这些特征且保持特定的语义，例如排序保证。此外，中断线程实际挂起的能力并不总是在所有平台都可以实现。

因此，不需要一个实现为所有3种形式的等待定义完全相同的保证或语义，也不需要支持线程中断实际挂起。

实现需要清晰的记录每个等待方法的语义和保证，当一个实现支持线程挂起中断，它必须遵守接口中定义的中断语义。

由于中断通常意味着取消，并且中断的检查通常也是不频繁的，实现可以用正常方法返回来响应中断。这样做是合适的，即便可以看出来中断发生在另外一个动作未阻塞线程后。实现需要记录这种行为。

### 内部实现
内部基于链表，对外表现的功能为FIFO。