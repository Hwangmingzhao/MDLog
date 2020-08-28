#### Bean的生命周期（简单的）：

1. 实例化
2. 依赖注入
3. 调用实现的BeanNameAware接口的SetBeanName接口，传入id属性作为参数
4. 调用实现的BeanFactoryAware接口的setBeanFactory接口
5. ~~ ApplicationContextAwear  ~~ set~~
6. 初始化预处理，执行这个方法postProcessBeforeInitialization，在Bean初始化完成之前进行
7. 执行**init-method，执行完标志着初始化完成( @PostConstruct)**
8. 执行postProcessAfterInitialization，初始化完成后执行，然后这个Bean就可以开始使用了
9. 执行实现的DisposableBean接口的destroy方法
10. 执行**destroy-method( @PreDestroy**)



#### Bean几种作用域：

singleton:在容器中只有一个实例

prototype:多个实例

request:一个请求一个bean

session:

全局session：



### Spring的优点

- 非侵入式设计:怎么理解呢，就是你写业务代码的时候不会使用SpringAPI，如果你要移植也比较好弄。

- 容器DI： 管理对象的生命周期、对象与对象之间的依赖关系 

  - **构造器注入 constructor-arg**

  - **setter注入 property**

  - **静态工厂：property，也需要这个成员对象也需要有setter方法，不同之处在于ref是静态工厂bean，并且这个静态工厂的bean还需要添加factory-method来表示使用哪个方法**

  - **实例工厂：property，与上述的区别在于**

    手动装配就是我需要手动指定这个ref是哪个bean，**自动装配**就是会自动去找合适的bean去完成依赖注入

  - **autowire:no**

  - **byName**

  - **byType**

  - **constructor**

  - 集合类型的注入

    - <list>

    - <set>

    - </map/>

      - <entry key="" valve-ref="">

    - <props> 键值对分别为String的map

      - ```xml
        <prop key="pp1">hello</prop>
        ```

- IOC 降低耦合度，方便开发： 即创建被调用的实例不是由调用者完成，而是由Spring容器完成，并注入调用者 ，你不用费心去实例化对象，你需要的话就能拿到

- 支持AOP： 把很多被业务逻辑反复使用的服务完全剥离出来，以达到复用 

- 支持声明式事务

- 方便测试

- 方便集成框架

- 封装了好用的API





#### BeanFactory与ApplicationContext的区别

beanfactory是一个最底层的接口，只有实例化对象和拿对象的功能。

ApplicationContext就是应用上下文，可以在以上的基础上实现国际化，访问资源，不同层有各自的上下文，AOP等

BeanFactory默认为全部都是延迟加载，AC则反之，可以配置为延迟加载。

如果构造对象时缺少某些属性，ApplicationContext初始化时会检验，而BeanFactory在第一次使用时未注入，才会抛出异常 。

ApplicationContext是BeanFactory的一个实现类，更高级，提供了系统架构服务



顺便讲一下上面说到的AC的扩展功能：

国际化：实际应用就是我们对不同地区需要输出不同的提示信息，那么我们就可以配置一个 ReloadableResourceBundleMessageSource 的Bean,然后我们可以通过上下文的getMessage方法，传入你要取的key和地区信息就可以获得对应的提示信息

访问资源：直接通过容器的getResource方法访问资源

载入多个上下文：不同项目使用不同的配置文件， 可以在applicationContext.xml文件中引用 ，

```xml
    <beans>
         <import resource="applicationContext-cache.xml"/>
    </beans>
```





### 事务

一些重要的概念：

- 事务的四个特性
  - 一致性的理解：约束不能被破坏
- 事务属性

Spring不直接实现事务，而是提供了一个接口`PlatformTransactionManager`,不同的平台要实现他，就自己去实现。 **Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。** 

比如JDBC事务**管理器**：这是一个实现类，那么对于JDBC事务的具体实现，sql本来就可以做到，所以他用的是数据库原生的实现。

```xml
 <bean id="transactionManager"class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
```

比如Hibernate事务**管理器**：这个管理器需要一个会话工厂去生产事务对象，然后通过这个对象的commit和rollback方法去实现事务。

```xml
<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory" />
</bean>
```
我们可以看到对于不同的事务管理器，他们所需要的东西也不一样。



实现`PlatformTransactionManager`接口的事务管理器有一个获取事务的方法，这个方法需要传入**事务属性**，也就是说我们可以指定我们想要的事务带有什么属性。

- **传播行为  propagation behavior**：当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。Spring定义了七种传播行为：

  对于不同的事务管理器：默认的传播行为是不同的。

| 传播行为                  | 含义                                                         |
| :------------------------ | :----------------------------------------------------------- |
| PROPAGATION_REQUIRED      | 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务 |
| PROPAGATION_SUPPORTS      | 表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行 |
| PROPAGATION_MANDATORY     | 表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常 |
| PROPAGATION_REQUIRED_NEW  | 表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NOT_SUPPORTED | 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NEVER         | 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常 |
| PROPAGATION_NESTED        | 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。注意各厂商对这种传播行为的支持是有所差异的。可以参考资源管理器的文档来确认它们是否支持嵌套事务 |

如果你在spring中使用了@Transactional 标注其为事务方法，那么在执行方法的时候，就会为你这个调用过程先定义一个连接，然后调用方法之后再commit一下，如果中间出错了就在catch块中rollback。

​		**PROPAGATION_REQUIRED**：就是说对于这个这个方法，你单独使用他的话，会为你生成一个事务，但如果你被**另一个事务方法**调用，在上一级方法中已经生成了一个事务的话，那么你可以共享这个事务。

​		**PROPAGATION_SUPPORTS**：对于这个方法，如果**上级有事务，那么他就是事务执行**，就跟上面的REQUIRED一样，但是如果**单独调用他**又或者**上级没有事务**，那么**他自己也就没有事务**。

​		**PROPAGATION_MANDATORY**：表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常，没什么好说的，意思是，他的上级必须要有一个事务，而他自己又不会生成事务。

​		**PROPAGATION_REQUIRED_NEW**：没有的话自己生成，**上级有的话他还是自己生成**，而且这个新的事务只为他一个方法服务，在使用他自己生产出来的事务前，他**先把前面的事务挂起一下**（这时候需要用到事务管理器去帮帮忙挂起），执行完之后，再恢复原来的事务。**注意一下**：这样的话，两个事务间互不影响，也就是说，原来的事务执行成功还是失败都不会影响自己的结果，而自己的方法失败了也不会影响前面的事务执行。

​		**PROPAGATION_NOT_SUPPORTED**: 不需要事务，如果原来就有事务的话我就挂起，挂起之后继续执行自己的方法，自己执行失败了也不会回滚，也不会影响原来事务的回滚。

​		**PROPAGATION_NEVER**: 永远不要事务，上级要是有事务我就闹，我不执行我还要抛异常。

​		**PROPAGATION_NESTED**：嵌套事务，如果上级没有事务，那就自己生产一个给自己用。如果上级有，那自己也生产一个。**这里重要的是**：嵌套事务，内层事务依赖于外层事务，外层要是失败了，内层也会失败，但是内层失败与否不会影响外层。对于外层事务，事务管理器不像**PROPAGATION_REQUIRED_NEW**一样给外层挂起，而是设置savepoint



- **隔离级别**：隔离级别定义了一个事务可能**受其他并发事务影响**的程度。 

  - 多个事务并发地执行，可能会操作同一个数据，就会导致以下的问题：

    - 脏读（Dirty reads）——脏读发生在一个事务读取了另一个事务改写但尚未提交的数据时。如果改写在稍后被回滚了，那么第一个事务获取的数据就是无效的。
    - 不可重复读（Nonrepeatable read）——不可重复读发生在一个事务执行相同的查询两次或两次以上，但是每次都得到不同的数据时。这通常是因为另一个并发事务在两次查询期间进行了更新。**重点是修改**
    - 幻读（Phantom read）——幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录。**重点在于新增或者删除**（ISOLATION_SERIALIZABLE就可以让他根本就没办法新增或者删除）

  - 为了解决这些问题，定义了几种隔离级别：他们的实现好像是对数据做一些拷贝什么的？

  | 隔离级别                   | 含义                                                         |
  | :------------------------- | :----------------------------------------------------------- |
  | ISOLATION_DEFAULT          | 使用后端数据库默认的隔离级别                                 |
  | ISOLATION_READ_UNCOMMITTED | 最低的隔离级别，**允许读取尚未提交的数据变更**，可能会导致脏读、幻读或不可重复读。就是什么时候都可以读，但是完全不能保证对不对。 |
  | ISOLATION_READ_COMMITTED   | 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。我确保你**这次的读取是成功的是被正确提交的**，下次会不会被改呢，不知道。 |
  | ISOLATION_REPEATABLE_READ  | 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。**如果不是自己改动的数据，怎么读都是一样的**。 |
  | ISOLATION_SERIALIZABLE     | 最高的隔离级别，完全服从ACID的隔离级别，确保阻止脏读、不可重复读以及幻读，也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的 |

- **只读**
- **事务超时**：特定时间内事务如果没有执行完毕，那么就会自动回滚，而不是一直等待其结束。
- **回滚规则**：默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚。声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以**声明事务遇到特定的异常不回滚**，即使这些异常是运行期异常。





## 3.1 编程式和声明式事务的区别

Spring提供了对编程式事务和声明式事务的支持，**编程式事务允许用户在代码中精确定义事务的边界，而声明式事务（基于AOP）有助于用户将操作与事务规则进行解耦。** 
简单地说，编程式事务侵入到了业务代码里面，但是提供了更加详细的事务管理；而声明式事务由于基于AOP，所以既能起到事务管理的作用，又可以不影响业务代码的具体实现。

## 3.2 如何实现编程式事务？

Spring提供两种方式的编程式事务管理，分别是：使用TransactionTemplate和直接使用PlatformTransactionManager。

### 3.2.1 使用TransactionTemplate

采用TransactionTemplate和采用其他Spring模板，如JdbcTempalte和HibernateTemplate是一样的方法。它使用回调方法，把应用程序从处理取得和释放资源中解脱出来。如同其他模板，TransactionTemplate是线程安全的。代码片段：

``` java
    TransactionTemplate tt = new TransactionTemplate(); // 新建一个TransactionTemplate
    Object result = tt.execute(
        new TransactionCallback(){
            //在doTransaction内执行操作
            public Object doTransaction(TransactionStatus status){  
                updateOperation();  
                return resultOfUpdateOperation();  
            }  
    }); // 执行execute方法进行事务管理
```

使用TransactionCallback()可以返回一个值。如果使用TransactionCallbackWithoutResult则没有返回值。

### 3.2.2 使用PlatformTransactionManager

示例代码如下：

```java
 
    DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager(); //定义一个某个框架平台的TransactionManager，如JDBC、Hibernate
    dataSourceTransactionManager.setDataSource(this.getJdbcTemplate().getDataSource()); // 设置数据源
    DefaultTransactionDefinition transDef = new DefaultTransactionDefinition(); // 定义事务属性
    transDef.setPropagationBehavior(DefaultTransactionDefinition.PROPAGATION_REQUIRED); // 设置传播行为属性
    TransactionStatus status = dataSourceTransactionManager.getTransaction(transDef); // 获得事务状态
    try {
        // 数据库操作
        dataSourceTransactionManager.commit(status);// 提交
    } catch (Exception e) {
        dataSourceTransactionManager.rollback(status);// 回滚
    }

```



## 4 声明式事务

### 4.1 配置方式

注：以下配置代码参考自[Spring事务配置的五种方式](http://www.blogjava.net/robbie/archive/2009/04/05/264003.html)

根据代理机制的不同，总结了五种Spring事务的配置方式，配置文件如下：

（1）每个Bean都有一个代理

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">
 
    <bean id="sessionFactory" 
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean"> 
        <property name="configLocation" value="classpath:hibernate.cfg.xml" /> 
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
    </bean> 
 
    <!-- 定义事务管理器（声明式的事务） --> 
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
 
    <!-- 配置DAO -->
    <bean id="userDaoTarget" class="com.bluesky.spring.dao.UserDaoImpl">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
 
    <bean id="userDao" 
        class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean"> 
           <!-- 配置事务管理器 --> 
           <property name="transactionManager" ref="transactionManager" />    
        <property name="target" ref="userDaoTarget" />
         <property name="proxyInterfaces" value="com.bluesky.spring.dao.GeneratorDao" />
        <!-- 配置事务属性 --> 
        <property name="transactionAttributes"> 
            <props> 
                <prop key="*">PROPAGATION_REQUIRED</prop>
            </props> 
        </property> 
    </bean> 
</beans>

```

（2）所有Bean共享一个代理基类

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">
 
    <bean id="sessionFactory" 
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean"> 
        <property name="configLocation" value="classpath:hibernate.cfg.xml" /> 
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
    </bean> 
 
    <!-- 定义事务管理器（声明式的事务） --> 
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
 
    <bean id="transactionBase" 
            class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean" 
            lazy-init="true" abstract="true"> 
        <!-- 配置事务管理器 --> 
        <property name="transactionManager" ref="transactionManager" /> 
        <!-- 配置事务属性 --> 
        <property name="transactionAttributes"> 
            <props> 
                <prop key="*">PROPAGATION_REQUIRED</prop> 
            </props> 
        </property> 
    </bean>   
 
    <!-- 配置DAO -->
    <bean id="userDaoTarget" class="com.bluesky.spring.dao.UserDaoImpl">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
 
    <bean id="userDao" parent="transactionBase" > 
        <property name="target" ref="userDaoTarget" />  
    </bean>
</beans>

```

（3）使用拦截器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">
 
    <bean id="sessionFactory" 
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean"> 
        <property name="configLocation" value="classpath:hibernate.cfg.xml" /> 
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
    </bean> 
 
    <!-- 定义事务管理器（声明式的事务） --> 
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean> 
 
    <bean id="transactionInterceptor" 
        class="org.springframework.transaction.interceptor.TransactionInterceptor"> 
        <property name="transactionManager" ref="transactionManager" /> 
        <!-- 配置事务属性 --> 
        <property name="transactionAttributes"> 
            <props> 
                <prop key="*">PROPAGATION_REQUIRED</prop> 
            </props> 
        </property> 
    </bean>
 
    <bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator"> 
        <property name="beanNames"> 
            <list> 
                <value>*Dao</value>
            </list> 
        </property> 
        <property name="interceptorNames"> 
            <list> 
                <value>transactionInterceptor</value> 
            </list> 
        </property> 
    </bean> 
 
    <!-- 配置DAO -->
    <bean id="userDao" class="com.bluesky.spring.dao.UserDaoImpl">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
</beans>

```

（4）**使用tx标签配置的拦截器**  先配置好命名空间，需要配置事务管理器bean，然后使用tx标签配置一个事务Bean，然后通过aop 将这个事务配置到所需的切入点中

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">
 
    <context:annotation-config />
    <context:component-scan base-package="com.bluesky" />
 
    <bean id="sessionFactory" 
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean"> 
        <property name="configLocation" value="classpath:hibernate.cfg.xml" /> 
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
    </bean> 
 
    <!-- 定义事务管理器（声明式的事务） --> 
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
 
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED" />
        </tx:attributes>
    </tx:advice>
 
    <aop:config>
        <aop:pointcut id="interceptorPointCuts"
            expression="execution(* com.bluesky.spring.dao.*.*(..))" />
        <aop:advisor advice-ref="txAdvice"
            pointcut-ref="interceptorPointCuts" />       
    </aop:config>     
</beans>

```

（5）全注解   首先还是要配好命名空间，然后设置注解扫描包，声明事务管理器bean，然后配置驱动，表明这些事务都由这个事务管理器去接管，最后@Transactional， @Transactional 注解只能应用到接口方法、类、还有public方法 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">
 
    <context:annotation-config />
    <context:component-scan base-package="com.bluesky" />
 
    <tx:annotation-driven transaction-manager="transactionManager"/>
 
    <bean id="sessionFactory" 
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean"> 
        <property name="configLocation" value="classpath:hibernate.cfg.xml" /> 
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
    </bean> 
 
    <!-- 定义事务管理器（声明式的事务） --> 
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
 
</beans>

```

此时在DAO上需加上@Transactional注解，如下：

```java
package com.bluesky.spring.dao; import java.util.List; import org.hibernate.SessionFactory;import org.springframework.beans.factory.annotation.Autowired;import org.springframework.orm.hibernate3.support.HibernateDaoSupport;import org.springframework.stereotype.Component; import com.bluesky.spring.domain.User; @Transactional
@Component("userDao")
public class UserDaoImpl extends HibernateDaoSupport implements UserDao {     
    public List<User> listUsers() {        
        return this.getSession().createQuery("from User").list();
    }  
```





**RequestParam**  汉语意思就是： **请求参数**。顾名思义 就是获取参数的  **?name=1**

**PathVariable**  汉语意思是：**路径变量**。顾名思义，就是要获取一个url 地址中的一部分值，那一部分呢？**/emp/1**





### 事务&AOP&代理

先大致说一下理解：

通过上面我们知道Spring支持两种事务，分别是编程式事务和声明式事务，声明式事务是通过AOP实现的，AOP是面向切面编程，我们可以使用这个方式在运行代码的前后执行一些其他的代码，那么如果使用AOP来实现事务的话，就是在我们的事务方法前后拦截下来然后给他加上一个事务，执行完之后再commit以达到实现事务的目的，而AOP为了获得对事务方法的控制权，就使用到了动态代理。

所以对于声明式事务，简单的一个执行步骤是：

**xml配置**:**配置数据源**，**配置事务管理器**（依赖于数据源），**配置一个事务及其属性**（依赖于事务管理器，然后设置传播级别等，method属性可以针对不同的切入点单独配置某个事务的属性），**配置切入点**（依赖于前面声明的事务，就是说被切入的点就变成了事务方法），最后一步可以看出，就是一般的AOP，所以说实现事务是AOP的功能的一个子集。

**注解配置**：前两步相同，在相应方法上使用@Transactional注解，并直接设置事务属性，这样的好处是，精准控制切入点，并且不同的事务的属性也不相同。

那么编程式事务，又是怎样实现的呢？是通过数据库实现的：

基本上就是，首先我要创建一个**事务管理器（管理不同的事务就用不同的事务管理器），依赖于数据源，数据源又依赖于数据库**，那么我们的事务的创建、提交、回滚都由这个事务管理器来进行操作（注意这一步是在方法内搞定的）。`TransactionTemplate`在使用前也需要经过这样的配置。

#### 三种代理(写在JavaSE)

代理模式的关键点是:代理对象与目标对象.**代理对象是对目标对象的扩展**,并会调用目标对象。





### SSM整合流程

1. web.xml 配置启动容器配置文件信息，配置DispatcherServlet ,一个就够，将页面请求交给他处理，配置Listener、Filter.
2. DispatcherServlet-servlet.xml  配置基于注解的映射，视图解析器，还有一些配置信息。
3. applicationContext.xml  Spring主配置，配置数据源，配置Mybatis会话工厂（指定Mapper文件位置），配置事务管理器，开启基于注解的事务管理，配置对不同方法的事务属性。
4. Mybatis配置文件的编写，主要是设置全局Mybatis属性，DAO接口及Mapper实现可以交给逆向工程完成，然后在此基础上再做修改。



以下是SSM整合过程中的一些新知识

- 关于Mybatis插件，可以实现分页功能。开始一个分页之后的查询自动会变为分页查询，而查询出来的结果可以再次被封装起来，里面就有很多分页信息。比如当前在第几页，一共有多少条记录。



- 改造为ajax，通过使用json数据，可以提高可扩展性。

  ![1567314845312](C:\Users\H\AppData\Roaming\Typora\typora-user-images\1567314845312.png)

  - 我们可以将很多自定义的信息封装到一个自定义类中，比如成功失败消息，状态码等等，为了能在这个类中放入多个对象，我们可以在里面放一个Map用来保存对象。这里有一个小小的编程技巧：实现**链式编程**

  - ```java
    public Msg int(String key,Object value){
        this.getMap().put(key,value);
        return this;
    }
    //使用
    Msg msg = new Msg(code,massage).add(key1,value1).add(key2,value2);
    
    ```

  - 然后怎么去利用ajax呢，ajax是一个异步加载请求，像一般的话，我们的页面在显示到我们的屏幕之前就已经把数据和jsp完全结合好了，再来显示。那么使用Ajax的话，我们就可以先把框架先加载完（没有任何数据），然后通过jquery发送ajax请求数据，再将数据跟我们的页面做整合，在成功的回调函数中整合数据和模板。

- 数据校验应该遵循 前端jquery格式校验->ajax用户名重复校验->后端校验(重要！！保证数据的正确性不被别人篡改)
  



### 一些常见的注解的作用

1. @Auto wired 和 @Resource

   1. @Auto wired是来自Spring的，默认按类型装配，想使用名称装配可以结合@Qualifier注解进行使用。

      `@Autowired()@Qualifier("baseDao")`

   2. @Resource（这个注解属于J2EE的）,默认按名字装配，若没有同名的，才会找类型相同的。

2. @Configuration 见得多啦，表示是一个配置类，里面有很多Bean.

3. @ComponentScan 注解。该注解默认会扫描该类所在的包下所有的配置类，相当于之前的 <context:component-scan>

4. @Bean(initMethod=”init”,destroyMethod=”destroy”)定义，在构造之后执行init，在销毁之前执行destroy。

5. @PropertySource 可以指定加载的配置文件

6. @EnableAsync 开启异步任务支持。注解在配置类上。

   @Async 注解在方法上标示这是一个异步方法，在类上标示这个类所有的方法都是异步方法。每次执行这个方法都会把他丢进线程中执行，前提是你的容器里有线程池Bean(即Executor类型对象，关于线程池的创建参考锁那个md)

7. @ResponseBody  映射web请求（访问路径和参数），处理类和方法

   @RequestBody  返回值放在response体内。返回的是数据而不是页面,也可以返回json数据，但是需要jackson支持

   @RestController

8. @ImportResource  对xml文件的支持，这个注解就是用来加载xml配置的

9. @Autowired 按类型注入，可以是接口

10. @Resource 按名称注入，这是java自带的注解

11. @Qualifier 增强Autowired ，使其按名称注入

12.  **@Service，@Controller 这些注解要放在接口的实现类上，而不是接口上面** 

13. @Autowired和@Resource是用来修饰字段，构造函数，或者设置方法，并做注入的。 

14. RequestMapping

15. @requestParam：用于指定get方式的？参数，可以设定默认值

16. @PathVariable:用于获取路径参数就是RequestMapping中指定的参数映射，而不是地址栏中的？






### 如何自定义一个注解

- **第一步，定义注解——相当于定义标记；**
- **第二步，配置注解——把标记打在需要用到的程序代码中；**
- **第三步，解析注解——在编译期或运行时检测到标记，并进行特殊操作**



首先是**定义注解：**

1. 定义一个注解使用@interface关键字，这个注解会继承**java.lang.annotation.Annotation**接口

```java
public @interface CherryAnnotation {
}
```

2. 实现注解，其实现部分只能定义一个东西：注解类型元素（annotation type element），看起来就像定义接口内方法一样：

   ```java
   //这就是一个最最最简单的注解
   public @interface CherryAnnotation {
   	public String name();
   	int age() default 18;
   	int[] array();
   }
   ```

   定义注解类型元素时需要注意如下几点：

   - 访问修饰符必须为**public**，不写默认为public；
   -  该元素的类型只能是**基本数据类型**、String、Class、枚举类型、**注解类型（体现了注解的嵌套效果）**以及上述类型的一位数组；
   - 该元素的名称一般定义为名词，如果注解中只有一个元素，请把名字起为value（后面使用会带来便利操作）；
   - ()不是定义方法参数的地方，也**不能在括号中定义任何参数，仅仅只是一个特殊的语法**；
   - default代表默认值，值必须和第2点定义的类型一致；
   - 如果没有默认值，代表**后续使用注解时必须给该类型元素赋值**。

3. 如何确定某个注解所能标注的元素类型 **@Target**

   ```java
   //@CherryAnnotation被限定只能使用在类、接口或方法上面
   @Target(value = {ElementType.TYPE,ElementType.METHOD})
   public @interface CherryAnnotation {
       String name();
       int age() default 18;
       int[] array();
   }
   //我们看一些ElementType这个枚举类型里有什么
   public enum ElementType {
       /** 类，接口（包括注解类型）或枚举的声明 */
       TYPE,
       //属性声明
       FIELD,
       METHOD,
       PARAMETER,
       CONSTRUCTOR,
       //局部变量声明
       LOCAL_VARIABLE,
       /** 注解类型声明 */
       ANNOTATION_TYPE,
       PACKAGE
   }
   ```

4. 如何确定一个注解的生命周期

   ```java
   public enum RetentionPolicy {
       /**
        * （注解将被编译器忽略掉）
        */
       SOURCE,
   
       /**
        * （注解将被编译器记录在class文件中，但在运行时不会被虚拟机保留，这是一个默认的行为）
        */
       CLASS,
   
       /**
        * @see java.lang.reflect.AnnotatedElement
        */
       RUNTIME
   }
   ```

   - 如果一个注解被定义为RetentionPolicy.SOURCE，则它将被限定在Java源文件中，那么这个注解即不会参与编译也不会在运行期起任何作用，这个注解就和一个注释是一样的效果，只能被阅读Java文件的人看到；
   - 如果一个注解被定义为RetentionPolicy.CLASS，则它将被编译到Class文件中，那么编译器可以在编译时根据注解做一些处理动作，但是运行时JVM（Java虚拟机）会忽略它，我们在运行期也不能读取到；
   - 如果一个注解被定义为RetentionPolicy.RUNTIME，那么这个注解可以在运行期的加载阶段被加载到Class对象中。那么在程序运行阶段，我们可以通过反射得到这个注解，并通过判断是否有这个注解或这个注解中属性的值，从而执行不同的程序代码段。我们实际开发中的自定义注解几乎都是使用的**RetentionPolicy.RUNTIME**；
   - 在**默认**的情况下，自定义注解是使用的**RetentionPolicy.CLASS**。

5. 如果一个类使用了自定义注解，那么继承他的类会不会也获得这个注解呢。**@Inherited**

   @Inherited注解，是指定某个自定义注解如果写在了父类的声明部分，那么子类的声明部分也能自动拥有该注解。@Inherited注解只对那些@Target被定义为ElementType.TYPE的自定义注解起作用，也就是说肯定只有能标注到类上面的才能用。

   

然后第二步，**配置注解**

1. 根据它所能标注的方法，在相应的类型上标注。并且要对我们定义的这些注解类型元素赋值，如下：

   ```java
   public class Student {
       @CherryAnnotation(name = "cherry-peng",age = 23,score = {99,66,77})
       public void study(int times){
           for(int i = 0; i < times; i++){
               System.out.println("Good Good Study, Day Day Up!");
           }
       }
   }
   ```

2. 如果注解本身没有注解类型元素，那么在使用注解的时候可以省略()，直接写为：**@注解名**，它和标准语法@注解名()等效！

3. 如果注解本本身只有一个注解类型元素，而且命名为**value**(作用体现出来了），那么在使用注解的时候可以直接使用：**@注解名(注解值**)，其等效于：**@注解名(value = 注解值)**

4. 如果注解中的某个注解类型元素是一个数组类型，在使用时又出现只需要填入一个值的情况，那么在使用注解时可以直接写为：**@注解名(类型名 = 类型值)**，它和标准写法：**@注解名(类型名 = {类型值})**等效！

5. 如果一个注解的@Target是定义为Element.PACKAGE，那么这个注解是配置在package-info.java中的，而不能直接在某个类的package代码上面配置。



最后第三步，**解析注解**

1. 注解的保持力处于运行阶段，即使用`@Retention(RetentionPolicy.RUNTIME)`修饰注解时，才能在JVM运行时，检测到注解，并进行一系列特殊操作。

   ```java
   public class TestAnnotation {
       public static void main(String[] args){
           try {
               //获取Student的Class对象
               Class stuClass = Class.forName("pojos.Student");
   
               //说明一下，这里形参不能写成Integer.class，应写为int.class
               Method stuMethod = stuClass.getMethod("study",int.class);
   	
               if(stuMethod.isAnnotationPresent(CherryAnnotation.class)){
                   System.out.println("Student类上配置了CherryAnnotation注解！");
                   //获取该元素上指定类型的注解
                   CherryAnnotation ca = stuMethod.getAnnotation(CherryAnnotation.class);
                   System.out.println("name: " + ca.name() + ", age: " + ca.age()
                       + ", score: " + ca.score()[0]);
               }else{
                   System.out.println("Student类上没有配置CherryAnnotation注解！");
               }
           } catch (ClassNotFoundException e) {
               e.printStackTrace();
           } catch (NoSuchMethodException e) {
               e.printStackTrace();
           }
       }
   }
   
   ```

   - 如果我们要获得的注解是配置在方法上的，那么我们要从Method对象上获取；如果是配置在属性上，就需要从该属性对应的Field对象上去获取，如果是配置在类型上，需要从Class对象上去获取。总之在谁身上，就从谁身上去获取！
   
   - isAnnotationPresent(Class<? extends Annotation> annotationClass)方法是专门判断该元素上是否配置有某个指定的注解；
   
   - getAnnotation(Class<A> annotationClass)方法是获取该元素上**指定的**注解。之后再调用该注解的注解类型元素方法就可以获得**配置时的值数据**；
   
   - 反射对象上还有一个方法getAnnotations()，该方法可以获得该对象身上配置的所有的注解。它会返回给我们一个注解数组，需要注意的是该**数组的类型是Annotation类型**，这个Annotation是一个来自于java.lang.annotation包的接口。
   
     

#### 如何使自定义注解实现某些特定的功能

那么我们首先要明白，注解可以在运行中通过反射来获得我们所设定的值，但是我们要实现具体的功能，注解自身是做不到的，所以是我们要去利用这些信息使用AOP去实现功能。

1. 创建自定义注解(可被标注在变量字段上的)，注解类型元素就是一些类名、方法名等，表示需要借助他们的帮助
2. 创建自定义注解，target为方法，用于标注在特定方法上
3. 创建一个切面，使用@Aspect标签，编写增强代码，在这个增强函数中使用@Around("@annotation(====被增强的代码的注解====)")，那么增强代码就可以获得被增强函数运行的结果。  
4. 获得结果后，分析结果的元素中是否有我们第一步自定义的注解，如果有的话那我们就执行增强方法。

所以大概的流程是：一个函数使用方法注解，在调用方法之后，增强方法（切面）会获得他的结果，然后在利用属性注解的信息去处理结果。





#### IOC容器的初始化（比较重要再看一遍）

首先我们要知道，有很多种IOC容器，所谓初始化，就是实例化一个ApplicationContext，不同的容器的初始化方式也不尽相同。

大致分为如下几步

1. BeanDefinition 的 Resource 定位：这里的Resource定位 是通过继承ResourceLoader 获得的，ResourceLoader代表了**加载资源的一种方式，正是策略模式的实现**。

2. 从 Resource中解析、载入BeanDefinition

3. BeanDefinition 在IoC 容器中的注册


具体来说一下比较重要的步骤：

1. 创建容器前的准备工作：记录下容器的启动时间、标记“已启动”状态、处理配置文件中的占位符，校验配置文件信息
2. 创建Bean容器，加载Bean，注册Bean：首先关闭原来的Bean容器，然后新建一个Bean容器，接着解析XML文件，解析出有什么Bean, 然后**注册Bean**。
3. 实例化所有singleton的Bean,不包括延迟加载的





## AOP

主要应用场景有：

- **权限校验**
- **缓存**
- **错误处理**
- 懒加载
- 记录跟踪，优化
- 持久化
- 同步（redis全局锁）
- **事务**
- **日志**

核心概念：

- 切面：对**横切关注点**的抽象

- 横切关注点：拦截哪些方法，如何处理

- joinpoint : 仅支持方法类型的连接点

- pointcut：你的切面如何跟连接点去进行匹配，这里需要学习切入点表达式，缺省使用AspectJ

- advice: 到了切入点之后要执行的代码

  - 前置、后置（不论是正常返回还是异常退出）、异常、最终（正常返回）、环绕

- 目标对象：代理的目标对象，你要增强这个方法那么就需要创建一个代理对象

- AOP代理：可以是JDK代理或cglib代理，默认的策略是如果目标类是接口， 

  则使用 JDK 动态代理技术，否则使用 Cglib 来生成代理。 

- 织入：将切面应用到目标对象并导致代理对象创建的过程

JDK动态代理：InvocationHandler是一个接口，通过实现该接口定义横切逻辑，并通过反射机制调用目标类 的代码，动态将横切逻辑和业务逻辑编制在一起。Proxy 利用 **InvocationHandler 动态创建 的一个符合某一接口的实例**，生成目标类的代理对象。

cglib：运行期动态生成新 的 class，相当于创建一个新的class去继承被代理对象





### SpringMVC

基本处理请求的流程：

1. Http请求到DispatcherServlet
2. DispatcherServlet去查询多个HandlerMapping，查找处理请求的Controller
3. 把请求提交给Controller（具体一点就是上一步的HandlerMapping找到了合适的Controller之后并不会直接把请求给到他而是返回一个处理链给DispatcherServlet，然后DispatcherServlet再将这个链给到 HandlerAdapter，在由它给到控制器）
4. 处理完业务逻辑后，返回ModelAndView（Controller->HandlerAdapter->DispatcherServlet）
5. DispatcherServlet使用ModelAndView查询ViewResolver查找对应的视图对象
6. 渲染后将视图给回Http

相关面试题：

优点：多种视图技术，与Spring继承（就拥有了AOP IOC等特点），角色比较清晰



### SpringBoot

用来简化spring应用的初始搭建以及开发过程 使用特定的方式来进行配置（properties或yml文件） 
创建**独立的spring引用程序** main方法运行 
**嵌入的Tomcat** 无需部署war文件 
简化maven配置 
自动配置spring添加对应功能starter自动化配置 
答：spring boot来简化spring应用开发，约定大于配置 

  



