### Consul的结构

由server和client组成，他们都可以被称为Agent

server有主从模式，即主节点和副节点。

client节点可有可无，但是最好要有，如果有client的话，就要join到server中。

普通的微服务部署在Client节点上或server节点上，向client注册自己，注册完之后，client将注册信息转发到Server中，这些信息是保存在Server中的。

如果有一个服务想要获得已经注册的某个服务的IP和端口，那么首先他自己这台服务器上要有一个Consul Client，他向这个Client请求，然后这个Client又会向Server请求信息，再给回请求方。

就好像一个班里有班长和副班长，他们掌管着全班所有同学的花名册，每个小组的小组长负责将自己小组的同学的花名册汇集给班长，并收集当前小组的同学的查看其他人的花名册请求，反馈给班长，由班长再来处理。



### consul是如何注册服务的

编写一个json文件，包含如下内容，然后放到consul 的config目录下

- 要注册的微服务的id和name

- 一个web项目，会暴露自己的**IP和端口**
- 健康检查要暴露的接口，发送心跳间隔等(consul Server通过Http请求确认是否可用，状态码需要为200)

```json
{
  "services": [
    {
      "id": "hello1",
      "name": "hello",
      "tags": [
        "primary"
      ],
      "address": "172.17.0.5",
      "port": 5000,
      "checks": [
        {
        "http": "http://localhost:5000/",
        "tls_skip_verify": false,
        "method": "Get",
        "interval": "10s",
        "timeout": "1s"
        }
      ]
    }
  ]
}
```



### consul是如何发现服务的（获取服务的IP和端口）

- 通过HTTP API方式：

  - 向自己所在机器上的consul client查询id是XXX的服务
  - 自己机器上的consul client其实就是本地的8500端口，请求发给它就OK

  ```
  curl http://127.0.0.1:8500/v1/health/service/hello?passing=false
  ```

  - 返回的结果有多少个符合的节点就返回多少个，这部分内容可以设置缓存，但要注意服务不可用的问题。

- 可以通过DNS，注册到client的服务会有一个域名，通过标准的DNS查询就可以查到IP但是查不到端口。而且只能查到一个IP地址，这就是DNS原本的特性。

- Consul Template



### 有关consul的健康检查

如果当前机器上的Agent挂掉了，那么在这台服务器上部署的服务，注册到这个Agent的服务，也会被认为是不可用的（因为健康检查就是由这个Agent负责的，负责人已死），尽管微服务本身还是好好的。

鉴于Consul健康检查的这种机制，同时避免单点故障，所有的业务服务应该部署多份，并注册到不同的Consul节点。





### Spring整合Consul过程

1. 添加maven依赖

```xml
		<!-- consul-client -->
        <dependency>
            <groupId>com.orbitz.consul</groupId>
            <artifactId>consul-client</artifactId>
            <version>0.10.0</version>
        </dependency>

        <!-- consul需要的包 -->
        <dependency>
            <groupId>org.glassfish.jersey.core</groupId>
            <artifactId>jersey-client</artifactId>
            <version>2.22.2</version>
        </dependency>

      <!-- consul的java客户端有两个：consul-client和consul-api -->
```

2. 创建consul实例并注册自己上去

```java
	/**
     * 注册服务
     * 并对服务进行健康检查
     * servicename唯一
     * serviceId:没发现有什么作用
     * 服务注册的时候不需要传递IP
     */
    public void registerService(String serviceName, String serviceId) {
        Consul consul = Consul.builder().build();            //建立consul实例
        AgentClient agentClient = consul.agentClient();        //建立AgentClient
        
        try {
            /**
             * 注意该注册接口：
             * 需要提供一个健康检查的服务URL，以及每隔多长时间访问一下该服务（这里是3s）
             */
            agentClient.register(8080, URI.create("http://localhost:8080/health").toURL(), 3, serviceName, serviceId, "dev");
        } catch (MalformedURLException e) {
            e.printStackTrace();
        }

    }
```

3. 发现服务

```java
/**
     * 发现可用的服务
     */
    public List<ServiceHealth> findHealthyService(String servicename){
        Consul consul = Consul.builder().build();
        HealthClient healthClient = consul.healthClient();//获取所有健康的服务
        return healthClient.getHealthyServiceInstances(servicename).getResponse();//寻找passing状态的节点
    }
```

4. 一些其他功能

```java
	/**
     * 存储KV
     */
    public void storeKV(String key, String value){
        Consul consul = Consul.builder().build();
        KeyValueClient kvClient = consul.keyValueClient();
        kvClient.putValue(key, value);//存储KV
    }
    
    /**
     * 根据key获取value
     */
    public String getKV(String key){
        Consul consul = Consul.builder().build();
        KeyValueClient kvClient = consul.keyValueClient();
        return kvClient.getValueAsString(key).get();
    }
    
    /**
     * 找出一致性的节点（应该是同一个DC中的所有server节点）
     */
    public List<String> findRaftPeers(){
        StatusClient statusClient = Consul.builder().build().statusClient();
        return statusClient.getPeers();
    }
    
    /**
     * 获取leader
     */
    public String findRaftLeader(){
        StatusClient statusClient = Consul.builder().build().statusClient();
        return statusClient.getLeader();
    }
```





### SpringBoot+Feign+Consul

1. 添加依赖

```xml
<!-- consul 服务注册和发现功能 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
```

2. 启动类使用@EnableDiscoveryClient注解

3. 规定自己的应用名

   ```yml
   server:
     port: 18901
   spring:
     application:
       name: provider
   ```

4. 在bootstrap.yml中配置Consul的配置（服务提供方，将自己注册到Agent上）

   ```yml
   spring:
     cloud:
       consul:
         host: localhost # consul所在服務地址
         port: 8500 # consul端口
         discovery:
           health-check-path: /actuator/health
           service-name: ${spring.application.name} # 服務提供者名稱
           heartbeat:
             enabled: true
   ```

4. 设计一些接口用来对外提供服务



####    服务消费方

1. 导入依赖（这里使用Feign进行远程调用）

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>

　　　　 <!--  使用feign开启访问 -->
        <dependency> 
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

2. 如果这边不作为服务提供方的话就不需要把自己注册到consul，在application.yml中配置连接到consul
3. 启动器添加@EnableFeignClients注解
4. 使用@FeignClient(name="provider") 去标注一个接口上，表示去访问应用名为provider的微服务的接口，这里跟Dubbo类似,直接访问这个接口的方法就会封装一个请求去请求服务提供方提供的服务。

```java
@FeignClient(name = "provider")
public interface ProviderService {

    @RequestMapping("/hello")
    public String hello();

}

//在使用中，我直接去调用这个接口，那么就会按照两个注解组合的方式去发起一个请求，FeignClient用IP地址替换，然后根据RequestMapping去封装请求体和参数
```

