### 核心组件

- Channels（包含多种应用场景的Channel）
- Buffers（多种数据类型的buffer都有）
- Selectors（允许单线程处理多个channel，是一个多路复用器）



要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪



从某个Channel读到Buffer中，举个例子，从文件Channel读到字节Buffer中。

- 调用file的`getchannel()`方法获得这个文件的通道
- 定义一个Buffer，使用allocate方法
- 调用通道的`read(buffer)`方法，这样buffer里就有内容了



### Buffer

使用Buffer读写数据一般遵循以下四个步骤：

1. 写入数据到Buffer
2. 调用`flip()`方法
3. 从Buffer中读取数据
4. 调用`clear()`方法或者`compact()`方法

当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过``flip()``方法将Buffer从**写模式切换到读模式**。在读模式下，可以读取之前写入到buffer的**所有数据**。

几个属性：capacity（容量），position（读写指针），limit（读模式时， limit表示你最多能读到多少数据，因此它是切换模式前position的位置）



向Buffer写：

- `channel.read(buf)`

- `buf.put`

从Buffer读

- `inChannel.write(buf)`

- `buf.get()`





#### Scatter与gather

**分散（scatter）**从Channel中读取，是指在读操作时将读取的数据写入多个buffer中。因此，Channel将从Channel中读取的数据“分散（scatter）”到多个Buffer中。比如将一个文件的内容读到多喝buffer中。

具体操作：

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };
//在读取时，会先将前面的buffer填满
channel.read(bufferArray);
```

**聚集（gather）**写入Channel是指在写操作时将多个buffer的数据写入同一个Channel，因此，Channel 将多个Buffer中的数据“聚集（gather）”后发送到Channel。比如多个buffer向一个文件写入。





### Channel和Channel之间的数据传输

FileChannel有一个transferForm方法

```java
//作为目的
public abstract long transferFrom(ReadableByteChannel src,long position, long count)
//作为来源
public abstract long transferTo(long position, long count, WritableByteChannel target)
```





### Selector

Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。

**好处**：在原有的阻塞IO中，如果有多个IO,然后各自都分配一个线程，这样就会堵塞。对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好。



使用：

- 创建Selector

- 设置Channel为非阻塞模式，FileChannel不能是非阻塞的

- 注册Channel到Selector  

  ```java
  SelectionKey key = channel.register(selector,Selectionkey.OP_READ);
  //这个对象包含了如下信息
  /*
  interest集合
  ready集合
  Channel
  Selector
  */
  ```
  - register()方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件：

    1. Connect
    2. Accept
    3. Read
    4. Write

    通道触发了一个事件意思是该事件已经就绪。所以，某个channel成功连接到另一个服务器称为“连接就绪”。一个server socket channel准备好接收新进入的连接称为“接收就绪”。一个有数据可读的通道可以说是“读就绪”。等待写数据的通道可以说是“写就绪”。

    

#### 通过Selector选择通道

一旦向Selector注册了一或多个通道，就可以调用几个重载的select()方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道。换句话说，如果你对“读就绪”的通道感兴趣，select()方法会返回读事件已经就绪的那些通道。



大概是这样的：多个通道注册到一个Selector中，当其中的一个或多个通过发生了一些动作（感兴趣的），Selector通过select()就能知道,并将它们放到一个集合内，我们就可以通过遍历这个集合去对这些通道执行一些操作。





## IO与NIO

Java IO的各种流是阻塞的。这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。 Java NIO的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）