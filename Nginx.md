#### Nginx是什么

Nginx是一个HTTP服务器，可以将本机上的静态文件通过HTTP协议展示给客户端。配置起来相当简单，定位机器上的某个路径即可。

也可以是反向代理，那么为什么叫反向代理呢，就是他在服务端和客户端之间进行了代理。基于这一层代理可以实现负载均衡、虚拟主机、



#### 负载均衡

机器水平扩展，需要进行负载均衡

```
upstream myweb {
	server 192.168.0.111:8080; # 应用服务器1
	server 192.168.0.112:8080; # 应用服务器2
}
server {
	listen 80;
	location / {
		proxy_pass http://myweb;
	}
}
```



#### 虚拟主机

一个机器放几个服务端，共用一个IP，那么对用户来说使用不同的域名能访问到不同的网站，下面的配置展示了如何通过不同的Server-name去请求不同端口， server_name配置还可以过滤有人恶意将某些域名指向你的主机服务器 

```
server {
	listen 80 default_server;
	server_name _;
	return 444; # 过滤其他域名的请求，返回444状态码
}
server {
	listen 80;
	server_name www.aaa.com; # www.aaa.com域名
	location / {
		proxy_pass http://localhost:8080; # 对应端口号8080
	}
}
server {
	listen 80;
	server_name www.bbb.com; # www.bbb.com域名
	location / {
		proxy_pass http://localhost:8081; # 对应端口号8081
	}
}
```



### 优点：

- 高并发。静态小文件
- 占用资源少。2万并发、10个线程，内存消耗几百M。
- 功能种类比较多。web,cache,proxy。每一个功能都不是特别强。
- 支持epoll模型，使得nginx可以支持高并发。
- nginx 配合动态服务和Apache有区别。（FASTCGI 接口）
- 利用nginx可以对IP限速，可以限制连接数。
- 配置简单，更灵活。



### WEB服务器事件处理模型



### 配置文件

- main:主要配置与业务无关的参数，主要就是启动用户，进程数，日志目录
  -  `worker_rlimit_nofile 65535;`: 用于指定一个nginx进程可以打开的最多文件描述符数目，这里是65535，需要使用命令“ulimit -n 65535”来设置。 
- event：定义event模型工作特性，设置事件模型，每个进程最大连接数等
- http：作为Web服务器的配置
  - server
  - location



#### 