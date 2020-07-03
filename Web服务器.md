## 目录

### WEB容器

- 什么是web容器
- 有哪些web容器
  - jboss
  - jetty
  - tomcat

### Nginx



#### 我们先比较几个名词

**Web服务器**：就是用于提供Web功能的服务器，一般来说就是用于响应HTTP请求的，包括接收请求和发送请求。比如**Apache**，也包括我们后面要说的**Nginx**。另外**IIS**（来自微软）也是主流的Web服务器，他们适用于不同的场景。（Web服务器可以访问磁盘上所有资源文件）

**Web容器**（Web应用服务器）：是一种服务器程序，注意他是一个程序，对外有一个端口，比如我们常用的tomcat，就是一种容器，作为一个程序它可以主动加载我们编写的servlet。在一台web服务器上我们可以有多个Web容器，即多个tomcat。**但是**，tomcat并不算彻底的只是一个Web容器，他还是一个Web服务器

>   Tomcat是一个Servlet/Jsp容器，它同时也作为一个web服务器使用。Tomcat = ( Web Server + Servlet container + JSP environment )，因为我们知道JSP也是转译为Servlet的，Tomcat接收请求之后，如果是JSP页面的话，Tomcat里面的JSP引擎可以将JSP转换为Servlet类。  

按照这样来看的话，其实单纯的Web服务器功能很单一，只能做到外部请求一个html页面，如果主机里面有的话就返回这样简单的功能，而tomcat还可以实例化servlet然后执行一些逻辑

**应用服务器**：Web服务器算是应用服务器的一个子集



### 有哪些Web容器（Web应用服务器）

#### 1. Jboss

JBoss是一个运行EJB的J2EE应用服务器。EJB即Enterprise Java Bean，按我的理解，这是一个可以将一些比较高级的JavaBean装载起来的Web容器。但是它自身不能加载Servlet，因此还是需要使用Tomcat.还可以支持PHP、.NET

> 除了性能问题，Tomcat的另一大缺点是它是一个受限的集成平台，仅能运 行Java应用程序。企业在使用时Tomcat，往往还需同时部署Apache Web Server以与之整合。

#### 2. Jetty







