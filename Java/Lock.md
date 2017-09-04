# Lock

## 自旋锁
让线程不断的在循环体内执行，直到循环的条件被其它线程改变而进入临界区。因为线程在不断的执行循环体，因此，响应比较快。当线程数上升时，因线程的每次执行都需要消耗CPU时间，竞争会加剧，性能会下降。适用于竞争不激烈的场景。

示例：
```
public class SpinLock {

  private AtomicReference<Thread> reference = new AtomicReference<>();

  public void lock(){
    Thread current = Thread.currentThread();
    while(!reference.compareAndSet(null, current)){
    }
  }

  public void unlock (){
    Thread current = Thread.currentThread();
    reference.compareAndSet(current, null);
  }
}
```
执行顺序不取决于进入lock的时间，非公平锁。
种类：
### TicketLock
主要解决的是执行顺序的问题，主要问题在于多核CPU。
```
public class TicketLock {
    private AtomicInteger                     serviceNum = new AtomicInteger();
    private AtomicInteger                     ticketNum  = new AtomicInteger();
    private static final ThreadLocal<Integer> LOCAL      = new ThreadLocal<>();

    public void lock() {
        int myticket = ticketNum.getAndIncrement();
        while (myticket != serviceNum.get()) {
        }
        LOCAL.set(myticket);
    }

    public void unlock() {
        int myticket = LOCAL.get();
        serviceNum.compareAndSet(myticket, myticket + 1);
    }
}
```
因为每次循环检查都需要从serviceNum中获取当前值，在高并发场景下，此处会影响到性能。
### CLHLock 
```
public class CLHLock {
    public static class CLHNode {
        private volatile boolean isLocked = true;
    }

    @SuppressWarnings("unused")
    private volatile CLHNode                                           tail;
    private static final ThreadLocal<CLHNode>                          LOCAL   = new ThreadLocal<CLHNode>();
    private static final AtomicReferenceFieldUpdater<CLHLock, CLHNode> UPDATER = AtomicReferenceFieldUpdater.newUpdater(CLHLock.class, CLHNode.class, "tail");

    public void lock() {
        CLHNode node = new CLHNode();
        LOCAL.set(node);
        CLHNode preNode = UPDATER.getAndSet(this, node);
        if (preNode != null) {
            while (preNode.isLocked) {
            }
            preNode = null;
            LOCAL.set(node);
        }
    }

    public void unlock() {
        CLHNode node = LOCAL.get();
        if (!UPDATER.compareAndSet(this, node, null)) {
            node.isLocked = false;
        }
        node = null;
    }
}
```
此类型的Lock构造了一个虚拟的链表，每个新的节点都会持有上个节点的句柄，每次检查都会检查上个节点的状态，进而驱动当前节点走向临界状态。这种方式，在NUMA（不同线程的上下文分布在不同的内存物理区域）的CPU架构下会存在性能损耗。
### MCSLock
```
public class MCSLock {
    public static class MCSNode {
        volatile MCSNode next;
        volatile boolean isLocked = true;
    }

    private static final ThreadLocal<MCSNode>                          NODE    = new ThreadLocal<MCSNode>();
    @SuppressWarnings("unused")
    private volatile MCSNode                                           queue;
    private static final AtomicReferenceFieldUpdater<MCSLock, MCSNode> UPDATER = AtomicReferenceFieldUpdater.newUpdater(MCSLock.class, MCSNode.class, "queue");

    public void lock() {
        MCSNode currentNode = new MCSNode();
        NODE.set(currentNode);
        MCSNode preNode = UPDATER.getAndSet(this, currentNode);
        if (preNode != null) {
            preNode.next = currentNode;
            while (currentNode.isLocked) {

            }
        }
    }

    public void unlock() {
        MCSNode currentNode = NODE.get();
        if (currentNode.next == null) {
            if (UPDATER.compareAndSet(this, currentNode, null)) {

            } else {
                while (currentNode.next == null) {
                }
                currentNode.next.isLocked = false;
                currentNode.next = null;                
            }
        } else {
            currentNode.next.isLocked = false;
            currentNode.next = null;
        }
    }
}
```
此类型的Lock构造了一个真实的链表，每个新的节点都会被上个节点持有句柄，当上个节点释放锁时，会通知下个节点。此时每个节点仅需要检查自己的状态，不需要访问其它线程的资源。相比上个类型的锁，性能上会有较大提升。

### LockSupport
基础的线程阻塞原语，用于创建lock或者其它的同步对象。
主要包括2类方法。
#### park
返回情况如下：
    1.  其它thread以当前线程作为目标调用unpark；
    2.  其它thread中断了当前线程的执行；
    3.  未知的任意其它原因导致的返回；
park调用在permit有效时，会立即返回，否则会阻塞线程。但是因为情况3的存在，故必须在重新检查返回条件的循环里调用此方法。
#### unpark
unpark调用在permit无效时，使得permit有效。

// 返回提供给最近一次尚未解除阻塞的park方法调用的blocker对象，如果该调用不受阻塞，则返回null。
static Object getBlocker(Thread t)
// 为了线程调度，禁用当前线程，除非许可可用。
static void park()
// 为了线程调度，在许可可用之前禁用当前线程。
static void park(Object blocker)
// 为了线程调度禁用当前线程，最多等待指定的等待时间，除非许可可用。
static void parkNanos(long nanos)
// 为了线程调度，在许可可用前禁用当前线程，并最多等待指定的等待时间。
static void parkNanos(Object blocker, long nanos)
// 为了线程调度，在指定的时限前禁用当前线程，除非许可可用。
static void parkUntil(long deadline)
// 为了线程调度，在指定的时限前禁用当前线程，除非许可可用。
static void parkUntil(Object blocker, long deadline)
// 如果给定线程的许可尚不可用，则使其可用。
static void unpark(Thread thread)

三种形式的 park 还各自支持一个blocker对象参数。此对象在线程受阻塞时被记录，以允许监视工具和诊断工具确定线程受阻塞的原因。在线程dump的时候，可以看到具体的阻塞对象。
