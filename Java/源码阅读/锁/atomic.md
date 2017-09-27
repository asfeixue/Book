# atomic

一个小的工具包，支持单个变量上的无锁线程安全编程。实际上，这个包将volatile值，字段，数组元素等概念扩展并基于如下表达式提供原子条件更新操作：
```
   boolean compareAndSet(expectedValue, updateValue);
```

这种方法(在不同类中参数类型不同)原子的设置一个变量为updateValue，如果其当前持有的是expectedValue，成功返回true。这个package下的类还包括获取和无条件设置值的方法，以及下面描述的弱条件原子更新操作weakCompareAndSet。

这些特殊的方法实现可以采用当代处理器上可用的高效的机器级别的原子指令。然而在某些平台，支持可能需要某种形式的内部锁定。因此，这些方法不是严格保证是非阻塞的 —— 线程可能在执行操作前被阻塞。

类 AtomicBoolean, AtomicInteger, AtomicLong, 和 AtomicReference的实例都提供相对应类型的单个变量的访问与更新。每个类型还提供了适用于该类型方法的实用方法。例如，类AtomicLong and AtomicInteger提供原子增量方法。如下是一个生成序列号应用：
```
class Sequencer {
   private final AtomicLong sequenceNumber
     = new AtomicLong(0);
   public long next() {
     return sequenceNumber.getAndIncrement();
   }
 }
```

原子访问和更新操作的内存效果通常和volatile一致。
- get 和读取volatile变量有同样的内存效果
- set 和写入(分配)volatile变量有同样的内存效果
- lazySet 和写入(分配)volatile变量有同样的内存效果，除了允许对后续(不是之前)的内存操作重排列外，其本身不会对非volatile写入添加重排序限制。在其它的使用上下文中，lazySet适用于垃圾收集，清空那些不在被使用的引用。
- weakCompareAndSet 原子的读取并且有条件的写入一个变量，但是不会产生任何happens-before排序，所以不提供weakCompareAndSet 目标之外的任何变量的读取和写入的相关保证。
- compareAndSet与其它所有的读更新操作如：getAndIncrement和volatile变量在读写上有同样的内存语义。

除了代表单一值的类，这个package还包含Updater类，可用于在任何选定的类上的选定的volatile字段获取compareAndSet操作。AtomicReferenceFieldUpdater, AtomicIntegerFieldUpdater, 以及AtomicLongFieldUpdater基于反射来提供相关字段类型的访问。主要用于原子数据结构中同一节点(例如：树节点的链接)中的一些volatile字段相互独立的处理来自原子更新的影响。这些类在如何与何时使用原子更新等方面得到更大的灵活性，放弃了直接基于反射的不太方便以及较弱的保证的使用方式。

AtomicIntegerArray,AtomicLongArray,以及AtomicReferenceArray类进一步将原子操作扩展到支持这些类型的数组。值得注意的是，这些类在其数组元素上提供volatile访问语义，在普通的数组上是不支持的。

原子类同时也支持weakCompareAndSet方法，其应用范围有限。在一些平台上，正常情况下，weak版本可能比compareAndSet有更好的效果，但不同之处在于任何给定的weakCompareAndSet方法调用都可能返回虚假的错误(也就是说，没有明显的原因)。错误返回意味如果需要，操作可以重试，依赖于当前变量保持expectedValue时重复调用并且没有其它线程尝试对变量设置成功。(这样的虚假失败可能是由于预期值与当前值不一致内存争用效应)另外weakCompareAndSet不提供同步控制通常需要的排序保证。然而当更新与程序中其它的排序无happens-before约束时，方法对于更新计数器与统计数据有用。当线程看到由weakCompareAndSet引起的原子变量更新时，其不需要知道任何其它变量的之前的更新。这在单一使用情况下是可以接受的，比如如下场景：更新性能统计数据。

AtomicMarkableReference类将一个boolean关联到一个引用上。例如：这个位可以用于一个数据结构中，代表引用的对象是否被逻辑删除。AtomicStampedReference类将一个Integer关联到一个引用上。可能的使用场景例如：代表一系列更新的版本号。

Atomic类主要设计用于实现非阻塞数据结构及相关基础设施类的构建块。compareAndSet方法不是锁的一般替代。适用于一个对象仅限于单个变量的关键更新。

Atomicl类不是java.lang.Integer以及相关类的通用替代品。其未定义hashCode与compareTo方法。(因为原子变量的值是不可预知的，它们不是hash table keys的好选择。)

此外，只提供在预期应用中通常有用的类。例如：没有表示字节的原子类。在那些偶尔需要这样做的情况下，可以使用AtomicInteger，byte值做适当转换来保存。还可以用Float.floatToIntBits(float)来保存floats以及Float.intBitsToFloat(int)转换为float，doubles可以用Double.doubleToLongBits(double)保存以及Double.longBitsToDouble(long)转换double。