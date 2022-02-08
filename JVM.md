### 你能保证 GC 执行吗？

不能，虽然你可以调用 System.gc() 或者 Runtime.gc()，但是没有办法保证 GC的执行



### 直接内存

Java1.8后，类的元数据放入nativememory, 字符串池和类的静态变量放入 java 堆中， 这样可以加载多少类的元数据就不再由MaxPermSize 控制, 而由系统的实际可用空间来控制。



### 永久代和方法区之间的联系

这里的 “PermGen space”其实指的就是方法区。不过方法区和“PermGen space”又有着本质的区别。**前者是 JVM 的规范，而后者则是 JVM 规范的一种实现**，并且只有 **HotSpot 才有 “PermGen space”**，而对于其他类型的虚拟机，如 JRockit（Oracle）、J9（IBM） 并没有“PermGen space”。由于方法区主要存储类的相关信息，所以对于动态生成类的情况比较容易出现永久代的内存溢出。最典型的场景就是，在 jsp 页面比较多的情况，容易出现永久代内存溢出。



## 两种GC:MinorGC（先）/FullGC（后）



### 什么时候触发MinorGC

#### （1） eden区满

即申请一个对象时，发现eden区不够用，则触发一次MinorGC。

#### （2）在FullGC触发之前

就是在重度垃圾回收之前先进行一次轻度的回收



### 什么时候会触发FullGC

除直接调用System.gc外，触发Full GC执行的情况有如下四种。

#### （1）旧生代空间不足

旧生代空间只有在新生代对象转入及创建为大对象、大数组时才会出现不足的现象，当执行Full GC后空间仍然不足，则抛出如下错误：java.lang.OutOfMemoryError: Java heap space

为避免以上两种状况引起的FullGC，**调优时应尽量做到让对象在Minor GC阶段被回收**、让对象在新生代多存活一段时间及不要创建过大的对象及数组。

#### （2） 永久代空间满（1.8后就不叫永久代了哦）

永久代中存放的为一些class的信息等，当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用CMS GC的情况下会执行Full GC。如果经过Full GC仍然回收不了，那么JVM会抛出如下错误信息：

java.lang.OutOfMemoryError: PermGen space

**为避免Perm Gen占满造成Full GC现象，可采用的方法为增大Perm Gen空间或转为使用CMS GC。**

#### （3）CMS GC时出现promotion failed和concurrent mode failure（还是旧生代空间不足的原因）

对于采用CMS进行旧生代GC的程序而言，尤其要注意GC日志中是否有promotion failed和concurrent mode failure两种状况，当这两种状况出现时可能会触发Full GC。

promotion failed是在进行Minor GC时，**survivor space放不下、对象只能放入旧生代，而此时旧生代也放不下造成的；**concurrentmode failure是在执行CMS GC的过程中同时**有对象要放入旧生代，而此时旧生代空间不足**造成的。

应对措施为：增大survivorspace、旧生代空间或调低触发并发GC的比率

#### （4）统计得到的Minor GC晋升到旧生代的平均大小大于旧生代的剩余空间

这是一个较为复杂的触发情况，Hotspot为了避免由于新生代对象晋升到旧生代导致旧生代空间不足的现象，在进行Minor GC时，做了一个判断，如果之前统计所得到的Minor GC晋升到旧生代的平均大小大于旧生代的剩余空间，那么就直接触发Full GC。

例如程序第一次触发MinorGC后，有6MB的对象晋升到旧生代，那么当下一次Minor GC发生时，首先检查旧生代的剩余空间是否大于6MB，如果小于6MB，则执行Full GC。

当新生代采用PSGC时，方式稍有不同，PS GC是在Minor GC后也会检查，例如上面的例子中第一次Minor GC后，PS GC会检查此时旧生代的剩余空间是否大于6MB，如小于，则触发对旧生代的回收。除了以上4种状况外，对于使用RMI来进行RPC或管理的Sun JDK应用而言，默认情况下会一小时执行一次Full GC。可通过在启动时通过- java-Dsun.rmi.dgc.client.gcInterval=3600000来设置Full GC执行的间隔时间或通过-XX:+ DisableExplicitGC来禁止RMI调用System.gc





### 对象分配规则

（1）对象**优先分配在Eden区**，如果Eden区没有足够的空间时，虚拟机执行一次Minor GC。

（2）**大对象直接进入老年代**（大对象是指需要大量连续内存空间的对象）。这样做的目的是避免在Eden区和两个Survivor区之间发生大量的内存拷贝（新生代采用复制算法收集内存）。

（3）长期存活的对象进入老年代。虚拟机为每个对象定义了一个年龄计数器，如果对象经过了1次Minor GC那么对象会进入**Survivor区**，**之后每经过一次Minor GC那么对象的年龄加1**，直到达到阀值对象进入老年区。

（4） 动态判断对象的年龄。如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代。（真实……）

（5） 空间分配担保。每次进行Minor GC时，JVM会计算Survivor区移至老年区的对象的平均大小，如果这个值大于老年区的剩余值大小则进行一次Full GC，如果小于检查HandlePromotionFailure设置，如果true则只进行Monitor GC,如果false则进行Full GC





## 分割线

其中与调优及debug相关的重要参数是-Xmx -Xms表示JVM运行时最大初始内存及最大内存

另外可以控制关闭手动gc，还可以指定运行模式，有-Xint （interpret） -Xcom -     Xmix（混合模式）



> https://blog.csdn.net/wang379275614/article/details/78471604



在1.8中，去掉了1.7中原有的永久区，取而代之的是元数据空间metaspace，这个空间保存在电脑本地内存中而不是JVM中

在new一个大对象的时候，1.8会将其放入老年区中



jstat -class（查看堆内存使用情况） -compiler -gc（查看各个区的使用情况）

jmap -histo -heap

使用jmap将运行信息打包成XX.dat文件，再使用jhat分析文件

jstack PID 查看一个进程中各个线程的状态

运行态 阻塞态 等待（wait join park）超时等待 终止态

监控远程tomcat 需要配置JMX



**系统一点来整理一下**：

三种参数类型：标配参数、X参数、**XX参数**

X参数：-Xint 解释执行（interpret） -Xcomp 第一次使用就编译成本地代码    -Xmix混合模式

**XX参数：**

- Boolean   -XX:+表示开启  -XX:-表示关闭
- 值类型  -XX:key=value



如果要查看一个正在运行中的程序的某项jvm参数是否开启或者获取他的值：

jps -l查看当前java进程

jinfo -flags  -PID 查看运行中所有参数

jinfo -flag 参数名 -PID 查看运行中某个参数



打印运行时参数  -XX:PrintFlagsFinal

打印初始化(刚安装jdk的)参数： -XX:+PrintFlagsInitial



常用参数：

- -Xms：初始值
- -Xmx：最大值
- -Xss 设置单个线程的栈空间大小，0表示是默认的初始值，一般是512K-1024K
- -Xmn   设置年轻代大小
- -XX:MetaspaceSize   设置元空间大小，使用的是**本地内存**
- -XX:SurvivorRatio  年轻代中eden区跟两个Survivor区的大小占比，默认是8，表示8:1:1 
- -XX:NewRatio   年轻代跟老年代的比例，如果设置为N，则表示 年轻代:老年代  =  1:N  
- -XX:MaxTenuringThreshold:  设置垃圾最大年龄，默认15。 如果在S0/S1中转换超过15次依然存货的话就移到老年区。



### 垃圾回收

**引用计数法**：一个对象没有被引用时就被回收，缺点时不能解决循环引用 和 浪费资源

**标记清除法**：在运行过程中，程序会暂停，执行标记和清除，这里有一个概念是根对象，会将根对象可达的标记为1，即不可被清除。其余的标记为0，然后被清除，，，问题有效率比较低，回收的内存空间碎片化

**标记压缩算法**：为了解决标记清除法碎片化的问题，将存活的对象压缩到内存的一端（移动对象）

**复制算法**：内存一分为二，一次只使用其中一半，GC时将存活对象复制到另一半（而且是在内存的一端），最后清除原来的那一半

复制算法在1.8的应用：在年轻代中，分为Eden区和两个Survivor区，在GC时，Eden区的存活对象会转移到To区，From区中的较年轻的对象会移动到To区，较老的对象会转移到老年代中

在JVM中，**年轻代适合使用复制算法，老年代适合使用标记清除或标记压缩算法（垃圾对象较少）**



**CMS垃圾收集器 ConcurrentMarkSweep**（针对老年代）:两次标记根节点（此时其他线程也暂停STW），两次标记后清理调整压缩（并行）



**G1垃圾收集器（重点）**：分为三步：开启垃圾收集器，设置堆的最大内存，设置最大停顿时间

有三种垃圾回收模式：YoungGC，MixedGC ，FullGC

https://blog.csdn.net/coderlius/article/details/79272773

  



### OOM SOFE

OOM有几种情况：

- java heap space:  堆内存不足，对象过多过大
- GC overhead limit exceeded: GC回收时间过长，GC多次工作但没有什么效果
- Direct buffer memory: 干翻本机内存，跟NIO有关，因为JVM的堆里都是引用，占用空间很小，但实际对象在内存中，所以内存爆掉。
- unable to create new native thread:不能创建更多本地线程，超过线程上限，主要跟操作系统平台有关。
- Meta space:元空间（方法区，共享  ，1.8后替代永久代）包含的信息有类信息，常量池，静态变量。  这个错误就是上述内容过多导致的。

![1583807310263](C:\Users\H\AppData\Roaming\Typora\typora-user-images\1583807310263.png)



#### JVM调优命令（JDK的）

- jps [option][hostid]  虚拟机进程状况工具
  - -q:只显示PID
  - jps -l ：输出应用程序main class的完整package名或者应用程序的jar文件完整路径名 
  - jps -m：输出传递给main 方法的参数 
  - jps -v：输出传递给JVM的参数 
- jstat 虚拟机统计信息统计工具
  - -**gc ：监控堆的状况，包括新生代，老年代，永久代的容量和gc时间**
  - **-class 监控类装载卸载数量及耗时**
  - -gcnew 新生代gc情况
  - -gcold
- jinfo [option] PID : 查看和调整虚拟机参数
  - -sysprops 　　  把虚拟机进程System.getProperties()的内容打印出来 
- jmap [option] PID：java内存映像工具
  - -dump   生成转储快照
  - -histo  显示堆中对象的详细信息，跟上面的 jstat -class不同
  - -heap 堆的详细信息
  - -finalizerinfo 输出等待执行finalize方法的队列中的对象
- jhat 分析导出的堆的转储快照
  - 解析这个文件后会启动一个WebServer，端口号是7000
- jstack [option]  vmid  堆栈跟踪工具 ，**可以分析CPU消耗过大的原因，还有死循环死锁等问题**
  - -F 当正常输出的请求不被相应时，强制输出线程堆栈
  - -l 除堆栈外，显示关于锁的附加信息
  - -m 如果调用到本地方法native的话，可以显示C/C++的堆栈 
  - 使用方法：
    1. 首先找到cpu使用情况，先找到进程占用比较高的，然后使用 top -Hp PID找到高占用的线程。
    2. 然后使用jstack ，接着在输出的信息中根据上一步的线程号找到对应的nid，可以看到当前线程的运行状态以及执行到什么方法
    3. 过一段时间再做一遍，如果还是执行这个方法，那么就很有可能是这里卡住了





### 字节码插桩技术