## Java沙箱

### Java沙箱的基本组成部分

- 字节码校验器

- 类加载器
  - 防止恶意代码去干涉恶意代码
  - 守护了被信任的类库边界
  - 将代码归入保护域，确定了代码可以进行哪些操作

  使用双亲委派模型在其中的作用，java核心api中定义类型不会被随意替换，那么外层恶意同名类得不到加载从而无法使用

- 存取控制器：可以控制核心API对操作系统的**存取权限**

- 安全管理器：是核心API和操作系统之间的主要接口。实现权限控制，比存取控制器的优先级高。

- 安全软件包



### 沙箱的要素

- 权限

  - 权限类型（就是一个类，如java.io.FilePermission, SocketPermission）
  - 权限名，就是对于某个权限类型的参数，比如对文件的权限，你需要提供一个文件名，比如对jvm属性的权限，你需要提供一个对应的jvm属性名
  - 允许操作，对某种权限类型指定的“资源”，对其可执行的操作的一个集合，比如你指定了一个文件，对这个文件只能读。

- 代码源

- 保护域

  - 用于组合代码源和权限。在于声明了代码A可以做权限B这样的事

- 策略文件 xxx.policy

  - 先看一个全局的 java.policy（节选）

  ```policy
  授权基于路径在file:$/*的class和jar包，所有权限
  grant codeBase "file:${{java.ext.dirs}}/*" {
          permission java.security.AllPermission;
          指明了JDK扩展包可以有全部权限
  };
  
  //directory/表示directory目录下的所有.class文件，不包括.jar文件；
  //directory/*表示directory目录下的所有的.class及.jar文件；
  //directory/-表示directory目录下的所有的.class及.jar文件，包括子目录；
  ```

  基本配置原则

  - 没有的权限表示没有
  - 只能配置有什么权限，不能配置禁止做什么
  - 同一种权限可多次配置，取并集
  - 统一资源的多种权限可用逗号分割

- 参数文件 xxx.security ,用于定义沙箱的一些参数

- 密钥库：保存密钥证书的地方



### 沙箱的启动

沙箱默认不启动，启动命令为

java -D **java.security.manager**

执行策略文件

java -D **java.security.policy=<url>**



### 沙箱的应用

编写一个启动类，读一个文件

在idea里面可以编辑启动的VM option，添加-Djava.security.manager

启动起来之后就会发现access denied



此时构造一个策略文件，执行目标目录下所有文件可读，此时**加上**参数

java -D java.security.policy=custom.policy

此时可读到文件内容。





## Java安全管理器  Java Security Manager

是核心API和操作系统之间的主要接口，主要用于实现权限控制

这是一个类，就是会检查一个应用的某个操作是否被允许在一个安全的上下文中被执行。



设置、获取当前管理器：

- 可以使用System类的setSecurityManager()方法来设置当前安全管理器；
- 可以使用System类的getSecurityManager()方法来获取当前安全管理器；

调用checkPermission方法，检查当前应用有没有该权限

```java
SecurityManager securityManager = System.getSecurityManager();
// 当前操作为检查当前应有有没有这个文件的写权限
securityManager.checkPermission(new FilePermission("D:\\security\\2.txt", "write"));
```

如果策略文件中**只有读权限**，那么这里就会抛出SecurityException



除了checkPermission，SecurityManager还有下面方法：

```java
checkAccept(String, int)
checkAccess(Thread)
checkAccess(ThreadGroup)
checkAwtEventQueueAccess()
checkConnect(String, int)
checkConnect(String, int, Object)
checkCreateClassLoader()
……
```



如果想要在代码运行的过程中关闭安全管理器，可以

```java
SecurityManager sm = System.getSecurityManager(); 
if(sm != null){ System.setSecurityManager(null); }
```

但是这样的话，那是不是别人在代码这样一写，那管理员辛辛苦苦整出来的规则岂不是就白费了？不可能，这个代码必须要有设置安全管理器的权限才可以执行。

```java
permission java.lang.RuntimePermission "setSecurityManager";
```





## 沙箱逃逸

针对java的沙箱逃逸，实际上就是绕过JavaSecurityManager。



### 利用单等号+home目录可写绕过

简单的来说就是，在默认的安全策略文件中有两个默认的策略文件，其中一个是在用户根目录下的，我们通过另外一个程序去篡改用户根目录下的那个策略文件，然后需要在沙箱中运行的那个程序好巧不巧在指定启动参数的时候 使用了**单个等号**，那么用户根目录下的policy文件也会生效，那么程序就有机会跳出手动编写的policy文件的限制。

解决方法：

	使用 两个等号 “==”执行policy文件。这样就能只使用一个policy文件。![1626665833789](C:\Users\H30008~1\AppData\Local\Temp\1626665833789.png)



### 后面还有很多，难度比较大，后面有时间就继续看……

https://www.mi1k7ea.com/2020/05/03/%E6%B5%85%E6%9E%90Java%E6%B2%99%E7%AE%B1%E9%80%83%E9%80%B8/