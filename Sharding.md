为了数据分片而服务，我们的重点放在分表上而并不分库



绑定表：指的是主表和子表都使用同一个字段来分表，比如说主表保存了所有数据而子表只保存了id和对应的密码等一小部分的信息（在业务上的需求用于比较频繁读取数据库的操作比方说用于登录），那么在分表的时候需要两张表同时分并且两张表的分表规则是一样的。字段不相同或者规则不相同都不能算是绑定表。

如果不配置绑定表关系的话，如果我们想对子表和主表联表查的话，那么将会把主表和子表的所有分表通通join到一起去查（笛卡尔积），假设主表有五片，子表只有四片，那么就会产生二十条查询语句，而如果两个都是按照首字母分片，那么只会根据你的查询条件，涉及到多少个分片就只查多少次。

```sql
SELECT i.* FROM t_order o JOIN t_order_item i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);
```

 其中`t_order`在FROM的最左侧，ShardingSphere将会以它作为整个绑定表的主表。 所有路由计算将会**只使用主表**的策略，那么`t_order_item`表的分片计算将会使用`t_order`的条件。故绑定表之间的分区键要完全相同。 也就是说只查主表，从表联动。

分片键 ，用于分片的数据库字段，ShardingSphere也支持以多个字段进行分片

分片算法：

- 精确分片 （比较常用）
- 范围分片
- 复合分片
- Hint分片

分片策略：是分片键和分片算法的结合



配置:我们该如何配置呢

首先我们可以根据自己的需求将数据划分到不同的数据源中的不同的表，可以使用均匀配置也可以自定义配置

分片策略配置：

- 数据源分片策略配置
- 分表策略配置



 ShardingSphere的3个产品的数据分片主要流程是完全一致的。 核心由`SQL解析 => 执行器优化 => SQL路由 => SQL改写 => SQL执行 => 结果归并`的流程组成。 

涉及到分片数据的查询，理解上面的过程就是查询的时候，首先要考虑让那些表去执行这些语句，然后对原查询语句做一些修改，查出结果后再将结果整合一下

- 解析引擎：解析SQL语句，对于不同的语言，会使用不同的解析引擎

- 路由引擎：根据语句所匹配的数据库，生成路由路径，也就是哪里有所需的数据就去哪里查，如果这个语句不带分片键，那么就会执行广播路由，另外还有单播路由，这是为了获取数据库的信息，那么只从一个库里获得就行了

- 改写引擎： 工程师面向逻辑库与逻辑表书写的SQL，并不能够直接在真实的数据库中执行 ， 需要将分表配置中的**逻辑表名称**改写为路由之后所**获取的真实表**名称 

  - 正确性改写：改查询的表名，列名。查询条件等等，目的就是能在正确的表中获取到数据

  - 补列：如果说查询的字段中不包含某个字段，但查询结果又 GROUP BY和ORDER BY 这个字段，那么在经过正确性改写后是不会查询这个列的，因此我们需要补列来获取这一部分的信息，表现为select多一个字段用于获取归并信息

    又或者是计算平均值时，需要将AVG 改写为 先查询SUM和COUNT，然后在归并这一步将他们再计算平均值 

    又或者是在执行插入语句时，对于自增主键，如果在同一张表里好实现，但是在分布式的表中就不是那么容易了，补列其实就是通过Sharding在插入时去添加一个主键，从而跳过原有的自增主键的策略

    ```mysql
    SELECT o.*, order_item_id AS ORDER_BY_DERIVED_0 FROM t_order o, t_order_item i WHERE o.order_id=i.order_id ORDER BY user_id, order_item_id;
    //改写不一定是一步达到目的
    SELECT COUNT(price) AS AVG_DERIVED_COUNT_0, SUM(price) AS AVG_DERIVED_ SUM_0 FROM t_order WHERE user_id=1;
    ```

  - 分页修正：分页查询在分布式数据表中也会是一个问题，如果说我们想要对排序后的结果进行分页查询，那么最终结果可能全部来自某个表，因此不能将分页后的结果进行归并，只能通过分页修正获取较多的数据，然后再做归并再做分页

    -  越获取偏移量位置靠后数据，使用LIMIT分页方式的效率就越低 

- 归并引擎：功能上分为遍历 排序 分组 分页 聚合五种类型，结构上分为内存归并 流式归并  装饰者归并 

  - 遍历归并：单向链表一样
  - 排序归并：用于ORDER BY语句，首先会将子集各自排好序，再做归并
  - 

说了这么多，sharding也不是适应所有的数据库以及所有的SQL语句的，也是有一些是没办法支持的

对于分表，支持数据定义语言DDL，数据操作语言DDL，数据控制语言DCL，事务控制语言TCL，和部分DAL

对于分表， 不支持CASE WHEN、HAVING、UNION (ALL)， 子查询只支持一层嵌套。 简单来说，通过子查询进行非功能需求，在大部分情况下是可以支持的。比如分页、统计总数等；而通过子查询实现业务查询当前并不能支持。 





### 使用教程

Spring

1. 导入Sharding依赖和对应xml的命名空间依赖
2. yml配置分表键和分表算法
3. 或者在xml中配置

```xml
<sharding:data-source id="shardingDataSource">
        <sharding:sharding-rule data-source-names="ds0,ds1">
            <sharding:table-rules>
                <sharding:table-rule logic-table="t_order" actual-data-nodes="ds$->{0..1}.t_order$->{0..1}" database-strategy-ref="databaseStrategy" table-strategy-ref="orderTableStrategy" />
                <sharding:table-rule logic-table="t_order_item" actual-data-nodes="ds$->{0..1}.t_order_item$->{0..1}" database-strategy-ref="databaseStrategy" table-strategy-ref="orderItemTableStrategy" />
            </sharding:table-rules>
        </sharding:sharding-rule>
    </sharding:data-source>
```

一个比较详细的xml配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:sharding="http://shardingsphere.apache.org/schema/shardingsphere/sharding"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://shardingsphere.apache.org/schema/shardingsphere/sharding
                        http://shardingsphere.apache.org/schema/shardingsphere/sharding/sharding.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context.xsd
                        http://www.springframework.org/schema/tx
                        http://www.springframework.org/schema/tx/spring-tx.xsd">
    <context:annotation-config />

    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="shardingDataSource" />
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter" p:database="MYSQL" />
        </property>
        <property name="packagesToScan" value="org.apache.shardingsphere.example.core.jpa.entity" />
        <property name="jpaProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
                <prop key="hibernate.hbm2ddl.auto">create</prop>
                <prop key="hibernate.show_sql">true</prop>
            </props>
        </property>
    </bean>
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager" p:entityManagerFactory-ref="entityManagerFactory" />
    <tx:annotation-driven />

    <bean id="ds0" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/ds0" />
        <property name="username" value="root" />
        <property name="password" value="" />
    </bean>

    <bean id="ds1" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/ds1" />
        <property name="username" value="root" />
        <property name="password" value="" />
    </bean>

    <bean id="preciseModuloDatabaseShardingAlgorithm" class="org.apache.shardingsphere.example.algorithm.PreciseModuloShardingDatabaseAlgorithm" />
    <bean id="preciseModuloTableShardingAlgorithm" class="org.apache.shardingsphere.example.algorithm.PreciseModuloShardingTableAlgorithm" />

    <sharding:standard-strategy id="databaseShardingStrategy" sharding-column="user_id" precise-algorithm-ref="preciseModuloDatabaseShardingAlgorithm" />
    <sharding:standard-strategy id="tableShardingStrategy" sharding-column="order_id" precise-algorithm-ref="preciseModuloTableShardingAlgorithm" />

    <sharding:key-generator id="orderKeyGenerator" type="SNOWFLAKE" column="order_id" />
    <sharding:key-generator id="itemKeyGenerator" type="SNOWFLAKE" column="order_item_id" />

    <sharding:data-source id="shardingDataSource">
        <sharding:sharding-rule data-source-names="ds0,ds1">
            <sharding:table-rules>
                <sharding:table-rule logic-table="t_order" actual-data-nodes="ds$->{0..1}.t_order$->{0..1}" database-strategy-ref="databaseShardingStrategy" table-strategy-ref="tableShardingStrategy" key-generator-ref="orderKeyGenerator" />
                <sharding:table-rule logic-table="t_order_item" actual-data-nodes="ds$->{0..1}.t_order_item$->{0..1}" database-strategy-ref="databaseShardingStrategy" table-strategy-ref="tableShardingStrategy" key-generator-ref="itemKeyGenerator" />
            </sharding:table-rules>
            <sharding:binding-table-rules>
                <sharding:binding-table-rule logic-tables="t_order, t_order_item" />
            </sharding:binding-table-rules>
            <sharding:broadcast-table-rules>
                <sharding:broadcast-table-rule table="t_config" />
            </sharding:broadcast-table-rules>
        </sharding:sharding-rule>
    </sharding:data-source>
</beans>
```

