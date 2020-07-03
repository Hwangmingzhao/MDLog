在分布式环境下，我们可以在不同的服务器上部署不同的服务，当我们需要传递参数调用其他服务器的方法时，我们就需要有一个中间的帮手为我们**建立连接、发送请求、序列化参数对象、返回结果**，而Dubbo就是这样的一个RPC框架，来帮我们完成这样的工作。

大概流程是：

- 首先我们把用户分为消费者和提供者，消费者是负责调用方法的，提供者则是提供服务的。
- 然后每个提供者有什么服务，他需要向注册中心说一声，这样使用者才能知道我可以调用什么。调用者在调用方法之前，需要向数据中心订阅，表示我需要服务。
- 这里需要说明的是: 这些服务可能在多个服务器上有副本，在真正调用时，注册中心会通过某些算法合理分配。
- 在这些之上，还有一个监视器负责监控消费者和提供者的行为



有几种注册中心：

- **Zookeeper**
- Redis
- Simple
- Multicast



启动Zookeeper(可选启动客户端)

启动Dubbo：java -jar dubbo-admin-0.1.jar



在这里如果我们想让项目变为分布式的，就是使用者跟实现者分离。那么我们应该这样做：

- 调用者调用接口，，，实现者实现接口，，，，那么我们就单独把接口提取出来放到另外一个项目中，再在调用者和实现者这边导入相应的依赖，然后问题来了：对于调用者来说，他现在只能看到接口而看不到实现类，要调用实现者实现的方法，我们就需要使用Dubbo来做到。

- 对原有的项目进行改造

  - 项目提供者将将服务（实现的接口）注册到注册中心
    - 导入dubbo依赖（底层有Spring）以及Zookeeper依赖
    - provider.xml：指定服务名字、指定注册中心位置、指定通信规则、暴露服务、实现服务

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
         xsi:schemaLocation="http://www.springframework.org/schema/beans     
               http://www.springframework.org/schema/beans/spring-beans.xsd    
               http://code.alibabatech.com/schema/dubbo        
               http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
  
      <!-- 提供方应用信息，用于计算依赖关系 -->
      <dubbo:application name="hello-world-app" />
      <!-- 使用multicast广播注册中心暴露服务地址 -->
      <dubbo:registry address="multicast://224.5.6.7:1234" />
      <!-- 用dubbo协议在20880端口暴露服务 -->
      <dubbo:protocol name="dubbo" port="20880"/>
      <!-- 声明需要暴露的服务接口 -->
      <dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService"/>
      <!-- 和本地bean一样实现服务 -->
     <bean id="demoService" class="com.alibaba.dubbo.demo.provider.DemoServiceImpl"/>
  
  </beans>
  
  ```

  ```java
  //通过加载配置文件将服务注册到注册中心
  public static void main(String[] args) throws Exception {
          ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new 				String[] {"provider.xml"});
          context.start();
          System.in.read(); // 按任意键退出
      }
  ```

  - 服务消费者订阅服务

    - 导入依赖（同上）
    - consume.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
        xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
     
        <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
        <dubbo:application name="consumer-of-helloworld-app"  />
     
        <!-- 使用multicast广播注册中心暴露发现服务地址 -->
        <dubbo:registry address="multicast://224.5.6.7:1234" />
     
        <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
        <dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" />
    </beans>
    ```

    ```JAVA
    public class Consumer {
        public static void main(String[] args) throws Exception {
            ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"/consumer.xml"});
            context.start();
            DemoService demoService = (DemoService)context.getBean("demoService"); // 获取远程服务代理
            String hello = demoService.sayHello("world"); // 执行远程方法
            System.out.println( hello ); // 显示调用结果
        }
    }
    ```

    

#### 整合SpringBoot：

- **服务提供者**导入dubbo-starter依赖，zookeeper自动导入

- 在application.properties配置dubbo参数。

  对于这一部分，我们可以使用注解方式简化配置

  - ```xml
    <!-- 如果有多个服务我们就需要配置多次-->
    <dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService"/>
        <!-- 和本地bean一样实现服务 -->
       <bean id="demoService" class="com.alibaba.dubbo.demo.provider.DemoServiceImpl"/>
    ```

  - 改为:  

    ```java
    //在实现的接口类使用@Service注解（Dubbo的）
    @com.alibaba.dubbo.config.annotation.Service
    public class ServiceImpl implement Service{
    	...
    }
    
    //然后在主配置类开启基于注解的Dubbo
    @EnableDubbo
    ```

- **服务消费者**也一样配置application.properties，，注册中心位置。。。

  - **重点是**：使用注解@Reference标注要使用的服务来自Dubbo

  ```java
  @Reference
  Service service;
  
  //主配置类
  @EnableDubbo
  ```

 

​	**总的来说，整合SpringBoot一共有如下三种方式：**

![1566457422183](C:\Users\H\AppData\Roaming\Typora\typora-user-images\1566457422183.png)

#### 一些可能会用到的配置

- 如果你在配置文件中想注册中心订阅了服务，但在使用过程中并没有真正使用他，那么就会报错。解决方法是，配置订阅服务属性check = "false"，这样直到在真正调用服务时才会检查。

  也可以使用consumer标签配置消费者缺省配置

  ```xml
  <dubbo:consumer check="false"></dubbo:consumer>
  ```

- 对于一个服务，你可以精确到某个方法，就是说，你实现的这个接口的多个方法，但是只把其中一个方法注册上去或者说不同的方法你有不同的要求，这样你就可以在xml文件中配置（基于注解的做不到）

- 有一个问题是，你调用的方法在其他服务器上，有可能因为网络延迟种种原因没办法立刻返回，那我又不可能一直等，那么就可以设置超时属性。

  - 精确优先（方法级优先，接口级次之，全局配置再次之）
  - 消费者设置优先（如果级别一样，则消费方优先，提供方次之）

- 重试次数，不包含第一次调用。在重试时会切换到别的服务器重试。

- 由于一个接口，或者说一个服务接口可以被实现多遍（多个版本），那么在提供方注册到注册中心时，虽然你注册的是同一个接口，但是应当注明是不同的实现类并添加属性version。在消费者使用服务的时候也可以指定要使用这个服务的哪个版本。

  ```xml
   <dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService"  version = "1.0.0"/>
      <!-- 和本地bean一样实现服务 -->
     <bean id="demoService01" class="com.alibaba.dubbo.demo.provider.DemoServiceImpl01"/>
  
  <!-- 版本二-->
   <dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService"  version = "2.0.0"/>
      <!-- 和本地bean一样实现服务 -->
     <bean id="demoService02" class="com.alibaba.dubbo.demo.provider.DemoServiceImpl02"/>
  ```

  

##   Zookeeper高可用



#### zookeeper宕机与dubbo直连（备用方案）

现象：注册中心关掉了，但是消费者仍然能消费服务，这是因为之前调用过了就会有缓存

![1566457815912](C:\Users\H\AppData\Roaming\Typora\typora-user-images\1566457815912.png)



要实现Dubbo重连，需要做一些配置：

在消费者调用的服务前使用注解  **`@Reference(url="127.0.0.1:28801")`**	这样就能直接找到提供者绕过了注册中心。



#### 负载均衡策略：

- RandomLoadBalance:基于权重的随机负载均衡机制
- RoundRobin LoadBalance:基于权重的轮询负载均衡
- LeastActive LoadBalance:最少活跃数
- ConsistentHash LoadBalance:一致性哈希





## Dubbo原理

优点：

- 面向接口的远程调用，粒度是接口
- 智能负载均衡
- 服务自动注册与发现
- 高度可扩展能力
- 流量调度
- 可视化的服务智力与运维



首先如果是我们在原来没有使用远程调用框架前，到底要怎么实现远程调用。首先我们的服务提供方需要开启网络服务，通过地址映射去处理请求，返回相应的结果给请求发送方。那么发送方首先需要生成一个请求，然后发出，接收到结果之后反序列化为一个对象，一旦对象被改变了那么怎么办，两边都要改。



架构如下

![img](https://images2017.cnblogs.com/blog/1147548/201709/1147548-20170928141450169-1251868962.png)





























