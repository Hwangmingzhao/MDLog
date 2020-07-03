### Java.Util.Concurrent

Atomic

Lock



#### volatile

- **轻量级**同步机制

  - 保证可见性
  - 不保证原子性
  - 禁止指令重排，保证有序性

- JMM(java内存模型)，规定每个线程由其工作内存，这是一份主内存的拷贝，更新之后再写回去，不能直接操作主内存所以线程之间的工作内存是不能相互访问的。以下是**几个JMM的特性**

  - 可见性：线程修改工作内存并写会主内存的时候，其他线程需要被通知数据已经发生改变。
  - 原子性：某个线程正在做某个具体业务时，中间不可以被加塞或者分割，要么同时成功要么同时失败。
  - 有序性：多线程环境下线程交替执行，由于编译器优化重排的存在（这个重排只会考虑一个线程内数据依赖性，而不会考虑线程之间是否发生数据依赖），所以重排会导致最终结果并不唯一，结果无法预测。
  
- 规定：

  - 线程加锁前，必须读取主内存的最新版本到工作内存。
  - 线程解锁前，必须把共享变量的值刷新回主内存
  - 加锁解锁是同一把锁
  
  对变量使用volatile后，一个线程更改他之后，其他线程能知道他被改了。而且不能保证多个线程对同一个变量的操作是原子性的，这是线程不安全，num++在多线程下是非线程安全的，可以使用Atomic Integer来实现原子性。

底层原理是在**写**一个使用volatile修饰的变量时，会在其后插入一条store，使其强制刷新回主内存（实现可见性），在**读**前加入一条load,使其强制从内存中读（实现禁止指令重排）。

**volatile的使用**：

- 多线程下实现单例模式可能会调用多次构造方法（理想是只有第一次新建一个静态对象时才执行一次），可以使用synchronized锁函数。
- 对volatile变量的操作不会造成阻塞, 而``synchronized``会出现阻塞
- 关键字``volatile``解决的下变量在多线程之间的可见性；而``synchronized``解决的是多线程之间资源同步问题






### ThreadLocal

恰好与volatile相反，每个线程内都有变量，但值是线程独有的。

这是一个通过继承了WeakReference的Entry实现的。

每个ThreadLocal里面都有一个Map,里面又包含多个Entry,Entry 是一个包含 key 和 value 的一个对象，**Entry的key为ThreadLocal，value为ThreadLocal对应的值，**只不过是对这个Entry做了一些特殊处理，即 使用 `WeakReference<ThreadLocal>`将 `ThreadLocal`对象变成一个弱引用的对象，这样做的好处就是在线程销毁的时候，对应的实体就会被回收，不会出现内存泄漏。

### CompareAndSwap（CAS）

是一条CPU并发原语，保证原子性。

**Atomic Integer**底层实现原子性原理：使用了unsafe类，里面的函数基本都是native函数比如CAS，保证原子性，调用操作系统底层资源访问内存。

  

### ABA

Atomic Integer的使用会有一个问题，其底层实现机制是比较期望值与当前主内存中的值是否相等，相等采去做修改，但是问题可能是，在你去做判断之前，内存中的值已经被修改过多次，但是最后一次又改回原来的样子。

但是这样就不是绝对的原子性了。

可以使用AtomicStamReference解决。



###　集合线程不安全问题

不安全的原因在于非原子性操作，其实很好理解；



### 对于集合类的并发写不安全：我们有几种方式解决

```java
new Vector();

//将list变为同步的
List<String> list = Collections.synchronizedList(new ArrayLis<>());

//第三种：使用CopyOnWriteXXX类
List<String> list = newCopyOnWriteList();

//同理，HashSet线程不安全也可以按照第二第三种方法解决

//但是HashMap也是线程不安全的，解决方法有差异
Map<String,String> map = ConcurrentHashMap<>();

```



















