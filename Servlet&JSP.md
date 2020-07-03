#### Servlet的生命周期

web容器加载servlet，生命周期开始。通过调用servlet的init()方法进行servlet的初始化。

通过调用service()方法实现，根据请求的不同调用不同的do***()方法。

结束服务，web容器调用servlet的destroy()方法。



#### 九大内置对象以及四大作用域

**JSP内置对象(9个内置对象):**

1、request对象  HttpServletRequest实例
request 对象是 javax.servlet.httpServletRequest类型的对象。 该对象代表了客户端的请求信息，主要用于接受通过HTTP协议传送到服务器的数据。（包括头信息、系统信息、请求方式以及请求参数等）。request对象的作用域为一次请求。

2、response对象 HttpServletResponse实例
response 代表的是对客户端的响应，主要是将JSP容器处理过的对象传回到客户端。response对象也具有作用域，它只在JSP页面内有效。

3、session对象  HttpSession实例
session 对象是由服务器自动创建的与用户请求相关的对象。服务器为每个用户都生成一个session对象，用于保存该用户的信息，跟踪用户的操作状态。session对象内部使用Map类来保存数据，因此保存数据的格式为 “Key/value”。 session对象的value可以使复杂的对象类型，而不仅仅局限于字符串类型。

4、application对象  ServletContext实例
application 对象可将信息保存在服务器中，直到服务器关闭，否则application对象中保存的信息会在整个应用中都有效。与session对象相比，application对象生命周期更长，类似于系统的“全局变量”。

5、out 对象  JspWriter实例
out 对象用于在Web浏览器内输出信息，并且管理应用服务器上的输出缓冲区。在使用 out 对象输出数据时，可以对数据缓冲区进行操作，及时清除缓冲区中的残余数据，为其他的输出让出缓冲空间。待数据输出完毕后，要及时关闭输出流。

6、pageContext 对象
pageContext 对象的作用是取得任何范围的参数，通过它可以获取 JSP页面的out、request、reponse、session、application 等对象。pageContext对象的创建和初始化都是由容器来完成的，在JSP页面中可以直接使用 pageContext对象。

7、config 对象 ServletConfig实例
config 对象的主要作用是取得服务器的配置信息。通过 pageConext对象的 getServletConfig() 方法可以获取一个config对象。当一个Servlet 初始化时，容器把某些信息通过 config对象传递给这个 Servlet。 开发者可以在web.xml 文件中为应用程序环境中的Servlet程序和JSP页面提供初始化参数。

8、page 对象 PageContext 实例
page 对象代表JSP本身，只有在JSP页面内才是合法的。 page隐含对象本质上包含当前 Servlet接口引用的变量，类似于Java编程中的 this 指针。

9、exception 对象
exception 对象的作用是显示异常信息，只有在包含 isErrorPage=“true” 的页面中才可以被使用，在一般的JSP页面中使用该对象将无法编译JSP文件。excepation对象和Java的所有对象一样，都具有系统提供的继承结构。exception 对象几乎定义了所有异常情况。在Java程序中，可以使用try/catch关键字来处理异常情况； 如果在JSP页面中出现没有捕获到的异常，就会生成 exception 对象，并把 exception 对象传送到在page指令中设定的错误页面中，然后在错误页面中处理

**四种属性范围:**
page(pageContext):只在一个页面中保存属性。 跳转之后无效。
request:只在一次请求中有效，服务器跳转之后有效。 客户端跳无效
session:再一次会话中有效。服务器跳转、客户端跳转都有效。 网页关闭重新打开无效
application:在整个服务器上保存，所有用户都可使用。 重启服务器后无效



#### 关于四种会话跟踪技术

- Cookie
  - Cookie它是存储在客户端或者客户端浏览器上的文本文件,很容易就可以查看到
  - Cookie不能存储中文
- Session
  - 多个请求共享一个会话，会话在第一次访问的时候就被创建
- Hidden
  - <input type="hidden">，非常适合步需要大量数据存储的会话应用。
- url重写



#### Servlet执行时一般实现哪几个方法？

1. public void init(ServletConfig config) 

   在Servlet的配置文件web.xml中，可以使用一个或多个<init-param>标签为servlet配置一些初始化参数。 **ServletConfig（配置对象）**里面就封装了这些信息。

2. public ServletConfig getServletConfig()

3. public String getServletInfo()：

   提供有关 servlet 的信息，如作者、版本、版权。

4. public void service(ServletRequest request,ServletResponse response)

   service() 方法是 Servlet 的核心。**每当一个客户请求一个HttpServlet 对象**，该对象的service() 方法就要被调用，而且传递给这个方法一个"请求"(ServletRequest)对象和一个"响应"(ServletResponse)对象作为参数。

5. public void destroy()

   destroy() 方法**仅执行一次**，即在服务器停止且卸装 Servlet 时执行该方法。

6. doGet() 方法
   当一个客户通过 HTML 表单发出一个 HTTP GET 请求或直接请求一个 URL 时，doGet() 方法被调用。与 GET 请求相关的**参数添加到 URL 的后面**，并与这个请求一起发送。当**不会修改服务器端的数据时，应该使用 doGet() 方法**。

7.  doPost() 方法
   当一个客户通过 HTML 表单发出一个 HTTP POST 请求时，doPost() 方法被调用。与 POST 请求相关的**参数作为一个单独的 HTTP 请求从浏览器发送到服务器**。当需要**修改服务器端的数据**时，应该使用 doPost() 方法。



### JSTL标签库



#### EL表达式：可用于获取变量的值、算数表达式

后台将对象发送到JSP页面，可以通过EL表达式获取并显示出来`${name}`，注意前面说的在JSP中有四个属性范围，寻找顺序是 page，request，session，application。

如果想要查找特定属性范围的某个对象或变量`${requestScope.name}` ,对封装的对象查找属性使用  .  来查找。    

 EL表达式在获取某个对象的属性值时，先将某个属性值首字母变成大写，然后加上get前缀，拼接成getter方法，通过反射将该对象构建出来，然后再对该对象执行getter方法，这与私有属性并没有关系，所以要注意，JavaBean的属性名要小写，且要有getter方法，不然会报错。所以基本要求是对象时POJO。

从List集合对象中获取某个值并显示：`${list_1[1].name }`

从Map中获取某个值并显示：

```jsp
<%
	Map map = new HashMap();
	map.put("a", new Person("aaa"));
	map.put("b", new Person("bbb"));
	map.put("1", new Person("ccc"));
	request.setAttribute("map", map);
%>
${map['1'].name }<!-- 是数字的话只能用括号，就算put进去的key值是字符串类型-->
${map.a.name }
```



#### SpEL 

它可以在运行时查询和操作数据，尤其是数组列表型数据，因此可以缩减代码量，优化代码结构 

**使用方法**

1. @Value

   ``` java
      //@Value能修饰成员变量和方法形参
      //#{}内就是表达式的内容
      @Value("#{表达式}")
      public String arg;
   ```

2. xml配置

   ```xml
   <bean id="xxx" class="com.java.XXXXX.xx">
       <property name="arg" value="#{表达式}">
   </bean>
   ```

3. Expression

**表达式语法**

1. 字面量（很少用，比如value="#{5}"  可以直接写成value=5)
2. 算数运算符
3. 字符串连接符 +
4. 比较运算符
5. matches





### Tomcat

一、Tomcat运行原理分析

1. Tomcat是运行在JVM中的一个进程。它定义为【中间件】，顾名思义，是一个在Java项目与JVM之间的中间容器。

2. Web项目的本质，是一大堆的资源文件和方法。Web项目没有入口方法(main方法)，，意味着Web项目中的方法不会自动运行起来。

3. Web项目部署进Tomcat的webapp中的目的是很明确的，那就是希望Tomcat去调用
   写好的方法去为客户端返回需要的资源和数据。

4. Tomcat可以运行起来，并调用写好的方法。那么，Tomcat一定有一个main方法。
5. 对于Tomcat而言，它并不知道我们会有什么样的方法，这些都只是在项目被部署进webapp下后才确定的，由此分析，必然用到了Java的反射来实现类的动态加载、实例化、获取方法、调用方法。但是我们部署到Tomcat的中的Web项目必须是按照规定好的接口来进行编写，以便进行调用

6. Tomcat如何确定调用什么方法呢。这取却于客户端的请求，http://127.0.0.1:8080/JayKing.Tomcat.Study/index.java?show这样的一个请求，通过http协议，在浏览器发往本机的8080端口，携带的参数show方法，包含此方法的路径为JayKing.Tomcat.Study，文件名为：index.java。