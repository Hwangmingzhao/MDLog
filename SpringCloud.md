### 微服务

是一种架构风格，将单一应用程度拆分为一组小的服务，独立部署独立发布运行于独立地进程中，一般基于RESTful API相互调用。

- 单机版
- 分布式：降低耦合度

优缺点：

- 服务足够内聚，聚焦一个指定的业务功能或业务需求
- 开发简单，开发效率提高
- 团队小，方便管理
- 松耦合
- 易于和第三方集成
- 只有业务逻辑的代码，不会跟前端代码发生关系

缺点：

- 分布式系统的相互调用问题
- 运维难度增加 
- 数据一致性问题
- 系统集成测试



**技术栈**：

服务开发

服务配置和管理

服务注册与发现

服务调用

服务熔断器

负载均衡

服务结构调用

消息队列

服务路由

服务监控

全链路追踪

服务部署

数据流操作开发包

事件消息总线



**为什么使用SpringCloud**

 Dubbo和SpringCloud对比：

也就是一般要有什么功能：功能定位，支持Rset,支持RPC，服务注册与发现，负载均衡，配置服务，服务调用链监控，高可用、容错等。

社区活跃度dubbo比较差

服务注册中心：zk / Eureka

服务调用方式：RPC / REST API

服务监控： Dubbo-monitor / SpringBoot Admin



#### 构建过程

使用SpringBoot构建小模块，用SpringCloud做集成



### Eureka

 `Eureka Server`和`Eureka Client`

Eureka Server提供服务注册服务，各个节点启动后，会在Eureka Server中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到。

Eureka Client是一个java客户端，用于简化与Eureka Server的交互，客户端同时也就别一个内置的、使用轮询(round-robin)负载算法的负载均衡器。

在应用启动后，将会向Eureka Server发送心跳,默认周期为30秒，如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个服务节点移除(默认90秒)。

Eureka Server之间通过复制的方式完成数据的同步，Eureka还提供了客户端缓存机制，即使所有的Eureka Server都挂掉，客户端依然可以利用缓存中的信息消费其他服务的API。综上，Eureka通过心跳检查、客户端缓存等机制，确保了系统的高可用性、灵活性和可伸缩性。



- 同步：每个 Eureka Server 同时也是 Eureka Client（逻辑上的）
  　　　多个 Eureka Server 之间通过复制的方式完成服务注册表的同步，形成 Eureka 的高可用
- 识别：Eureka Client 会缓存 Eureka Server 中的信息
  　　　即使所有 **Eureka Server** 节点都宕掉，服务消费者仍可使用缓存中的信息找到服务提供者
- 续约：微服务会周期性（默认30s）地向 Eureka Server 发送心跳以Renew（续约）信息（类似于heartbeat）
- 续期：Eureka Server 会定期（默认60s）执行一次失效服务检测功能
  　　　它会检查超过一定时间（默认90s）没有Renew的微服务，发现则会注销该微服务节点

Eureka 有一个 Region 和 Zone 的概念，你可以理解为现实中的大区（Region）和机房（Zone）

Eureka Client 在启动时需要指定 Zone，它会优先请求自己 Zone 的 Eureka Server 获取注册列表

同样的，Eureka Server 在启动时也需要指定 Zone，如果没有指定的话，其会默认使用 defaultZone





### 负载均衡Ribbon:

 通过Spring Cloud的封装，可以让我们轻松地将面向服务的REST模版请求自动转换成客户端负载均衡的服务调用 

使用方法：

- 服务提供者启动多个服务实例并主策到同一个注册中心或者相关联的集群中（注册中心相互注册）

- 服务调用者使用@LoadBalance修饰RestTemplate：利用了RestTempllate的拦截器，使用RestTemplateCustomizer对所有标注了@LoadBalanced的RestTemplate Bean添加了一个LoadBalancerInterceptor拦截器，而这个拦截器的作用就是对请求的URI进行转换获取到具体应该请求哪个服务实例ServiceInstance。
  - 具体如下：
  - 创建了一个LoadBalancerInterceptor的Bean，用于实现对客户端发起请求时进行拦截，以实现客户端负载均衡
  - 创建了一个RestTemplateCustomizer的Bean，用于给RestTemplate增加LoadbalancerInterceptor
  - 维护了一个被@LoadBalanced注解修饰的RestTemplate对象列表，并在这里进行初始化，通过调用RestTemplateCustomizer的实例来给需要客户端负载均衡的RestTemplate增加LoadBalancerInterceptor拦截器。
- 如果使用了feign来进行服务调用则默认集成了ribbon



### 容错保护：Hystrix

是一个延迟和容错库，用于隔离远程访问系统，防止级联失效，在使用的时候需要使用`@HystrixCommand(fallbackMethod = "queryItemByIdFallbackMethod")`这个方法一般放在同一个类中，注解指定这个方法失效的话会调用指定的方法，并且在执行这个方法会在另外一个单独的线程池中运行

**进阶：**

创建回调类，实现feign接口，扩展原方法为降级方法，并在配置文件中配置feign



### Feign客户端-声明式REST调用

使用RestTemplate调用服务会使得代码十分冗余。

使用Feign非常简单，创建一个接口并添加一些注解即可。SpringCloud对Feign进行了增强，使其支持SpringMVC注解并整合可Eureka和Ribbon

1. 创建接口使用FeignClient修饰
2. 声明方法使用MVC注解，参数使用@PathVariable或RequestParam修饰
   1. 这两个注解的区别：一个是填充url中的EL表达式一个是直接补充到url后面
3. 启动类添加Enable注解
4. 直接调用这个方法就行

流程分析：
1、由于我们在入口程序使用了@EnableFeignClients注解，Spring启动后会扫描标注了@FeignClient注解的接口，然后生成代理类。
2、我们在@FeignClient接口中指定了value，其实就是指定了在Eureka中的服务名称。
3、在FeignClient中的定义方法以及使用了SpringMVC的注解，Feign就会根据注解中的内容生成对应的URL，然后基于Ribbon的负载均衡去调用REST服务。



### Zuul服务网关

将权限控制日志收集这样的功能剥离出来，将其放在访问前端的部分。服务网关是微服务架构中一个不可或缺的部分。通过服务网关统一向外系统提供REST API的过程中，除了具备服务路由、负载均衡功能之外，它还具备了权限控制等功能 

- 身份认证及安全
- 审查与监控：监控边缘位置的数据
- 动态路由：将请求路由到不同的后端集群
- 压力测试
- 负载分配
- 静态响应处理：在边缘位置之间建立部分响应
- 多区域弹性

首先单独建立Zuul Demo,配置端口号，便可以在配置文件内通过建立请求路径和URL的映射，访问时直接访问服务网关，网关转发请求到指定的或负载均衡后的后端服务器。

**进阶：**通过将Zuul自己注册到Eureka中，然后在建立映射时将url改为service-id

**进阶：**Zuul也可以是一个集群并且跟nginx集成

 ![img](https://img-blog.csdnimg.cn/20181116185651362.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pwY2FuZHpoag==,size_16,color_FFFFFF,t_70) 



### SpringCloudConfig

我们开发项目时，需要有很多的配置项需要写在配置文件中，如：数据库的连接信息等。
这样看似没有问题，但是我们想一想，如果我们的项目已经启动运行，那么数据库服务器的ip地址发生了改变，我们该怎么办？
如果真是这样，我们的应用需要重新修改配置文件，然后重新启动，如果应用数量庞大，那么这个维护成本就太大了！

SpringCloudConfig为分布式系统外部化配置提供了服务端和客户端的支持，服务端负责连接存放配置文件的仓库，客户端则需要新建一个bootstrap.yml配置文件用于连接服务端并使用远程仓库的配置文件信息

服务端：

- 在application.yml中配置仓库地址
- 设定文件在仓库中的目录，还有分支名称
- 可以通过配置中心暴露的端口直接查看配置文件内容

客户端

- 配置bootstrap.yml，连接到配置中心，并指定你要去拿什么服务的和环境和什么分支的配置文件

- bootstrap.yml会在加载application.yml前加载，相当于一开始就从配置中心获取配置文件，接着你就可以在代码中使用@value请求到需要的配置

**进阶：**

如果配置文件内容在仓库中被修改，对于配置中心能够获得最新的配置，但是在客户端则需要重启才可以生效

- 在客户端整合actuator监控
- 对于需要动态更新配置bean使用@RefreshScope注解
- 在application.yml中开启所有端点
- 发送post请求刷新配置 http://client-ip:clientport/actuator/refresh 





## 	面试题部分

微服务的优缺点：

- 服务内聚
- 松耦合
- 面向接口编程
- 易于与第三方集成



与dubbo的区别：

- 调用方式不同：RPC VS RestAPI
- 注册中心不同
- SpringCloud作为RestAPI调用，需要服务网关进行调用管理



REST和RPC的对比：

- RPC要求服务方和调用方都有接口定义，耦合度比较高，容易出现冲突

  > REST是一种架构风格，指的是一组架构约束条件和原则。满足这些约束条件和原则的应用程序或设计就是 RESTful。REST规范把所有内容都视为资源，网络上一切皆资源。REST并没有创造新的技术，组件或服务，只是使用Web的现有特征和能力。 可以完全通过HTTP协议实现，使用 HTTP 协议处理数据通信。REST架构对资源的操作包括获取、创建、修改和删除资源的操作正好对应HTTP协议提供的GET、POST、PUT和DELETE方法。

- 两者使用协议不同：HTTP  VS TCP



微服务技术栈

维度(springcloud)
服务开发：springboot spring springmvc
服务配置与管理:Netfix公司的Archaiusm ,阿里的Diamond
服务注册与发现:Eureka,Zookeeper
服务调用:Rest RPC gRpc
服务熔断器:Hystrix
服务负载均衡:Ribbon Nginx
服务接口调用:Fegin
消息队列:Kafka Rabbitmq activemq
服务配置中心管理:SpringCloudConfig
服务路由（API网关）Zuul
事件消息总线:SpringCloud Bus



 **Eureka和Zookeeper区别** 

- Eureka取CAP的AP（可用 容错），注重可用性，Zookeeper取CAP的CP（一致性 容错）注重 

- 因为zk选举时会瘫痪而不保证高可用，eureka的客户端在想某个服务端注册时发现连接失败后会自动切换到其他节点，但是不同服务端看到的信息不一定是一样的。

- zk有主从节点，eureka平等

  >  当Eureka Server 节点在短时间内丢失了过多实例的连接时（比如网络故障或频繁启动关闭客户端）节点会进入自我保护模式，保护注册信息，不再删除注册数据，故障恢复时，自动退出自我保护模式。 



服务熔断和服务降级

 服务熔断：相当于保险丝，出现某个异常，直接熔断整个服务，而不是一直等到服务超时。通过维护一个自己的线程池，当线程到达阈值的时候就启动服务降级，如果其他请求继续访问就直接返回fallback的默认值。 



**什么是Spring Cloud Bus?**
spring cloud bus 将分布式的节点用轻量的消息代理连接起来，它可以用于广播配置文件的更改或者服务直接的通讯，也可用于监控。
如果修改了配置文件，发送一次请求，所有的客户端便会重新读取配置文件。



Eureka的自我保护模式

在一般情况下，客户端跟服务端通过发送心跳去保持联系，如果90秒都没发送心跳则会在注册表注销实例，但如果是出现网络异常则会导致同时大部分实例都失去连接但是他们本身是健康的，因此使用了自我保护模式，此时并不会删除注册表中的数据



**负载均衡**

 只需要做到，每一个上游都均匀访问每一个下游，就能实现“将请求/数据【均匀】分摊到多个操作单元上执行”。 其中每一层都有其实现负载均衡的技术栈。Ribbon只不过实现了其中一层负载均衡

- 客户端-》反向代理层

  我们访问一个网址，实际上在比较大的公司会映射多个ip，一般通过DNS轮询去切换IP（nginx的外网ip）

- 反向代理-》站点层

  由nginx实现，在nginx.conf中配置

  - 轮询
  - 最少连接路由，谁当前连接少就选谁
  - ip哈希
  - 响应时间短优先

- 站点层-》服务层

  使用服务连接池实现，一般两层之间通过RPC或RestAPI进行调用，Dubbo和Robbin有具体实现

- 数据层

  数据库分库分表，以降低数据库压力



### 从低级到高级的配置方式（比如有一个URL我要获取）

- 直接写在代码里
- 使用@Value（ "${myspcloud.item.url}" ）注解读取配置文件的内容
- 使用@ConfigurationProperties标注配置类，可以指定根标签，然后会根据属性名和成员名向下匹配
- 通过注册中心获取，Eureka要什么就给什么
  - 对于对外提供服务的机器，需要配置自己的应用名，还有配置Eureka的地址
  - 对于需要使用服务的机器，除了进行上述配置，还要开启负载均衡然后就可以通过应用名去获得对应的IP，就可以组装得到需要的URL



 