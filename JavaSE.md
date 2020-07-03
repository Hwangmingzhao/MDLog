#### 不同编码下中文所占字节数：

- 采用ISO8859-1编码方式时，一个中文字符与一个英文字符一样只占1个字节；
- 采用GB2312或GBK编码方式时，一个中占2个字节；
- 而采用UTF-8编码方式时，一个中文字符会占3个字节



####  小数如果不加 f 后缀，默认是double类型。double转成float向下转换，意味着精度丢失，所以要进行强制类型转换。 



#### Object只有以下方法，object类是所有类的超类  
getClass(),

**hashCode()**,包装类和String类已经对这两个方法进行了重写，都是比较的内容，equals对于基础数据类型是比较值，对于**一般的**类的引用等价于 == ，即判断引用是否相同

 **equals(),**  

clone(), 

toString(), 

notify(), notifyAll(),  wait(), 

finalize()  GC时调用



#### 数组复制最快的方法

System.arraycopy()



#### 迭代遍历数组时安全删除方法

iterator  支持从源集合中安全地删除对象，只需在 Iterator 上调用 remove() 即可。

这样做的好处是可以避免 **Concurrent Modified Exception** 因为List本身并不线程安全，当打开 Iterator 迭代集合时，同时又在对集合进行修改。有些集合不允许在迭代时删除或添加元素，但是调用 Iterator 的remove() 方法是个安全的做法。 



#### 对象的序列化

只保存了对象的状态，而静态变量保存了类的状态。

因此对于一个对象如果改变了对象属性和静态属性，序列化后静态变量不会被改变。



#### spring MVC与struts2的区别

- spring mvc的入口是servlet，而struts2是filter。



#### JDBC Statement

- JDBC提供了Statement、PreparedStatement 和 CallableStatement三种方式来执行查询语句，其中 Statement 用于通用查询， PreparedStatement 用于执行参数化查询，而 CallableStatement则是用于存储过程.
- 对于PreparedStatement来说，数据库可以使用已经编译过及定义好的执行计划，由于 PreparedStatement 对象已预编译过，所以其执行速度要快于 Statement 对象”。
- PreparedStatement可以阻止常见的SQL注入式攻击。



#### 参数传递

java中引用类型的实参向形参的传递，只是传递的引用，而不是传递的对象本身。

就是说你传递一个对象给一个函数，实际上将原来的引用复制了一份，也指向原来的对象，但是对这个引用的操作会(我觉得会啊)影响调用者的那一份数据。



#### JAVA的三大特性

封装：数据被封装到类的内部，不允许外部程序直接访问而是要通过提供的方法去访问和修改

继承：类与类之间的关系，父类拥有通性，子类拥有特性，好处是代码可以复用

多态： 

**方法重载**都是编译时多态。根据实际参数的数据类型、个数和次序，Java在编译时能够确定执行重载方法中的哪一个。

​    **方法重写**表现出两种多态性，当对象引用本类实例时，为编译时多态，否则为运行时多态。比如，父类引用子类对象，那么实际调用方法时如果子类有重写那就运行子类否则运行父类。如果存在了向上转型，那么就算在父类中也调用了父类中原有的方法都会





#### 方法重写

方法名相同，参数类型相同

子类返回类型**等于**父类方法返回类型，

子类抛出异常小于等于父类方法抛出异常，（异常本身或子类）

子类访问权限大于等于父类方法访问权限。（更容易被访问），而私有方法并没有重写的概念，因此对于私有方法并没有运行时多态的概念



#### 关于泛型

- 虚拟机中没有泛型，只有普通类和普通方法
- 所有泛型类的类型参数在编译时都会被擦除



#### 小数取整

- Math.floor()   表示向下取整，返回double类型   （floor---地板）

- Math.ceil()   表示向上取整，返回double类型    （ceil---天花板）

- Math.round()  四舍五入，返回int类型
- **取模运算，余数的符号跟被除数符号相同**





#### Servlet生命周期

1. init()：仅执行一次，负责在装载Servlet时初始化Servlet对象

   初始化阶段：Servlet启动，会读取配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象，将ServletConfig作为参数来调用init()

2. service() ：核心方法，一般HttpServlet中会有get,post两种处理方式。在调用doGet和doPost方法时会构造servletRequest和servletResponse请求和响应对象作为参数。

3. destory()：在停止并且卸载Servlet时执行，负责释放资源



#### 关于四种内部内的访问规则

![img](https://uploadfiles.nowcoder.com/images/20180701/3807435_1530425536125_D49BCBCCF82CF58C566E12F1E3130070)

局部内部类就是方法里面有一个类，里面只能访问这个方法里面的final修饰的常量和形参， 从JDK8.0开始，还可以访问所在方法的实际上的最终变量或参数（没有被final修饰但只进行了一次赋值的变量或参数） 



#### 关键字列表 (依字母排序 共50组)：

abstract, assert, boolean, break, byte, case, catch, char, class, **const**（保留关键字）, continue, default, do, double, else, enum, extends, final, finally, float, for, **goto**（保留关键字）, if, implements, import, instanceof, int, interface, long, native, new, package, private, protected, public, return, short, static, strictfp, super, switch, synchronized, this, throw, throws, transient, try, void, volatile, while



#### JRE判断程序结束

前台线程执行完毕，与之相对的是后台线程即守护线程

1、后台线程不会阻止进程的终止。属于某个进程的所有前台线程都终止后，该进程就会被终止。所有剩余的后台线程都会停止且不会完成。
2、可以在任何时候将前台线程修改为后台线程，方式是设置Thread.IsBackground 属性。
3、不管是前台线程还是后台线程，如果线程内出现了异常，都会导致进程的终止。（GC报错）

4、托管线程池中的线程都是后台线程，使用new Thread方式创建的线程默认都是前台线程



#### 关于继承

1.类与类之间的关系为继承，只能单继承，但可以多层继承。

> ```
> 在一个子类被创建的时候，首先会在内存中创建一个父类对象，然后在父类对象外部放上子类独有的属性，两者合起来形成一个子类的对象。所以所谓的继承使子类拥有父类所有的属性和方法其实可以这样理解，子类对象确实拥有父类对象中所有的属性和方法，但是父类对象中的私有属性和方法，子类是无法访问到的，只是拥有，但不能使用。就像有些东西你可能拥有，但是你并不能使用。所以子类对象是绝对大于父类对象的，所谓的子类对象只能继承父类非私有的属性及方法的说法是错误的。可以继承，只是无法访问到而已。
> ```

 2.类与接口之间的关系为实现，既可以单实现，也可以多实现。

 3.**接口与接口之间的关系为继承，既可以单继承，也可以多继承。在继承接口时不要求实现。**



#### 关于转型

`Interface i = new InterfaceImpl();`这里的引用只能调用接口内原有的方法，新增的方法不能用，同理，继承也一样。



#### 关于this

this不能在static的方法中使用~



#### 关于集合接口

集合接口分为：Collection和Map，list、set实现了Collection接口



#### 几种字符串

String StringBuffer StringBuilder都是final类，都是不可以被继承的。

final用在类上就是不能继承，用在成员上就是常量



####　native方法是由非JAVA语言编写的



0



#### 关于几种JAVA

**javac.exe是编译.java文件**

java.exe是执行编译好的**.class文件**(可以被执行)

javadoc.exe是生成Java说明文档

jdb.exe是Java调试器（debug）

javaprof.exe是剖析工具



####　GC回收线程

进入DEAD的线程，它还可以恢复，GC不会回收。

线程其实也是一个对象，真正宣布一个对象死亡，至少需要经历2次标记过程。当第一次标记时会同时进行一次筛选(判断此对象是否有必要执行finalize方法)。如果对象没有覆盖该方法，就面临死亡。



#### 关于流

按照流是否直接与特定的地方（如磁盘、内存、设备等）相连，分为节点流和处理流两类。

- 节点流：可以从或向一个特定的地方（节点）读写数据。如FileReader.
- 处理流：是对一个已存在的流的连接和封装，通过所封装的流的功能调用实现数据读写。如BufferedReader.处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为流的链接。



#### 如果一个对象引用为空

会加载静态方法，虽然这是空指针，但是指针的作用是指向对象，那静态方法跟对象就没有关系，所以静态成员和函数是可以访问的。



#### 重写和重载的多态表现

- 编译时多态：就是编译时我就知道你应该执行同一个函数的哪个函数体。

  - 方法重载都是编译时多态，在编译时看到你的参数类型数量就知道你要用哪一个实现了。

- 运行时多态：与上面反之，就是运行的时候去找实现。

  - 方法重写，父类引用指向子类对象，对于某一个方法，如果子类有实现就用子类的，如果子类没有重写就用父类的实现。

     程序运行时，Java从**实例所属的类（new 出来的类）开始寻找匹配的方法执行**，如果当前类中没有匹配的方法，则沿着**继承关系逐层向上，依次在父类或各祖先类中寻找匹配方法**，直到Object类。 

     简而言之，对于一个类的引用，只能执行他自己有的实例方法（包括从父类继承过来的），而且是从最年轻的往回找，找到就执行。

    但静态方法，你是什么类型的引用就用哪个类的实现，因为当你声明一个引用Person p时，在编译器就绑定了对应的静态函数。

- 多态的作用：
  - 提高可重用性
  - 扩展代码模块



#### 如何在迭代的同时删除List的元素

使用迭代器的remove方法，可以安全地删除List的元素。



#### JAVA鲁棒性特点

- java能检查程序在编译和运行时的错误
- java自己操纵内存减少了内存出错的可能性
- java还实现了真数组，避免了覆盖数据的可能



#### 反射

`Method[] getDeclaredMethods()` 返回 Class 对象表示的类或接口的**所有已声明的方法**数组，但是不包括从父类继承和接口实现的方法。

`Method[] getMethods() `返回当前 Class 对象表示的类或接口的所有**公有成员方法**对象数组，包括已声明的和从父类继承或实现接口的方法。



#### 一些细节

- 构造方法不需要同步
- 子类可以重写父类的同步方法



#### Java体系结构

java编程语言

java类文件格式

java API:它是运行库的集合，提供一套访问系统主机资源的标准方法

jvm



#### 基本运算

多种基本类型的运算，会将类型小的转型为大的执行计算

char < short < int < float < double  不同类型运算结果类型向右边靠齐。

**Java中的byte，short，char进行计算时都会提升为int类型。**就是说两个byte类型相加，其实就是两个int相加，所以结果也是int，因此不能直接付给byte，因为不能自动向下转型。



#### wait()   sleep()  yield()  join()

wait()放弃锁

join()底层调用wait()，因此也会释放锁

sleep()不放弃锁:

-  把cpu的时间片交给其他线程，但是并没有指定把CPU的时间片接下来到底交给哪个线程，而是让这些线程自己去竞争（这里的线程并不会有锁的限制，都可以在这段时间内执行各自的任务） ,就算线程的睡眠时间到了，他也不是立即会被运行，只是从睡眠状态变为了可运行状态，是不会由睡眠状态直接变为运行状态的 

yield()让当前正在运行的线程回到可运行状态，以**允许具有相同优先级**的其他线程获得运行的机会，也不放弃锁



#### HashTable HashMap异同

> ①继承不同。
>
> ```
> public class Hashtable extends Dictionary implements Map public class HashMap extends AbstractMap implements Map
> ```
>
> ②
>
> Hashtable 中的方法是同步的，而HashMap中的方法在缺省情况下是非同步的。在多线程并发的环境下，可以直接使用Hashtable，但是要使用HashMap的话就要自己增加同步处理了。
>
> ③
>
> Hashtable中，key和value都不允许出现null值。
>
> 在HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。当get()方法返回null值时，即可以表示 HashMap中没有该键，也可以表示该键所对应的值为null。因此，**在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断。**
>
> ④两个遍历方式的内部实现上不同。
>
> Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。
>
> ⑤
>
> 哈希值的使用不同，HashTable直接使用对象的hashCode。而HashMap重新计算hash值，是在hashCode的基础上在计算一遍。
>
> ⑥
>
> Hashtable和HashMap它们两个内部实现方式的数组的初始大小和扩容的方式。HashTable中hash数组默认大小是11，增加的方式是 old*2+1。HashMap中hash数组的默认大小是16，而且一定是2的指数。 





#### 关于Class Loader

使用指定的二进制名称来加载类，这个方法的默认实现按照以下顺序查找类：

- 调用`findLoadedClass(String)`方法检查这个类是否被加载过

- 使用父加载器调用`loadClass(String)`方法，如果父加载器为`Null`，类加载器装载虚拟机内置的加载器调用`findClass(String)`方法装载类，这个方法通常子类重写实现。

- 如果，按照以上的步骤成功的找到对应的类，并且该方法接收的`resolve`参数的值为`true`,那么就调用`resolveClass(Class)`方法来处理

  **`ClassLoader`的子类最好覆盖`findClass(String)`而不是这个方法`classLoader`。**
  **除非被重写，这个方法默认在整个装载过程中都是同步的（线程安全的）。**

```java
//name表示类的名字，是.class
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
    /*
    首先理解这个同步代码块的意思，括号内getClassLoadingLock(name)需要传入一个锁对象，
    在这个类的构造函数执行时会判断是否有并行加载类的能力（并行加载多个不同的类而不是一个类），如果有的话那就有多个锁，并且使用parallelLockMap这个Map存起来（存的内容是一个类作为key,一个对象作为value）而且这个Map是线程安全的Concurrent的，否则的话就只能有一把锁，就直接返回this就OK
    protected Object getClassLoadingLock(String className) {
        Object lock = this;
        if (parallelLockMap != null) {
            Object newLock = new Object();
            lock = parallelLockMap.putIfAbsent(className, newLock);
            if (lock == null) {
                lock = newLock;
            }
        }
        return lock;
    }
    */
        synchronized (getClassLoadingLock(name)) {
            //首先查看这个类是否已经在被加载过的列表中
            Class c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                //如果这个类没有被加载过，会执行以下的加载过程
                //如果有父加载器，那就调他的loadClass方法
                //这样就能一级一级向上去寻找加载器
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }
                
				//如果发现上一级还是没有返回的话，那么就调用自己的
				//这样就是一级一级又往下回来
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            /*这个方法的作用是：链接指定的类。这个方法给Classloader用来链接一个类，如果这个类已经被链接过了，那么这个方法只做一个简单的返回。否则，这个类将被按照 Java™规范中的Execution描述进行链接。*/
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

**java中的类大致分为三种：**

> 1、系统类
>
> 2、扩展类
>
> 3、自定义的类

**类装载方式，有两种:**

> 1、隐式装载， 程序在运行过程中当碰到通过`new` 等方式生成对象时，隐式调用类装载器加载对应的类到`jvm`中。
> 2、显式装载， 通过`class.forName()`等方法，显式加载需要的类

**在`java`中类装载器把一个类装入`JVM`，经过以下步骤：**

> 1、装载：查找和导入Class文件
> 2、链接：其中解析步骤是可以选择的
> （a）检查：检查载入的class文件数据的正确性
> （b）准备：给类的静态变量分配存储空间
> （c）解析：将符号引用转成直接引用
> 3、初始化：**对静态变量，静态代码块执行初始化工作**

**java类装载器**:（他们各自负责加载的类的位置不同，从上到下为父到子）

- Bootstrap（libraries里的）

  前面有提到`Bootstrap Loader`是用`C++`语言写的，依`java`的观点来看，逻辑上并不存在`Bootstrap Loader`的类实体

- Extension（扩展类加载器）

- Application 应用程序（系统）类加载器

- 自定义（`AppClassLoader`类路径下的，也就是自己写的）



大致讲一下一个过程：比如说有一个类他在**类路径**下（是我自己写的），根据层次，他需要使用`AppClassLoader`加载他，而且，这个类所依赖和引用的类也由我加载。

但是在真正执行加载的时候我是一级一级往父加载器丢的，那么我引用的一些jre中的类，会一直向上找，找到BootstrapClassLoader然后被加载，那我自己写的这个类，我的父加载器们当然是没办法找到的，所有又会被父加载器向下丢回来由AppClassLoader加载进jvm。

#### 加强一下类加载机制

- **装载**：会在内存中生成一个Class文件，携带着这个类型的各种信息，这个Class的来源不一：

  - 从jar包和war包内读取
  - 动态代理生成
  - 由JSP转换而来
  - 编译生成

- **链接**：

  - 检查：检查这个Class文件的信息是否满足虚拟机的要求，检查正确性
  - 准备：为类分配内存（不是对象哦）并设置类变量的初始值，在方法区中开辟内存空间，注意这个初始值并不是代码中设定的值，一般来说是0。如果这是一个静态常量那么就会有代码中写好的初始值
  - 解析：虚拟机将常量池中的符号引用替换为直接引用的过程

- **初始化：**开始真正执行程序代码了，比如说执行静态块的代码和初始化类变量

  - 注意以下几种情况**不会执行类初始化：** 

    1. 通过子类引用父类的静态字段，只会触发父类的初始化，而不会触发子类的初始化。 

    2. 定义对象数组，不会触发该类的初始化。 

    3. 常量在编译期间会存入调用类的常量池中，本质上并没有直接引用定义常量的类，不会触发定义常量所在的类。 

    4. 通过类名获取 Class 对象，不会触发类的初始化。 也就是说静态块并不会被执行。

    5. 通过 Class.forName 加载指定类时，如果指定参数 initialize 为 false 时，也不会触发类初始化，其实这个参数是告诉虚拟机，是否要对类进行初始化。 

    6. 通过 ClassLoader 默认的 loadClass 方法，也不会触发初始化动作。 

    

- 使用

- 卸载





#### 主方法入口应该这样写

java中可以**有多个重载的main**方法，

只有public static void main(String[] args){}是函数入口



#### null与类方法

`((TestClass)null).testMethod();`这样是可以的，前提是`testMethod()`是类方法。

虽然被强转型的是null，null可以被强制类型转换成**任意类	（不是任意类型对象）**，于是可以通过它来执行静态方法。



#### String StringBuilder StringBuffer

效率： **StringBuilder>String**>StringBuffer(线程安全)

因为，Buffer是在Builder基础上加锁，因此效率会低。

String的拼接有两种实现类型：

- `String string = "a" + "b"+ "c";`这种在编译时确定了；

- ```java
  String a = "a";
  String b = "b";
  String c = a+b;  //c是一个堆中的对象，因为是StringBuilder对象
  //字符串的拼接底层使用StringBuilder将两个字符串拼起来再返回这个Builder的toString
  ```

  

#### 关于out of memory

### OOM SOFE

OOM有几种情况：

- java heap space:  堆内存不足，对象过多过大
- GC overhead limit exceeded: GC回收时间过长，GC多次工作但没有什么效果
- Direct buffer memory: 干翻本机内存，跟NIO有关，因为JVM的堆里都是引用，占用空间很小，但实际对象在内存中，所以内存爆掉。
- unable to create new native thread:不能创建更多本地线程，超过线程上限，主要跟操作系统平台有关。
- Meta space:元空间（方法区，共享  ，1.8后替代永久代）包含的信息有类信息，常量池，静态变量。  这个错误就是上述内容过多导致的。



#### 关于变量初始值

函数内变量必须要有初始值才可以只用，而类变量可以不负初值，基本类型是0



#### 关于try catch

我们都知道，finally块中return会覆盖try和catch中的return

但是实现过程是，首先计算try中return的值，保存起来，等待finally块执行结束后再返回。

如果finally中不返回，而又改变了try中要返回的变量的值，是不会影响最终返回的值的。



#### 继承

继承时，不可避免会遇到方法的重写。而final方法又是不能重写的。

但是如果父类的final方法是private的，那么对于子类而言，虽然是"继承了"这个方法，但是他看不到，所以如果**声明了同名方法也不算重写**。



#### 四种类型的引用

1、强引用：一个对象赋给一个引用就是强引用，比如new一个对象，一个对象被赋值一个对象。

2、软引用：用SoftReference类实现，一般不会轻易回收，只有内存不够才会回收。

3、弱引用：用WeekReference类实现，一旦垃圾回收已启动，就会回收。

4、虚引用：不能单独存在，必须和引用队列联合使用。主要作用是跟踪对象被回收的状态。



#### 命令规范的应用

System是java.lang包下的一个类，

out为System的final静态成员（PrintStream类型），

println()是PrintStream类的实例方法。

java规范中包名一般是小写的

类名一般是大写的



#### 三种代理

- 静态代理：代理对象和被代理对象实现同一个接口，代理对象接收一个被代理对象作为成员（即target），如果要调用target的某一个方法，就在代理对象的同一方法内调用target的方法，在调用的前后可以进行扩展。
  - 缺点: 因为代理对象需要与目标对象实现一样的接口,所以会有很多代理类,类太多.同时,一旦**接口增加方法,目标对象与代理对象都要维护**.
  - 如何解决静态代理中的缺点呢?答案是可以使用动态代理方式

- 动态代理（jdk代理，接口代理）：代理对象不需要实现接口，但是目标对象必须实现接口。

  - `java.lang.reflect.Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces,InvocationHandler h )`方法获得一个代理对象，这个代理对象就可以执行被代理对象实现的接口的方法
  - 使用实例：

  ```java
   public Object getProxyInstance(){
          return Proxy.newProxyInstance(
                  target.getClass().getClassLoader(),
                  target.getClass().getInterfaces(),
                  new InvocationHandler() {
                      @Override
                      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                          System.out.println("开始事务2");
                          //执行目标对象方法
                          Object returnValue = method.invoke(target, args);
                          System.out.println("提交事务2");
                          return returnValue;
                      }
                  }
          );
       
       
  //使用
  // 给目标对象，创建代理对象
  IUserDao proxy = (IUserDao) new ProxyFactory(target).getProxyInstance();
  proxy.save();
  ```

- Cglib代理（子类代理）：它是在内存中构建一个子类对象从而实现对目标对象功能的扩展，通过“继承”可以继承父类所有的公开方法，然后可以重写这些方法，在重写时对这些方法增强，这就是cglib的思想

  ```java
  
  public class MyMethodInterceptor implements MethodInterceptor{
   
      /**
       * sub：cglib生成的代理对象
       * method：被代理对象方法
       * objects：方法入参
       * methodProxy: 代理方法
       */
      @Override
      public Object intercept(Object sub, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
          System.out.println("======插入前置通知======");
          Object object = methodProxy.invokeSuper(sub, objects);
          System.out.println("======插入后者通知======");
          return object;
      }
  }
  
  
  Enhancer enhancer = new Enhancer();
  // 设置enhancer对象的父类
  enhancer.setSuperclass(HelloService.class);
  // 设置enhancer的回调对象
   enhancer.setCallback(new MyMethodInterceptor());
  // 创建代理对象
  HelloService proxy= (HelloService)enhancer.create();
  // 通过代理对象调用目标方法
  proxy.sayHello();
  
  ```

- 两种动态代理的使用：

  - 如果要被代理的对象是个实现类（即实现了接口的），那么Spring会使用JDK动态代理来完成操作（Spirng默认采用JDK动态代理实现机制）
  - 如果要被代理的对象不是个实现类那么，Spring会强制使用CGLib来实现动态代理

- 两者速度比较：在jdk1.8中，JDK代理更快了。





#### 关于一个文件能有多少个类

一个.java文件至多有一个public的**外部类**

可以有多个main方法，但是**public static void main(String[] args)**的方法只能有一个。



#### JVM内存

![1567328844808](C:\Users\H\AppData\Roaming\Typora\typora-user-images\1567328844808.png)

不存在什么堆帧的



#### JAVA中数组

数组是一个**对象**，因而会被存储在**堆**中，而且是**大小不可变**的，ArrayList底层是数组，扩容的原理也是新建一个数组。



#### 抽象类与接口的区别

1. **抽象类**可以有构造方法，接口中不能有构造方法。  

2. 抽象类中可以有普通成员变量，接口中没有普通成员变量  

3. 抽象类中可以包含非抽象的普通方法，接口中的所有方法必须都是抽象的，不能有非抽象的普通方法。  

4. 抽象类中的抽象方法的访问类型可以是public，protected和（默认类型,虽然 eclipse下不报错，但应该也不行），但接口中的抽象方法只能是public类型的，并且默认即为public abstract类型，1.8后可以是static的。  
5. 抽象类中可以包含静态方法，接口中不能包含静态方法  
6. 抽象类和接口中**都可以包含静态成员变量**，**抽象类中的静态成员变量的访问类型可以任意，但接口中定义的变量只能是public static final类型**，并且默认即为public static final类型。  
7. 一个类可以实现多个接口，但只能继承一个抽象类。  



#### 加载驱动方法

1. Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");

2. DriverManager.registerDriver(new com.mysql.jdbc.Driver());

3. System.setProperty("jdbc.drivers", "com.mysql.jdbc.Driver");



#### super，this

代表父类对应的对象，所以用super访问在子类中无法直接使用的父类成员和方法，比如子类覆盖了的方法和属性。

注意与super(),this()的区别：

1. 构造器中第一行默认是super()，一旦直接父类的构造器中没有无参的，那么必须显式调用父类的某个有参构造。
2. 构造器中第一行的super()可以换成this()，但是this()和super()只能出现一个。
3. super，this关键字与super()，this()不是一回事，前者表示当前调用者的父类与其本身，后者是为了构造器相互调用。

在子类的构造函数中，无论是有参还是无参，只要不是显示调用父类构造函数，都会隐式调用无参构造函数，因此父类如果没有无参构造函数或者子类没有调用有参构造函数都会报错。





#### JSON

属性名称必须使用括号括起来

属性与其对应值使用：分割

若是一个对象就用{}  ；若是一个值，就直接写；若是一个数组，用[]

属性与属性直接的分割使用逗号



#### 如果想要取一个浮点数的整数部分

parseInt(a);  不止可以转字符串

Math.floor(a);



#### 关于垃圾回收

在JVM,对于一个对象，一个对象到GC Roots没有任何引用链相连时就是一个可回收对象，就是说没有人用它了。可回收对象在被回收之前，JVM会判断是否有finalize方法，如果有则会调用finalize方法，在这个方法里面对象可以自救的。

**使用System.gc可以提醒VM执行GC单不能强制执行**

所以说，当对象不再被任何对象引用时，GC会调用该对象的finalize()方法。而不是被收集后，收集后了都没东西了还执行什么啊。

**对于finalize()方法：**

> 如果对象在进行可达性分析后发现没有与 GC Roots 相连接的引用链，那他将会被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行 finalize 方法。
注意：当对象没有覆盖 finalize 方法，或者 finalize 方法已经被虚拟机调用过，虚拟机将这两种情况都视为 “没有必要执行”。也就是说，finalize 方法只会被执行一次。
如果这个对象被判定为有必要执行 finalize 方法，那么这个对象将会放置在一个叫做 F-Queue 的队列之中，并在稍后由一个虚拟机自动建立的，优先级为 8 的 Finalizer 线程去执行它。
注意：如果一个对象在 finalize 方法中运行缓慢，将会导致队列后的其他对象永远等待，严重时将会导致系统崩溃。
finalize 方法是对象逃脱死亡命运的最后一道关卡。稍后 GC 将对队列中的对象进行第二次规模的标记，如果对象要在 finalize 中 “拯救” 自己，只需要将自己关联到引用上即可，通常是 this。
如果这个对象关联上了引用，那么在第二次标记的时候他将被移除出 “即将回收” 的集合；如果对象这时候还没有逃脱，那基本上就是真的被回收了。

**对于一个对象何时会被判断为是垃圾**

- 引用计数法

引用计数法就是如果一个对象没有被任何引用指向，则可视之为垃圾。这种方法的缺点就是不能检测到环的存在。

- 2.根搜索算法（可达性分析法）

根搜索算法的基本思路就是通过一系列名为”GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链(Reference Chain)，当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。



#### 使用泛型并不会提高运行性能

因为泛型不会影响java虚拟机生成的**汇编代码**，在编译阶段，虚拟机就会把泛型的**类型擦除**，还原成没有泛型的代码，顶多编译速度稍微慢一些，执行速度是完全没有什么区别的。



#### 接口的变量

在接口里面的变量默认都是public static final 的，它们是公共的,静态的,最终的常量.相当于全局常量，可以直接省略修饰符。



#### JAVA8中的Stream

可以方便开发者使用Lambda表达式，以及实现对集合的查找遍历过滤及常见计算

通过stream方法创建Stream。在聚合操作中，与Labda表达式一起使用，显得代码更加的简洁。这里值得注意的是，我们首先是stream方法的调用，其与iterator作用一样的作用一样，**该方法不是返回一个控制迭代的 Iterator 对象，而是返回内部迭代中的相应接口： Stream**，其一系列的操作都是在操作Stream,直到feach时才会操作结果，这种迭代方式称为**内部迭代**。

使用方法:

1. 创建Stream:通过stream()方法，取得集合对象的数据集。
2. Intermediate:通过一系列中间（Intermediate）方法，对数据集进行过滤、检索等数据集的再次处理。如上例中，使用filter()方法来对数据集进行过滤。
3. Terminal通过最终（terminal）方法完成对数据集中元素的处理。如上例中，使用forEach()完成对过滤后元素的打印。

**在一次聚合操作中，可以有多个Intermediate，但是有且只有一个Terminal**

Stream的操作有**Intermediate、Terminal和Short-circuiting**：

- Intermediate  主要是用来对Stream做出相应转换及限制流，实际上是将**源Stream转换为一个新的Stream**，以达到需求效果：map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 skip、 parallel、 sequential、 unordered
  - concat方法将两个Stream连接在一起，合成一个Stream。若两个输入的Stream都时排序的，则新Stream也是排序的
  - distinct方法以达到去除掉原Stream中重复的元素，生成的新Stream中没有没有重复的元素。
  - filter方法对原Stream按照指定条件过滤，在新建的Stream中，只包含满足条件的元素，将不满足条件的元素过滤掉
  - map方法将对于Stream中包含的元素使用给定的转换函数进行转换操作，新生成的Stream只包含转换生成的元素。
  - flatMap方法与map方法类似，都是将原Stream中的每一个元素通过转换函数转换，不同的是，该换转函数的对象是一个Stream，也不会再创建一个新的Stream，而是将原Stream的元素取代为转换的Stream。flatMap传入的Lambda表达式必须是Function实例，参数可以为任意类型，**而其返回值类型必须是一个Stream。**
  - peek方法生成一个包含原Stream的所有元素的新Stream，同时会提供一个消费函数（Consumer实例），新Stream每个元素被消费的时候都会执行给定的消费函数，并且消费函数优先执行。
  - skip方法将过滤掉原Stream中的前N个元素，返回剩下的元素所组成的新Stream
- Terminal（有点像spark的操作）：forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、iterator
- Short-circuiting：
  anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit



#### 在不同情况下在方法内创建一个String

- String str = "a" ;    首先在创建一个名为str的引用，然后在**常量池**中查找有没有a，没有的话就创建一个字符串a，将引用指向它。

- String str = new String("a");     首先创建一个str的引用，然后在堆中创建一个对象并将引用指向这个内存的首地址，然后String对象会去查看**常量池是否有"a"**，有的话就指过去没有的话就新建一个

  > 关于字符串字面量通俗点解释就是，使用双引号""创建的字符串，在堆中创建了对象后其引用插入到字符串常量池中（jdk1.7后），可以全局使用，遇到相同内容的字面量，就不需要再次创建。



### 常量池

JVM中的常量池分为几种：

- class文件常量池，主要存放Class文件的信息，记录方法名描述符之类的，在编译时就生成了，因此叫做静态常量池
- 运行时常量池：当类加载到内存后，JVM就会将Class文件常量池的内容放到运行时常量池中，这个常量池位于方法区中，在1.8前位于永生代，1.8后位于元空间。
- 全局字符串常量池：在jdk7以后，这个常量池并不直接存储字符串对象本身了，而是字符串常量的引用，实际的对象在堆里



#### 当你new一个对象的时候都发生了什么

首先我们要知道一个对象在内存中是怎样布局的：

- 对象头：又分为两部分。第一部分MarkWord保存了**对象自身**的运行数据（HashCode,GC分代年龄，锁对象标记），第二部分就是类型指针 Class Metadata Address （这是哪个类的对象信息），如果这个对象是一个数组那么就会有第三部分，即这个数组的长度。
  - 第一部分MarkWord的内容不固定，会根据标志位的不同而保存不同的信息
- 实例数据：这部分就是我们运行程序时使用到的实例数据，无论是从父类继承的还是子类自身的
- 对齐填充：为了使一个对象的大小为8个字节的整数倍。

然后我们要搞清楚我们是如何访问一个对象的：

- 首先我们直接得到一个引用，这个引用指向堆中的一个对象，但是这又是怎样指向的呢：
  - 直接指针法：引用中直接存储在堆中的地址，如果对象因为GC而被移动那么这个引用也要修改
  - 句柄法：在堆中划一个部分存储句柄，引用指向这个句柄，句柄指向对象

现在开始new一个对象：

1. 首先先去看一下常量池有没有这个类的信息（忘记的话回去看常量池），然后检查这个类是否已经被类加载器加载。
2. 分配内存空间
3. 将对象初始化为零值，保证这些实例变量在不赋初值的情况下就能被使用。
4. 对对象头设置必要的信息
5. 然后执行构造方法，主要就是初始化实例数据。**类变量的初始化**操作在类加载的初始化阶段<clinit>方法完成
6. 然后这个引用被指向那个内存空间（上述两种方法）



#### ThreadLocal

线程本地变量，在使用时我们的目的一般是解决多线程问题，但是解决的方式跟锁和同步是不一样的。

1. lock 的资源是多个线程共享的，所以访问的时候需要加锁。
2. ThreadLocal 是每个线程都有一个副本，是不需要加锁的。
3. lock 是通过时间换空间的做法。
4. ThreadLocal 是典型的通过空间换时间的做法。

ThreadLocal的使用：

```java
public class Test {

    public static void main(String[] args) {
        ThreadLocal<String> local = new ThreadLocal<>();
        //设置值
        local.set("hello word");
        //获取刚刚设置的值
        System.out.println(local.get());
    }
}


//这是set方法，首先尝试获取这个ThreadLocal所在的Thread所拥有的ThreadLocalMap,
 public void set(T value) {
        Thread t = Thread .currentThread();
        // 获取线程绑定的ThreadLocalMap
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
           //第一次设置值的时候进来是这里
            createMap(t, value);
    }



```

 虽然Thread 可以引用ThreadLocalMap，但是不能调用任何ThreadLocalMap 中的方法。这也就是我们平时都是通过ThreadLocal 来获取值和设置值 。

看到这个get/set方法就感觉很像一个HashMap,其实也差不多，在第一次创建一个ThreadLocal的时候就会创建一个Map（数组实现，解决冲突的方式是开放地址法），之后·加入的ThreadLocal就会加入这个Map中。

然后这个Map中保存的键值对Entry的key（即ThreadLocal）是一个继承了弱引用的类型，因此在线程结束后会自动回收，而value则是强引用。



 #### HashMap存1w条数据，构造时传10000还会触发扩容吗

这是一个关于扩容的问题，大多数集合类都可以扩容，目的是为了保证有足够的空间来存储数据。但是很多构造方法会在初始化时设置初始大小。

而HashMap能装多少数据，计算公式是threshold * 负载因子，那这个threshold是大于构造函数参数的2的n次幂，如传入1000，那么threshold为1024，如果不传入负载因子则使用默认0.75，那么容量是1024*0.75.

本题中，在第一次扩容前的真实容量为18384*0.75。

关于resize，**扩容必须满足两个条件**：
1、 存放新值的时候 当前已有元素的个数 (size) 必须大于等于阈值

2、 存放新值的时候当前存放数据发生hash碰撞（当前key计算的hash值换算出来的数组下标位置已经存在值）

​	 根据新容量（原容量的2倍）新建数组，将旧数组上的数据（键值对）转移到新的数组中，这里包括：遍历旧数组的每个元素，重新计算每个数据在数组中的存放位置。（原位置或者原位置+旧容量），将旧数组上的每个数据逐个转移到新数组中，这里采用的是**尾插法**。 



#### 如何使用一个对象的clone方法

- 首先该对象需要实现cloneable接口，实现这个接口不代表clone是它的方法，只是作为一个标记, 标记这个类是否支持克隆
- 覆盖clone方法，将可见性改为public

如果一个类实现了Cloneable接口，Object的clone方法就返回这个对象的逐域拷贝，否则就抛出CloneNotSupportedException异常。如果实现了这个接口，**类和它所有的超类**都**无需调用构造器**就可以创建对象

但是使用时要注意，这是一个浅拷贝，如果成员类型是对象，那么就只能复制对象的引用，而无法创造一个成员对象的副本



#### 接口内方法可以有default实现

java8特性，一个接口的抽象方法如果使用default修饰符，那么就可以有默认实现



#### var

java10特性：var是Java10中新增的**局部类型变量**推断。它会根据后面的值来推断变量的类型，所以var必须要初始化。也就是说，在局部变量中，我们不必再费心去判断返回值类型了，相当于自动将var转为合适的类型



#### JVM中的线程和操作系统中的线程

两者具有直接的映射关系，你在JVM中创建一个线程，也相当于在操作系统中创建一个线程。

之前的笔记说过，java前台线程的结束代表着***java进程***的结束，后台运行的线程有

- 虚拟机线程
- 周期性任务线程
- GC线程
- 编译器线程
- 信号分发线程



#### 整理一下JVM内存

- 线程私有：
  - PC计数器
    - 跟操作系统类似，但是不是指向机器码而是**字节码**
    - 唯一一个没有OOM的区域
  - 虚拟机栈 VM Stack
    - 与线程的生命周期相同
    - 每调用一个方法会创建一个frame（学过的）
    - 保存的信息：本地变量表，操作数帧，运行时常量池的引用
  - 本地方法栈
    -  本地方法栈服务的对象是JVM执行的native方法，而虚拟机栈服务的是JVM执行的java方法 ，至于这个naive方法是用什么语言写的要根据操作系统而定
- 线程共享
  - 方法区
    
    - 运行时常量池
    
    >  方法区存放装载的类数据信息包括:
    >   (1):基本信息:
    >        1)每个类的全限定名
    >        2)每个类的直接超类的全限定名(可约束类型转换)
    >        3)该类是类还是接口
    >        4)该类型的访问修饰符
    >        5)直接超接口的全限定名的有序列表
    >   (2):每个已装载类的详细信息:
    >        1)运行时常量池:
    >          存放该类型所用的一切常量(直接常量和对其它类型、字段、方法的符
    >         号引用),它们以数组形式通过索引被访问,是外部调用与类联系及类型对
    >         象化的桥梁。它是类文件(字节码)常量池的运行时表示。(还有一种静态常量池,在字节码文件中)。
    >        2)字段信息:
    >          类中声明的每一个字段的信息(名,类型,修饰符)。
    >        3)方法信息:
    >          类中声明的每一个方法的信息(名,返回类型,参数类型,修饰符,方
    >         法的字节码和异常表)。
    >        4)静态变量
    >        5)到类 classloader 的引用:即到该类的类装载器的引用。
    >        6)到类 class 的引用: 虚拟机为每一个被装载的类型创建一个 class 实例, 用来代表这个被装载的类 
  - 堆
    
    - 保存对象实例
    - 另外在1.8后保存大对象会放到元空间中，这部分是机器内存而不是虚拟机内存
- 直接内存
  
  - 在NIO中会使用到，会使用JVM以外的内存区域



#### JVM的组成

- 类加载器：将java代码转换为字节码
- 运行时数据区：将字节码加载到内存中-》内存看上面
- 执行引擎：转换成机器码
- 本地库接口：调用机器码的代码库



#### 垃圾收集算法以及垃圾收集器

接触过的垃圾收集算法：

- 标记清除：最简单，没有被使用就收集
- 复制：分配一个区域，用到的就复制到另外一边排整齐，然后当前区域全部清除
- 标记整理：直接在同一个区域整理
- 分代收集：分代，根据不同代的特点选择不同的算法

接触过的垃圾收集器：

- 连续：
  - 单线程, 使用复制算法，并且STW，目前依然是Client模式下默认的新生代垃圾回收器，意思是在新生代中使用这种垃圾收集器。
- ParNew
  - 连续的多线程版本，也是STW
- Parallel SCavenge:
  - 前者的改进，高效
- Serial Old：
  - 连续收集器的老年代版本，使用标记整理算法
- CMS：
  - 用于老年代使用，多线程，标记清除
- G1：
  - 标记整理，可控制停顿时间，高效，体验好

垃圾收集器是运用了不**同的垃圾收集算法的组合**和根据实际情况而设计出来的的垃圾收集方案的落地



#### ConcurrentHashMap的原理

java1.8中，HashMap会进行变化，从数组链表变成红黑树，另外扩容也是重要的一个点。

jdk1.8前，在CHM中，由一个个segment组成，因此可以表述为分段锁，为什么线程安全呢：

- 每个Segment继承了可重入锁（就是进了房间也就获得了厕所的锁），每次锁住的不是整个Map而是Segment。需要注意的是，在CHM中相当于有**十六个segment**（可以改，但是一旦创建这个就不可以改），每个segment就像一个Hashmap，是数组链表或红黑树实现，理论上支持16个线程并发写，如果16个线程刚好操作不同的16个segment

1.8后，取消了segment这个概念，Node类型数组table中每一个table就是一个并发单元，有多少个table理论上就能有多高的并发度

-  计算hash，得到数组下标，如果table为空，则表示ConcurrentHashMap还没有初始化，则进行初始化操作：initTable() 

-  如果没有hash冲突就直接CAS插入 
-  如果存在hash冲突，就加锁来保证线程安全 
-  最后一个如果该链表的数量大于阈值8，就要先转换成黑红树的结构 



#### 总结一些比较生僻的集合类知识

- List
  - ArrayList: **扩容：容量*1.5+1**
  - Vector:**扩容：容量*2**
  - LinkedList: 双向循环链表
- Set:内部实现都是对应的Map
  - HashSet如果要放入自定义类必须重写Hashcode和equals方法，插入元素的过程是判断Hashcode是否相同，如果相同则插入同一个位置，再去判断内容是否相同。而我们的目的很明确，就是要防止相同内容的元素被重复放入，因此首先我们先要让他们落到同一个位置然后在比较内容的时候再剔除掉
    - 重写Hashcode是为了使内容相同但地址不同的对象落到同一个位置
    - 重写equals是为了让内容相同的元素可以被比较
- HashMap: 初始16,0.75，注意何时扩容，以及扩容时的线程不安全问题
  - HashTable:初始11，0.75，扩容为**容量*2+1**（因为保证为奇数）



#### 多线程并发

为什么实现Runnable而不是继承Thread，类是单继承的，接口是多实现的。

Callable接口怎么用，首先初始化Callable对象，向线程池提交这个对象的同时可以获取这个Callable对象的Future对象，再通过这个对象的get方法获得返回值，实际中可以使用数组保存Future对象。

- 线程的生命周期： 新建（new）-> 就绪（start）->运行（run）->阻塞 ->死亡（stop/抛异常/正常结束）



#### 面向对象的三大特性


1. **封装**

> 封装指的是属性私有化，根据需要提供setter和getter方法来访问属性。即隐藏具体属性和实现细节，仅对外开放接口，控制程序中属性的访问级别。

> 封装目的：增强安全性和简化编程，使用者不必在意具体实现细节，而只是通过外部接口即可访问类的成员。

1. **继承**

> 继承是指将多个相同的属性和方法提取出来，新建一个父类。
> Java中一个类只能继承一个父类，且只能继承访问权限非private的属性和方法。 子类可以重写父类中的方法，命名与父类中同名的属性。

> 继承目的：代码复用。

1. **多态**

> 多态可以分为两种：设计时多态和运行时多态。
> 设计时多态：即重载，是指Java允许方法名相同而参数不同（返回值可以相同也可以不相同）。
> 运行时多态：即重写，是指Java运行根据调用该方法的类型决定调用哪个方法。



#### 一个线程两次调用 start() 方法会出现什么情况？

 Java 的线程是不允许启动两次的，第二次调用必然会抛出 IllegalThreadStateException，这是一种运行时异常 



#### 线程的 run() 和 start() 有什么区别？

start() 方法用于启动线程，run() 方法用于执行线程的运行时代码。run() 可以重复调用，而 start() 只能调用一次。
底层**start()方法是使用C语言写的**，调用JVM_startThread，开启子线程，然后调用里面的run方法



#### 什么是CAS

CAS是compare and swap的缩写，即我们所说的比较交换。

CAS是一种操作系统的操作原语，可以使得一个值在被改变时不被打扰，其实就是一个锁，很多同步类中都使用了这个操作原语来完成特定的操作。

CAS是一种乐观锁。
操作包含三个操作数 ——内存位置（V）、预期原值（A）和新值(B)。 当且仅当内存地址V的值与预期值A相等时，将内存地址V的值修改为B。CAS是通过无限循环来获取数据的，若果在第一轮循环中，a线程获取地址里面的值被b线程修改了，那么a线程需要自旋，到下次循环才有可能机会执行。

 缺点：

1. ABA问题：一个线程a将数值改成了b，接着又改成了a，此时CAS认为是没有变化，其实是已经变化过了。
   解决办法：可以使用版本号标识，每操作一次version加1。在java5中，已经提供了AtomicStampedReference来解决问题 
2.  CAS机制所保证的只是一个变量的原子性操作，而不能保证整个代码块的原子性。 换言之就是这个锁锁的是一个变量



#### 为什么线程通信的方法wait(), notify()和notifyAll()被定义在Object类里？

在Java中，synchronized(对象)，任意对象都可以当作锁来使用，由于锁对象的任意性，所以这些通信方法需要被定义在Object类里。
（1）为什么wait()必须在同步（Synchronized）方法/代码块中调用？

答：调用wait()就是释放锁，**释放锁的前提是必须要先获得锁**（实际从实现原理上来说是这个锁对象的监视器），先获得锁才能释放锁。

（2）为什么notify(),notifyAll()必须在同步（Synchronized）方法/代码块中调用？

答：notify(),notifyAll()是将锁交给含有wait()方法的线程，让其继续执行下去，如果自身没有锁，怎么叫把锁交给其他线程呢；（本质是让处于入口队列的线程竞争锁）

 **notify()就是对对象锁的唤醒操作。但有一点需要注意的是notify()调用后，并不是马上就释放对象锁的，而是在相应的synchronized(){}语句块执行结束，自动释放锁后，JVM会在wait()对象锁的线程中随机选取一线程，赋予其对象锁，唤醒线程，继续执行。这样就提供了在线程间同步、唤醒的操作。** 



#### 如何判断线程是否安全？

明确共享数据 对共享变量的操作是不是原子操作 ， 当某一个线程对共享变量进行修改的时候，对其他线程是可见的
**保证原子性的是加锁或者同步， 提供了volatile关键字来保证可见性， synchronized和锁和 volatile都能保证有序性** ,CAS本身就是一个锁哦。





#### 位运算的运用

两个相同的数进行异或，结果为0；

一个数 & 自己的相反数 ，结果为二进制下这个数最右侧的1

一个数 & 自己减一，结果为这个数去除最右端的一个1

一个数 & 只有一个1的数，可以判断这个数在当前位是否为1

0 异或任何数都等于这个数本身



#### SOLID

- S:单一职责原则
  
- O:开闭原则：对扩展开放，对修改关闭，也就是通过扩展已有模块的功能来满足新的需求。
  
  - 具体实现：抽象约束，封装变化
  
- L:里氏替换原则：子类可以扩展父类的功能但不能改变原有的功能

- I:接口隔离原则：把大的接口拆分为小的接口
  
- D:依赖倒置：降低耦合，高层模块不应该依赖低层模块而是各自依赖其抽象，核心是面向接口编程而不是面向实现编程
  
  - 每个类尽量提供接口或抽象类，或者两者都具备。
  - 变量的声明类型尽量是接口或者是抽象类。
  - 任何类都不应该从具体类派生。
  - 使用继承时尽量遵循里氏替换原则。
  
  



#### 集合类源码解析

- ArrayList: 基于动态再分配对象数组实现，有扩容操作，为原数组大小*1.5，由于实现了RamdonAccess接口支持快速随机访问，因此使用for循环去遍历会比较好，而不是使用迭代器
- LinkedList:  继承于AbstractSequentialList的双向链表。它也可以被当作**堆栈、队列或双端队列**进行操作。推荐使用foreach循环或者迭代器进行遍历， 头结点、尾结点都有**transient关键字修饰**，这也意味着在**序列化时该域是不会序列化**的。  
  - 可以通过getFirst方法实现队列操作和栈操作
- Stack：继承了Vector所以是线程安全的，也是复用了Vector的方法
- TreeMap：内部是一个红黑树



#### 运行时异常和非运行时异常

首先看一下继承结构：可抛出-》异常/错误   异常-》运行时异常/其他异常

- 运行时异常：都是RuntimeException的子类，比如空指针，数组越界等等异常，你可以选择是否捕获他，一般就是你代码写错了，特点是**编译时不会检查**。
- 非运行时异常：就是不继承RuntimeException的，编译时会检查，也就是**检查型异常**，在定义方法时必须要声明可能会抛出的检查型异常，调用这个方法时就必须对这些异常进行捕获，或者继续向外抛。



#### 基本类型跟装箱类的==

**（1）int与Integer、new Integer()进行==比较时，结果永远为true**

**（2）Integer与new Integer()进行==比较时，结果永远为false**

**（3）Integer与Integer进行==比较时，看范围；在大于等于-128小于等于127的范围内为true，在此范围外为false。**



#### OOM类型



#### SubList方法





#### 接口的方法

简单来说就是静态方法要有方法体

普通方法只能是public ,合法的关键词可加可不加，default和abstract不能同时加



#### 加密方法

 常见的 对称加密 算法主要有 DES、3DES、AES 等，常见的 非对称算法 主要有 RSA、DSA 等，散列算法 主要有 SHA-1、MD5  



### 一些反射相关

首先回顾一下反射，就是通过一个类本身去活得它的实例及方法，然后你可以对这个反射回来的镜像。我们一般的目的就是为了调用方法嘛

**对于方法列表的获取：**

getMethod是所有的公有方法（列表），包括从父类继承过来的

getDeclaredMethod是这个类自己声明的方法（列表） 

Class 中会维护一个 ReflectionData 的软引用，作为反射数据的缓存，保存了方法列表和成员列表之类的。

**获取方法：**

通过Method的copy方法返回方法的一个拷贝。

**调用方法**：

使用什么样的方式去调用这些方法，取决于使用哪种**MethodAccessor**

分别有：

**MethodAccessorImpl** 是通过动态生成字节码来进行方法调用的，是 Java 版本的 MethodAccessor

**DelegatingMethodAccessorImpl** 就是单纯的代理，真正的实现还是 NativeMethodAccessorImpl。

**NativeMethodAccessorImpl** 是 Native 版本的 MethodAccessor 实现。（性能较低但是加载的资源较少，如果调用次数较多，就可以转为使用第一种实现）

#### 为什么反射效率会低下呢

1. 调用方法时会对方法参数做封装和解封
2. 需要检查方法可见性
3. 需要校验参数
4. 难以内联



## 1、获取Class对象的三种方式

1.1   getClass(); 

1.2 任何数据类型（包括基本数据类型）都有一个“静态”的class属性 

1.3 通过Class类的静态方法：forName（String  className）(常用)



#### Comparable&Comparator