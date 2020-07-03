版本控制工具

优势：

不需要联网   完整性保证  尽可能添加数据而不是删除或者修改数据  分支操作OK  与Linux命令全面兼容



命令行操作：

1、本地库操作

2、远程库操作（GitHub 码云）



本地库操作

1、本地库初始化 git init

2、设置签名 区分不同开发人员的身份  这跟github 码云的账号密码没有任何关系

​    项目级别（仅在当前本地库范围内有效，优先级较高）：git config

​    用户级别（对当前使用电脑的用户有效）：git config --global

​     git config user.name  Hwangmingzhao

​	 git config user.email  Hwangmingzhao@qq.com

​	 cat .git/config  查看签名信息 

3、在主机本地目录下增加文件后，可以通过git status查看信息，应显示当前文件未加入缓存区（仅在工作区），接着使用**git add->git commit**提交

​	git add 文件名   / git add .

​	git commit  文件名    /   git commit提交全部 

​	git commit -a 直接提交跳过add

​	git commit -m message  不用在vim里添加信息

4、版本控制操作：查看历史信息 git log  查看回退步数 git reflog

​    前进后退：

​         基于索引值：git reset --hard 前面的索引值

​         使用^：git reset --hard HEAD^  只能往后退

​         使用~：git reset --hard HEAD~N(后退步数)

​    reset参数：

​         --soft:仅在本地库移动指针

​         --mixed: 移动指针，重置暂存区

​                   以上两个都不会修改工作区文件（即你当前的文件夹的内容）

​         --hard:  三个区同步移动

5、删除一个已提交文件：

​    首先你的文件要commit到本地库中

​    然后你删除工作区文件后，还需要在git中提交删除的操作

​    要找回文件的话，就回退到删除之前的历史记录就行

6、比较文件

​    将工作区的文件和暂存区进行比较

​         git diff [文件名]

​    将工作区中的文件和本地库历史文件记录比较

​         git diff [历史记录][文件名]

7、分支操作：并行开发，不会对master造成影响

​    创建分支 git branch 分支名

​    查看分支 git branch -v

​    切换分支 git checkout 分支名

​    合并分支

​         首先切换到要被切换的分支

​         执行merge 要合并的分支名

​    分支冲突：同一文件的同一行出现内容冲突，merge之后这个文件中会显示冲突，这时候需要按自己的需求编辑文件然后add commit(这里不需要文件名)





远程库操作：

1、本地库推送到远程库： git push 远程库别名 本地库分支

2、克隆：git clone 远程库地址  此时clone的项目名称是以远程库名称为准

​    会下载远程库 

​    创建别名

​    直接初始化本地库

3、使用git fetch 别名 分支名 拉取远程库到本地，但此时还没有更新当前本地库文件，需要使用merge合并两个版本，这个时候就会有冲突。



一个较为完整的过程是：

1. 用户一将自己的本地库推到远程库中，
2. 用户二可以克隆这个远程库的某一个分支到本地库中，
3. 通过加入团队他就可以将他的修改推送到远程库中，
4. 如果用户一想推送自己的本地库到远程库中，首先需要pull操作，如果**两个文件有冲突，首先要保证本地库中没有等待缓冲，然后在使用push推上去。**



1. 建仓库

2. 创建仓库地址别名 git remote add 别名 地址

3. 推送到远程库  git push 别名 分支  首先要确保该成员已经加入团队

4. 克隆，相当于下载之后初始化本地库 git clone 地址

5. 拉取某个分支 git fetch 别名 分支名 ，但此时还没有更新当前本地库文件 可以通过git checkout 别名/分支名 查看fetch过来的东西

6. 合并分支  git merge 别名/分支名   合并

7. pull = fetch+merge

8. 

   









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

- jps [option] [hostid]  虚拟机进程状况工具
  - -q:只显示PID
  -  jps -l ：输出应用程序main class的完整package名或者应用程序的jar文件完整路径名 
  -  jps -m：输出传递给main 方法的参数 
  -  jps -v：输出传递给JVM的参数 
- jstat 虚拟机统计信息统计工具
  - -**gc ：监控堆的状况，包括新生代，老年代，永久代的容量和gc时间**
  - **-class 监控类装载卸载数量及耗时**
  - -gcnew 新生代gc情况
  - -gcold
- jinfo [option] PID : 查看和调整虚拟机参数
  -  -sysprops 　　  把虚拟机进程System.getProperties()的内容打印出来 
- jmap [option] PID：java内存映像工具
  - -dump   生成转储快照
  - -histo  显示堆中对象的详细信息，跟上面的 jstat -class不同
  - -heap 堆的详细信息
  - -finalizerinfo 输出等待执行finalize方法的队列中的对象
- jhat 分析导出的堆的转储快照
  - 解析这个文件后会启动一个WebServer，端口号是7000
- jstack [option]  vmid  堆栈跟踪工具 ，**可以分析CPU消耗过大的原因，还有死循环死锁等问题**
  -  -F 当正常输出的请求不被相应时，强制输出线程堆栈
  - -l 除堆栈外，显示关于锁的附加信息
  - -m 如果调用到本地方法native的话，可以显示C/C++的堆栈 
  - 使用方法：
    1. 首先找到cpu使用情况，先找到进程占用比较高的，然后使用 top -Hp PID找到高占用的线程。
    2. 然后使用jstack ，接着在输出的信息中根据上一步的线程号找到对应的nid，可以看到当前线程的运行状态以及执行到什么方法
    3. 过一段时间再做一遍，如果还是执行这个方法，那么就很有可能是这里卡住了