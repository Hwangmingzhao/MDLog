## Flink

高吞吐、低延迟、高性能的流式数据处理框架

框架运行框架类似于 顾客-》大堂经理-》柜员

顾客（program）将工作（job）给到大堂经理（Job manager)，大堂经理把这份工作的任务（Task）分给有空的柜台（Task Manager），每个柜台都是一个独立的JVM进程，每个柜台不止一个人在工作，可能有多个人（Task slot），是Flink中最小的资源单位。假如一个taskManager有3个slot，他就会给每个slot分配1/3的内存资源。

概括一下他们的关系：

一个job会被分成多个任务，每个任务会被一个TaskManager去处理，一个TaskManager有多个slot，一个task也可以继续被分成多个子任务，每个子任务会占用一个slot。

**数据流**

- 无界数据流：没有时间边界，需要持续不断地处理
- 有界数据流：有始有终，处理这样的数据的方式被称为批处理

**程序结构**

```scala
val env = ExecutionEnvironment.getExecutionEnvironment
//如果是实时数据流的话我们需要创建一个StreamExecutionEnvironment)。这个对象可以设置执行的一些参数以及添加数据源。所以在程序的main方法中我们都要通过类似下面的语句获取到这个对象：

// get input data,可以选择多种数据来源
val text = env.readTextFile("/path/to/file")

val counts = text.flatMap { _.toLowerCase.split("\\W+") filter { _.nonEmpty } }
  .map { (_, 1) }
  .groupBy(0)
  .sum(1)

counts.writeAsCsv(outputPath, "\n", " ")
```



**Flink的安装和使用**

1. 首先到官网下载需要的版本的包，然后解压。

2. 使用如下maven命令以flink模板为模板的maven工程

   ```shell
   $ mvn archetype:generate                               \
         -DarchetypeGroupId=org.apache.flink              \
         -DarchetypeArtifactId=flink-quickstart-java      \
         -DarchetypeVersion=1.9.0
   ```

3. 添加需要的maven依赖

4. 在StreamingJob（流式处理）中编写数据处理代码，基本流程是 环境-》数据-》处理-》输出

   ```java
   public static void main(String[] args) throws Exception {
   		final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
   		DataStream<Tuple2<String, Integer>> dataStreaming = env
   			.socketTextStream("localhost", 8082)//从8082端口读取输入
   			.flatMap(new Splitter())
   			.keyBy(0)
   			.timeWindow(Time.seconds(5))
   			.sum(1);
   
   		dataStreaming.print();
   
   		// execute program
   		env.execute("Flink Streaming Java API Skeleton");
   	}
   	public static class Splitter implements FlatMapFunction<String, Tuple2<String, Integer>> {
   
   		@Override
   		public void flatMap(String sentence, Collector<Tuple2<String, Integer>> out) throws Exception {
   			for(String word : sentence.split(" ")){
   				out.collect(new Tuple2<String, Integer>(word, 1));
   			}
   		}
   
   	}
   ```

5. 上述代码是从某个端口获取数据输入，因此安装netcat工具，绑定8082端口。

6. 启动代码，向8082端口写入数据

7. 可以之后，打包项目，启动flink进程 ，start-cluster.bat

8. 访问localhost:8081端口，上传打好的jar包，然后submit

9. 此时向8082端口写入数据。

服务器启动

scp streamjobtest-1.0-SNAPSHOT.jar root@192.167.190.119:/目标目录

flink run jar包路径 -c 启动类全限定名

提交到服务器上运行并查看资源使用状态



#### API的使用

**获取数据源**

1）基于文件的数据源

```java
readTextFile(path)
readFile(fileInputFormat, path)
readFile(fileInputFormat, path, watchType, interval, pathFilter, typeInfo)
```

（2）基于Socket的数据源（本文使用的）

`socketTextStream`

（3）基于Collection的数据源

```java
fromCollection(Collection)
fromCollection(Iterator, Class)
fromElements(T ...)
fromParallelCollection(SplittableIterator, Class)
generateSequence(from, to)
```

**转化方法**

1. Map ：One  -> One
2. FlatMap ：One -> N
3. Filter  :   根据筛选条件输出符合条件的元素
4. KeyBy：DataStream→KeyedStream 根据某个位置的字段将数据分为不同的块，比如同一个班的同学
   1. 除了可以传入整型参数，还可以传入自定义KeySelector，也就是说你可以自定义key是什么，比如说姓名的最后一个字，只要最后一个字一样的话就会被归为一类
5. reduce：reduce需要针对分组或者一个window(窗口)来执行，reduce需要数据是有区间有分组的，也就是分别对应于keyBy、window/timeWindow 处理后的数据，根据ReduceFunction将元素与上一个reduce后的结果合并，产出合并之后的结果，reduce过后依然会保留在同一个分组中
6. window :   框定指定区间（时间or数据条数）的数据，唯一的区别是`window(...)`针对keyby之后的keyedStream，而`windowAll(...)`针对非被Key化的数据流。
7. fold（已弃用） :   把数据进行一个汇总
8. Union将两个DataStream合为一个Stream

**DataSetAPI：**使用上差异不大，处理的是批数据，使用的方法方法名有一点点不同



##### TableAPI & SQL：可以跟流式和批式转换

大体的模式：

1. 注册1到n个Table，相当于创建一个Table集合
2. 从Table集合中取出一个Table，取出的方法可以使用sql的方法
3. 然后像数据库操作一样，使用TableAPI或者SQL的方式去把想要的数据查出来，由于像是数据库操作一样，所以查出来的东西还是一个Table
4. 最后Sink，就是把数据导出输出

一个简单的demo，就是把数据像存入数据库一样（就变成了一个Table），然后通过sql语句去实现各种操作。

```java
public class WordCountSQL {



    public static void main(String[] args) throws Exception {

        // set up execution environment
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        BatchTableEnvironment tEnv = BatchTableEnvironment.create(env);

        DataSet<WC> input = env.fromElements(
            new WC("Hello", 1),
            new WC("Ciao", 1),
            new WC("Hello", 1));
		//把这个DataSet存入一个名为WordCount的表里，字段分别为 word 和 frequency
        tEnv.registerDataSet("WordCount", input, "word, frequency");

        //利用sql语句实现wordcount
        Table table = tEnv.sqlQuery(
            "SELECT word, SUM(frequency) as frequency FROM WordCount GROUP BY word");
        
        //或者说对于sql函数也提供了一些api去使用，类似于hibernate那样
        Table filtered = table
                .groupBy("word")
                .select("word, frequency.sum as frequency")
                .filter("frequency = 2");

        DataSet<WC> result = tEnv.toDataSet(table, WC.class);

        result.print();
    }

    public static class WC {
        public String word;
        public long frequency;
        public WC() {}
        public WC(String word, long frequency) {
            this.word = word;
            this.frequency = frequency;
        }
        @Override
        public String toString() {
            return "WC " + word + " " + frequency;
        }
    }
}
```

更多细节：

- 将数据转换为一个表（Table）方法有很多种，可以是外部文件、真实的数据库、消息系统、一个已存在的Table对象、还有上面的流式数据和批式数据

- Sink的去向有很多，包括各种文件格式、存储系统以及消息系统

  - 一个批Table只能写入BatchTableSink中，而流Table需要一个`AppendStreamTableSink`、`RetractStreamTableSink`或者`UpsertStreamTableSink`

  ```java
  // 创建一个TableSink
  TableSink sink = new CsvTableSink("/path/to/file", fieldDelim = "|");
  // 将结果Table写入TableSink中
  
  ```

- 一个“类SQL”的操作是如何进行的：

  - 首先要经过翻译，因为flink肯定不是面向一个真实的数据库，翻译的第一步是**优化逻辑计划**，可以使用explain去查询执行计划
  - 第二步是翻译为一个**DataStream或DataSet程序**

- 一种场景下的应用： 如果我们的数据源是来自关系型数据库，那么我们可以直接转为Table，**先做一些简单的处理**比如说过滤空值、减少可见字段等，然后转换为**Dataset或DataStream**，再利用对应的API做复杂处理

  - Table转为DataStream： Append Mode:只能INSERT 　Retract Mode:它使用一个boolean标识来编码INSERT和DELETE更改。





#### OCC数据加工，以MRS中的Flink作为底座

![img](http://jiayongji.page.huawei.com/Flink/2020-11/intro-to-flink-and-practice-and-application-in-occ/pasteex_screenshot_intro-to-flink-and-practice-and-application-in-occ_20201102150644.png)



### Flink的应用场景

1. 事件驱动型应用，就是说来一个事件，那么会针对这个事件去操作本地的数据然后去进行一次计算。
   1. 比如说反欺诈情境下，被监控用户的一次借款就是一个事件，当事件发生时，我会调取一段时间的消费记录或者行动轨迹等信息进行一次计算，然后给出结果
2. 数据分析应用，就是广义上的数据分析，既支持流式处理也支持批处理，来一个数据我处理一个。
   1. 比方说流量监控
   2. 实时数据分析，大屏数据展示
   3. 大规模图分析
3. 数据管道，把水从沼泽输送到池塘中，其中可以包含转换、丰富数据的功能。
   1. 实时查询索引构建











# ApacheFlink视频教程总结



## EP02

如果以传统的批处理方式处理流式数据，假如一个事件跨越了两个采集区间，那么在两个区间之间就需要保存一些中间状态，传递到下一个区间中

> 举个例子：假如我要每一个小时统计通过我家门前100m的河的小船的数量，如果有一艘船在三点55分进来，四点15分出去，那么在三点到四点的区间中我只能知道这个船进来了，我要统计这个船是否出去，那么就要把小船进来的这个状态传递到四点到五点的状态中。

基于状态的流式处理需要有两个关键点：

- 他可以累计状态以及维护大量的状态（状态可以是多种形式的）
- 时间，根据时间去决定该收的数据是否都收完了，然后去生成结果--->这样才能产生精准的结果

基于状态的流式处理的四个特征：

1. 状态容错

   1. 一条数据只能改变一次状态（精确一次，全局的）

   2. 如何在分布式场景下为多个拥有本地状态的算子产生一个全域一致的快照（global consistent snapshot）

      1. 笨的方法：假如说你一条数据进来要按顺序经过三个算子处理，三个算子在不同的节点，那么当一条数据通过全部算子时，把三个算子在处理完这条数据之后产生的状态打包起来组合成一个快照。  但是这样的缺点就是，我位于开头的算子明明已经处理完了一条数据，面对下一条数据我明明可以开始执行了，但是因为前面的数据还没有走完剩下的算子，所以为了生成上一条数据的全局快照，我的状态不能被下一条数据更改。那这样效率就会及其低下

      2. 使用检查点，在checkpoint的时候将各个节点的状态同步到一个DFS中，如果有某一个节点挂掉了，那就从checkpoint恢复状态。

         一个保存点barrier被设置到一小批数据之后，从数据进入到数据源开始到这一小批数据被处理完成。基于这个保存点会形成一个快照：在数据进入到数据源时记录数据的偏移值，然后这批数据每完成一个算子也会相应记录一个状态，结束最后一个算子之后便最终完成了这个快照

2. 状态维护（状态很小-->JVM Heap）/ （状态很大 --> RocksDB）

   1. 我们的状态可以通过代码去向状态后端注册、更新、读取
   2. 如果使用的是JVM Heap，那么在生成全局一致快照时 需要序列化

3. **event-time**处理

   1. 以事件真实发生的时间，而不是接收到数据的时间。

   2. 如果说我要处理的是三点到四点的数据，那么是不是我在三点到四点之间收集到的数据就是我们要的数据呢，显然不一定。我们需要处理的是三点到四点**发生**的事件的数据，只有当我把我需要的所有数据都汇总到一个batch中，我才能说我能得到我想要的输出结果。

   3. Watermarks是一个带有时间戳的特殊事件，算子在接收这个事件之后就不会再接收时间戳比它小的事件了。

      > 我们知道，流处理从事件产生，到流经source，再到operator，中间是有一个过程和时间的。虽然大部分情况下，流到operator的数据都是按照事件产生的时间顺序来的，但是也不排除由于网络、背压等原因，导致乱序的产生（out-of-order或者说late element）。
      >
      > 但是对于late element，我们又不能无限期的等下去，必须要有个机制来保证一个特定的时间后，必须触发window去进行计算了。这个特别的机制，就是watermark。
      >
      > 举个例子：我想要统计0~10s内发生的数据，即event-time是在10s前的，这些数据从发生到来到我的处理器这边可能会有延迟，然后我认为我能给这个过程最多3.5秒的时间。
      >
      > 好，接下来哗哗哗来了一大堆数据，然后我再这些数据流中插入一个W（13.5）首先第6s来了一个数据（ET是5s），显然没问题，但是可能，后面还有其他数据，为了把所有合适的数据都放到一个窗口中执行，我决定等,  然后10s来了一个数据（ET是9.5s），还是可以，我继续等。然后11s来了一个数据（ET是10.5），这个时候我把这个数据放到10s-20s的窗口中，第一个窗口还是不能启动，因为我没接收到水位线W（13.5），然后13s来了一个数据（ET是8s，这就体现出作用了，虽然他迟到了，但是它还可以被处理），到了13.5s，接收到一个W（13.5）,OK，直接开始第一个窗口的任务，后面就算有ET < 10的过来我直接丢掉，（或者先记录下来，后面可能会用到），我都给了你3.5s的时间你都到不了，那等不了你了。
      >
      > 基于对我要处理的数据的认知，我可以设置watermarks什么时候插入到数据流中
      >
      > 另外一个博客：写的还可以：https://www.cnblogs.com/jmx-bigdata/p/13458117.html

   4. Flink中有三种时间：processing time / event time / ingestion time , 代码中可以切换，如果是processing time ，不能设置watermarks，其他的两个默认是200ms

4. 状态保存和迁移

   1. 手动设置checkpoint-->savepoint



## EP04

#### DataStream基本转换

![1606372331597](C:\Users\H30008~1\AppData\Local\Temp\1606372331597.png)



![1606372696905](C:\Users\H30008~1\AppData\Local\Temp\1606372696905.png)

## EP06

window有很多种：他们分别有不同的策略决定一个数据会被发送到哪个window中

- tumbling window：窗口之间的元素无重复
- sliding window：窗口之间的元素可能重复
- session window：如果一段时间内没有事件发生，那么就会开启一个新的window， 譬如说你登录了B站，在连续半个小时的使用中，这些事件都在一个窗口中，如果你半个小时没有操作，那么下一次的操作就会被记入下一个窗口中

![1606377332809](C:\Users\H30008~1\AppData\Local\Temp\1606377332809.png)

- global window


对于window的一些处理步骤：

![1606377735201](C:\Users\H30008~1\AppData\Local\Temp\1606377735201.png)

对于非必选的操作，如下：

- evictor 是用来删除某些元素的，实现evicBefore 和 evicAfter方法，下面是一些内置的Evictor
  - CountEvictor
  - DeltaEvictor
  - TimeEvictor
- trigger 指定了何时触发窗口内的计算，触发计算后会开启一个新的窗口。
- 一些窗口的创建其实就是创建了一个全局窗口然后添加相应的evictor和trigger

可以去看TopSpeedWindowing.java这个示例



## EP7

有状态访问例子

一个状态的输出是由上一个状态加上当前输入叠加计算而来的，也就是说就算来了完全相同的数据，但是如果前面的状态不相同，那么计算出来的下一个状态也不相同。其实状态就是**某种形式的数据**。一个operator就会有一个state，想象一个处理流程，一个数据的到达其实本质上就是在通过不同方式不断更新operator的状态。

**在什么情况下需要使用状态呢**

- 去重，就像是数据库主键，是否去重其实是根据原来有的数据去执行的
- 窗口计算：在这里指的是你的数据进来，但是没有触发计算的时候需要**暂时**保存在某一个地方，那么这也叫一个状态。
- 机器学习，一个状态就是一个模型（参数的组合）
- 访问历史数据：把历史数据也放在状态中。
- 结合CheckPoint去做故障恢复



### 状态类型

- 从管理形态上来说，分为：Managed State &  Raw State

  - Managed State是由Flink框架去管理的，譬如ValueState，ListState，MapState，通过框架提供的接口进行管理和更新操作。
  - Raw State （原始状态）：自己定义数据结构

- Keyed State & Operater State

  - Operator State是每一个Operator都会有一个，每一个并行度都会有一个，比方说你有一个Map算子处理的是DateStream，两个节点，那么对于这个算子就会有两个State

    - 对State的使用：
      - 如果你希望一个算子能够管理Operator State，那么就需要实现CheckpointedFunction，然后State就定义为成员变量，这个感觉跟ThreadLocal有点像。最后重写CheckpointedFunction的方法。在这些方法里你可以操作你的State。包括你可以从恢复过来的State去取数据来恢复你的现场。
      - Operater State的数据结构有ListState 、ListUnionStat 、Broadcast State
  - Keyed State 是每一个Key都会有一个State，
    - 对KeyState的使用：
      - KeyState的数据结构有 ValueState、MapState、ListState、ReducingState、AggregatingState



**怎么去管理我的内存**

- 内存，but
  - 247运行 高可靠    ----》 内存是有限制的，装不下这么多
  - 恰好计算一次  -----》内存挂掉就GG了
  - 实施产出不延迟  -----》 内存怎么扩展嘛，不能横向扩展机子
- 我希望好用的状态管理应该有什么特性：
  - 容易使用，可以装很多种数据
  - 读写快，恢复快，横向扩展很方便
  - 可以持久化，还能容错

现在我们有了不同的状态，数据进来之后产生了很多个状态，那么我们需要有一个地方去存储和读取这些状态，现在有两种存放的方式： 内存、持久化的

- 内存就是JVM heap，缺点就是可能会OOM，也不好扩展
- 持久化的有两种： FsStateBacked (受限于TaskManager的内存和磁盘大小) \ RocksDBStateBackend 



#### 状态的容错机制与故障恢复

- 使用CheckPoint进行全局分布式快照
- 发生故障时从CheckPoint恢复，但是要满足一定条件
  - 必要条件：数据源支持重发（Kafka），如果不能重发，那么CheckPoint到宕机的那段时间发生的数据就无法被恢复



## EP8

![1607587245607](C:/Users/H30008~1/AppData/Local/Temp/1607587245607.png)

#### 怎么去获取一个table

首先可以认为Table是从一个table env（环境）里scan（扫描）出来的，那么一个即将要被扫描出来的table**应该先注册**到env中。

- descriptor
- 自定义数据源
- 从DataStream注册

同样，我们也可以将结果写到table中，一样需要先注册，tableEnv.registerTableSink

#### table的API操作

![1607667724976](C:/Users/H30008~1/AppData/Local/Temp/1607667724976.png)

![1607668223244](C:/Users/H30008~1/AppData/Local/Temp/1607668223244.png)

关于**易用性**，For Example:

![1607668414473](C:/Users/H30008~1/AppData/Local/Temp/1607668414473.png)

![1607668449913](C:/Users/H30008~1/AppData/Local/Temp/1607668449913.png)

这个很强，我们就不用select一大堆列名了

![1607668536018](C:/Users/H30008~1/AppData/Local/Temp/1607668536018.png)

如果要实现table上的聚合操作，根据不同的输入输出组合有不同的聚合类

![1607669278308](C:/Users/H30008~1/AppData/Local/Temp/1607669278308.png)







## EP9

SQL

- 声明式API
- 自动优化-》屏蔽了State
- 流批统一 -》一样的SQL一样的结果



SQL中的聚合，上面一集用的是TableAPI的聚合

首先我们要知道，聚合是针对一批数据进行的，也就是窗口的概念。前面提到Flink有几种窗口：固定窗口、滑动窗口、会话窗口。



下面这个例子我们用写SQL的角度去理解：

- 首先我们的目的是统计每个用户每小时点击的次数
- 然后我们要取什么列
  - 用户名
  - 当前窗口的末尾边界时间
  - 点击次数
- 为了做聚合，我们就需要group by user
- 然后还有一步就是，group by还需要加一个时间窗口

![1607670133193](C:/Users/H30008~1/AppData/Local/Temp/1607670133193.png)



如果我们不针对窗口去做聚合，那么就是一个实时处理的模式，来一条数据就更新一下结果，那么相应的它对你的Sink也会有要求

![1607671705727](C:/Users/H30008~1/AppData/Local/Temp/1607671705727.png)







# 进阶EP1

Runtime：整体架构

![1608111630178](C:/Users/H30008~1/AppData/Local/Temp/1608111630178.png)

如果我们使用的是Yarn这样的资源管理系统，那么我们从客户端提交一个task的过程是：

1. 首先AM和TM都没有起起来，client向Yarn申请一个资源把AM先拉起来
2. client向AM中的Dispatcher提交任务，然后任务会给到JM
3. JM向RM请求计算资源
4. RM向yarn去要资源，yarn把TM起起来
5. TM把自己注册到JM中
6. JM看到有slot去执行任务了，就把任务丢给他们执行



Slot就是计算资源。可以用（CPU ，内存）这样的二元组去描述

![1608112422790](C:/Users/H30008~1/AppData/Local/Temp/1608112422790.png)



用户提交的是一个JOB，然后这个JOB首先会被分解成一个DAG（jobGraph），DAG中每一个节点(算子)就是一个Task，可能会有多个。

然后这个DAG会进一步被解析成ExecutionGraph，考虑并发，比如说flatMap会被分成多个task。

![1608112835192](C:/Users/H30008~1/AppData/Local/Temp/1608112835192.png)



调度策略：

1. Eager调度：一次性调度所有Task，适用于流式处理。
2. Lazy_from_source：先跑上游数据，上游跑完了下游再起进程，适用于批处理

错误恢复：

1. Task挂了
   1. 重启所有的task
   2. 重启某个task，只适用于task之间是没关联的情况
   3. 重启pipeLine Region，通俗的来说流处理的region会重启
2. AM挂了
   1. 现在的话所有的任务还是要重启的（可优化空间巨大）



## 进阶EP02

在Flink中的时间的地位：很重要，尤其是流式处理。

不同的时间语义：

1. 如果说我希望恢复之后重新处理数据的结果是一样的，那就得用Event time
2. 或者你的数据关心的是发生时间那么就必须使用event time



在现实中我们的数据并不会完全乱序的，他可能是局部有一点小乱，但是大体上是有序的，所以我们可以接触watermark这个东西来为数据做一些局部的集中，让他在每个小的batch之间保证是完全有序的。



产生watermark的两种方式：

- SourceFunction中产生

  - `void collectWithTimestamp(T element, long timestamp);`
  - `void emitWatermark(Watermark mark);`

- 在流程中指定

  - `public SingleOutputStreamOperator<T> assignTimestampsAndWatermarks`

  ![1608973656762](C:/Users/H30008~1/AppData/Local/Temp/1608973656762.png)



![1609729704884](C:/Users/H30008~1/AppData/Local/Temp/1609729704884.png)



# EP3

state  &  checkpoint

checkpoint的生成所需要的数据就是state

state:流式计算中持久化了的状态































# 尚硅谷



### 运行时架构

- 运行时组件

  - JobManager 作业管理器
    - 控制程序执行的主进程，只有一个JobManager
    - JobManager会接收应用程序，这个应用程序包括 JobGraph ，logical dataflow graph ,  以及整个jar包
    - 然后把JobGraph 转换为一个执行图 ExecutionGraph，这个执行图指导数据的流动和执行方式。
    - 接着就去向资源管理器请求slot资源，资源拿到之后就会把执行图发给任务管理器，开始干活。
  - TaskManager 任务管理器（负责干活）
    - 工作进程，一个任务管理器包含一定数量的slot，而一个Job也会包含多个任务管理器。
    - 同一个任务的TaskManager之间是可以交换数据的。
  - ResourceManager 资源管理（用于调配资源）
    - 有不同的资源管理器，Yarn Mesos K8sn 
  - Dispatcher 分发器

- 任务提交流程

  ![1625020430251](C:\Users\H30008~1\AppData\Local\Temp\1625020430251.png)

  - 如果使用Yarn作为ResourceManager

    ![1625021175452](C:\Users\H30008~1\AppData\Local\Temp\1625021175452.png)

    - 注意到，jar包和配置是先上传到HDFS上的。

    - 在ApplicationMaster中实际上也会有一个资源管理器ResourceManager，这个是Flink自己的ResourceManager，在JobManager请求资源的时候，首先会向自己的资源管理器请求，**但是自己的资源管理器会交给YARN这个资源管理器去处理**

- 任务调度原理



- 并行度

  - 一个特定算子的子任务的个数被称之为并行度，一般来说一个stream的并行度，可以认为是其**所有算子中最大的并行度**

    以下图为例子，上面对应的是整个Job的每一个步骤（算子），每一个算子都可以设置并行度，根据并行度就可以把每一个操作步骤分解为一个**Task**

  ![1625023420155](C:\Users\H30008~1\AppData\Local\Temp\1625023420155.png)

  那么上面这个应用程序就会被划分为七个子任务，那么这些子任务可能会以下面的方式去分配到不同的slot中，如果说我确定了需要五个slot，而一个TaskManager里有三个slot，那么就会分配两个TaskManager给这个Job。但是实际上，其实还不需要五个slot，两个就够了，因为这取决于**Job的并行度（2），参照下面第二张图**，一个slot是可以放很多个子任务的。他们不是一个任务的子任务即可，这种策略可以优化资源的使用，被称为槽位共享，默认开启。比如Source[1]和Source[2]不能放一起。

  一个Slot就会使用一个JVM的线程去执行。

  ![1625034224254](C:\Users\H30008~1\AppData\Local\Temp\1625034224254.png)

  ![1625034762792](C:\Users\H30008~1\AppData\Local\Temp\1625034762792.png)

  ![1625043802454](C:\Users\H30008~1\AppData\Local\Temp\1625043802454.png)

  :airplane: 如果前后的数据关系是**One-to-one**的话而且**并行度相同**而且在**同一个共享组**中的话，前后任务可以合并为一个大的子任务。这样可以减少通信开销。



- 一个Job从代码层面到最终在TaskManager执行起来，整个图的变化

  ![1625042619808](C:\Users\H30008~1\AppData\Local\Temp\1625042619808.png)














# 大数据实操

- 为什么不同的datastream为什么并行度不同
- 并行度可以在操作算子中设置，但是KeyedStream不行

- Kafka





#### 大数据学习之路

Hadoop 学习完之后就要分方向了：数仓离线计算、实时计算、流式计算等等。离线重点掌握 Hive、MapReduce，实时重点掌握 Spark、Flink。然后像 Zookeeper、Kafka、HDFS、Yarn 无论哪个方向都得学的。后台回复「**全部视频**」获取。

**五、推荐书单**

其实我不太建议新手一上来就啃书的，我都是建议我群里的群友根据我整理的一份面经来复习的，面经上虽然有题目和答案，但我都是建议看着问题，通过搜索引擎去整理出自己的答案。把每个模块看完了再系统性去看书，比较不会晦涩，也能抓的住重点。

Hadoop权威指南
Hadoop技术内幕：深入解析Hadoop Common和HDFS
Spark技术内幕
Hadoop技术内幕：Yarn
Spark大数据处理技术
Hive编程指南
Hbase企业级实战
Storm分布式实时计算模式
从paxos到zookeeper分布式一致性协议
Kafka源码剖析
数据仓库
Java编程思想（部分章节）
Java并发编程实战
深入理解java虚拟机
Java消息服务
Linux高性能服务器编程
Linux内核设计与实现
统计学习方法
机器学习实战
大话设计模式
大型网站技术架构
memcache全面剖析
快学Scala
剑指offer



我自己理解了一下整个过程和架构：

- 首先我们的大数据主要是批数据，那么为了对数据进行处理，出现了Hadoop，他一开始包括了HDFS用来存储，MapReduce用来计算，这样就能满足我们比较基本的针对批数据的处理。
- 那么后来对Hadoop添加了Yarn，使用Yarn来管理计算资源
- 再到后来，需要满足对流式数据的处理，出现了Flink，flink本身借鉴了MapReduce的编程模型，同时适配了两类数据，然后效率又很高，还改进了API，那么他就更加好用了， 因此我就希望把Hadoop里的MapReduce换成Flink，只需要使用一些组件就可以了
  - 在这里我们回归到数据处理的本身，其实并不是什么非常高深的过程，都是通过转化-》转化而来的，即数据的拆分与重组。转化的过程我们使用了MapReduce这样的一个编程模型，这个模型可以充分利用我们分布式系统的威力提高计算效率，另外一个问题就是如何提高整套流程的效率（从数据的流入、处理、流出），因此在一套框架之中，如果说有哪个新的东西出来能提高效率而且它能够兼容上下游，那么我们就可以把它换掉。

### HDFS

**Blocks:**在HDFS中，一个Block很大，有128MB，如果一个文件大于128M，那么这个文件就会以128M为单位分割为一些**chunk**，如果一个文件被拆分，那么一个文件也不会局限于存储在一个磁盘中了。

### 数据仓库

数据仓库里的数据并不是用来支撑业务流程的运行的，而是用来进行回溯和分析的，是用来支撑决策的制定的，也更加详细，粒度也较小。对于一个公司来说可能会有多个应用，但是多个应用中相同的部分会被集中保存到数据仓库中，比如订单信息























