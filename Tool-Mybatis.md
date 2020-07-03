### Mapper

```xml
在Mapper中需要显式匹配接口
<mapper namespace="com.atguigu.crud.dao.DepartmentMapper">
</mapper>
```





### 全局配置文件

xml配置结构，需注意先后关系

- configuration（配置）
  - properties（属性） 相当于一个配置文件，保存一些变量的值，后续可以使用它。也可以指定一个外部的properties文件或yml
  - settings（设置）
    - 是否开启缓存
    - 懒加载
    - 设置超时时间
  - typeAliases（类型别名）
    - 指的是为java类名生成一个别名
  - typeHandlers（类型处理器）
    - 配置java类型和jdbc数据类型的转换，用于返回结果和preparedStatement，一般都映射好了
    - 可以实现typeHandler接口去自定义
  - objectFactory（对象工厂）
  - plugins（插件）
    - 添加分页助手这样的插件
  - environments（环境配置）
    - environment（对应多种环境可以设置不同的环境变量）
      - transactionManager（事务管理器）
      - dataSource（数据源）这个可以在Spring的配置文件中配置
  - databaseIdProvider（数据库厂商标识）
  - mappers（映射器）
    - 可以单单指定一个mapper也可以映射一个包

1. properties（属性） 相当于一个配置文件，保存一些变量的值，在整个配置文件中可以使用，以及`SqlSessionFactoryBuilder.build()`的参数中。

   读取配置顺序是，先xml文件里的，然后再是resource指定的配置文件路径。

2. settings  :这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的**运行时行为**。 下表描述了设置中各项的意图、默认值等。

   lazyLoadingEnabled:  延迟加载，

   useGeneratedKeys：自动生成主键

   mapUnderscoreToCamelCase:  是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。

3. typeAliases（类型别名）:为一个Java类（包名.类名）指定一个别名，有两种方式，一是直接为别名指定类名`<typeAlias alias="Author" type="domain.blog.Author"/>`,二是指定所在包，包内的类使用@Alias注解为自己使用别名。

4. typeHandlers（类型处理器）当我们查询出数据库结果或者保存java对象到数据库时，涉及类型的转换。

   你可以重写类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型。 具体做法为：实现 `org.apache.ibatis.type.TypeHandler` 接口， 或继承一个很便利的类 `org.apache.ibatis.type.BaseTypeHandler`， 然后可以选择性地将它映射到一个 JDBC 类型。比如：

   `@MappedJdbcTypes(JdbcType.VARCHAR)`,接着你就可以在配置文件中添加这个处理器以取代默认的处理器。

5. plugins （插件）：有点像AOP编程，通过一个实现了`Interceptor`接口的方法来实现插件，使用

   ``` java
   @Intercepts({@Signature(
     type= Executor.class,
     method = "update",//表示拦截update方法,对其增强
     args = {MappedStatement.class,Object.class})})
   ```

6. mappers 指定映射文件，可以每个mapper单独设置，也可以配置包扫描。





### 映射文件

映射文件只有很少的几个顶级元素（按照应被定义的顺序列出）：

- `cache` – 对给定命名空间的缓存配置。

- `cache-ref` – 对其他命名空间缓存配置的引用。

- `resultMap` – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。

- `sql` – 可被其他语句引用的**可重用语句块**。

- `insert` – 映射插入语句

  - `selectKey` 通过这个语句在执行增删查改之前确定主键的取值，实际用途是针对Oracle这样的没有自增主键功能的数据库，可以在插入之前先通过查询序列化的值作为主键，再做插入。

  - ```xml
      <selectKey keyProperty="id" resultType="int" order="BEFORE">
        select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
      </selectKey>
    ```

- `update` – 映射更新语句

- `delete` – 映射删除语句

- `select` – 映射查询语句

  - `resultType`从这条语句中返回的期望类型的类的完全限定名或别名。 注意如果返回的是集合，那应该设置为**集合包含的类型**，而**不是集合本身**,  可以使用 resultType 或 resultMap，但不能同时使用.

    



#### 一些要知道的点

1. 增删改方法（接口内写返回类型即可，无需在映射文件中写）可以返回整型、长整型和布尔类型，表示影响的行数和是否成功。
2. 关于自增主键会有这样的一个问题，你的数据库使用了自增主键，因此你在插入一个主键为null的对象时，数据库会为你这个数据自动生成一个主键。**但是你这个对象在jvm中依然是没有主键的**，那么我们可以通过在<insert>标签内将`useGeneratedKeys`设置为true，并将这个自增主键的值绑定到对象的某个属性上，那么当你执行插入时，他会返**回这个对象在数据库中的主键的值**并赋值到该对象的对应属性上。
3. 参数处理
   1. 单个参数：mybatis不会做特殊处理，意思就是，如果你的方法只传入一个参数，那么在你的映射文件实现中**#{}**怎么写都可以，都是取出这个值。
   2. 多个参数：会做特殊处理，你的参数在映射文件实现中会被转换为param1，param2，也就是说**#{param1}表示第一个参数**.
   3. **命名参数（推荐）**: 在接口方法参数前使用``@param("paraname")`，这样的话在映射文件实现时就可以使用**#{paraname}**表示对应哪个参数。
   4. POJO：如果多个参数又恰好是某个对象的属性，那么直接传入一个对象，在实现时直接使用**#{类的属性名}**就可以访问属性。
   5. 直接传入Map:  使用**#{KEY}**取出Map中对应属性的值。
   6. 或者直接将常用的参数集合封装为一个对象。
   7. Collection（List，Set） 或数组也会特殊处理，使用collection[0]，list[0]  array[0] 来访问集合内元素。
   8. 使用#{} 和 ${}  取参数值的区别，**#{} 预编译，类似占位符，防止sql注入，大多数情况下使用**，${}取出的值直接拼接，会有sql注入风险，但是如果原生jdbc不支持占位符的话那么就需要用${}，比如排序（order by）、表名、升降序等 。
4. **Select**
   1. 如果ResultType是map，那么查询出来的每个属性对应的值会封装成map中的一个键值对。
   2. 多个查询出来的对象封装到Map中，key是自定义属性，value是对象，那么可以声明resultType为类名，然后接口内的方法使用@MapKey("属性名")来指定封装的Map用什么作为key

5. **ResultMap**

   1. 我们先回到一个问题上，我们查询到了结果，如果这个结果能代表一个对象，那么自然就要将数据库中每一列跟我们的类的属性一一关联起来，这个时候我们如果开启了驼峰命名转换规则的话，那么就可以关联起来，但是为了功能更强大，我们会使用ResultMap

   2. 场景一：多表查询，我们的查询结果不可能总是一个对象，肯定要涉及多个表的联合查询。

      左边都是数据库的列名，右边就是对应的Java类属性名

      ![1567493236807](C:\Users\H\AppData\Roaming\Typora\typora-user-images\1567493236807.png)

      这样就将每一列一一对应到Bean的属性中

      也可以使用联合<association>：

      ![1567494298733](C:\Users\H\AppData\Roaming\Typora\typora-user-images\1567494298733.png)

      又或者，这个联合属性的值是通过另一条查询语句获得的。

    3. 场景二：一对多，一个对象里使用List保存多个同类型对象，一个部门有多个员工。

       <collection property="emps" ofType="Employee">
   
       ![1567497975771](C:\Users\H\AppData\Roaming\Typora\typora-user-images\1567497975771.png)
   
       
   
         
   
   ​      
   

### 动态SQL

- if    `<if test=""></if>`使用OGNL表达式设置条件
  - id!=null
  - lastName!=null and lastName!=&quot；&quot；
  - 如果出现拼装有问题，比如where后面第一个条件为空，有两种解决方案：
    - 在where后添加第一个条件为1=1，即恒成立，再将后面的条件用and拼接起来。
    - 使用<where></where>标签将条件全部包裹起来，然后去掉原来的where，这样的话，会自动去掉第一个and或者or。
  - 使用set更新数据时，使用if的话，会出现多出逗号的情况，这时候可以使用<set></set>将设置项包括起来，然后去掉原来的set就可以了。
- choose   带break的switch
  - <choose>  <where test=""></where><where test=""></where>  <otherwise></otherwise></choose>
- trim（很好用的）  
  - <trim  prefix="" prefixOverrides="" suffix="" suffixOverrides="" ></trim> 
  - 分别是添加前缀，去掉前缀，添加后缀，去掉后缀
- foreach 
  - <foreach collection=""  item="" separator=""></foreach>
  - collection 要遍历的集合
  - item 复制给指定的变量
  - separator  每个元素之间的分隔符
  - open 为遍历出来的结果拼接一个开始字符（不是每一个变量哦）
  - close 为遍历出来的结果拼接一个结束字符
  - 可用于**批量保存**
- 两个内置参数
  - _parameter:代表整个参数
    - 单个参数：就是这个参数
    - 多个参数：代表被封装的map
  - _databaseId: 如果配置了DatabaseIdProvider标签，就代表当前数据库别名。
- bind 绑定OGNL表达式到一个变量中，方便以后使用。





### 缓存

默认定义了两级缓存

1. 默认情况下，只有一级缓存（基于SqlSession级别的缓存，也称为本地缓存）开启。
2. 二级缓存需要手动开启和配置，基于namespace级别的缓存，全局缓存。
3. 可以通过实现Cache接口来已定义二级缓存。



**一级缓存：**一个Session对应一个一级缓存

1. 同一次会话期间查询到的数据放在本地缓存中
2. 下一次要获取相同数据，就在缓存中找。
3. 一级缓存的失效情况：
   1. 不同的Session
   2. 不同的查询条件（没有这个数据）
   3. 会话相同，在两次相同查找中间进行了增删改
   4. 会话相同，手动清除一级缓存   

**二级缓存**：一个namespace对应一个二级缓存，namespace就是不同的对象，比如员工有一个缓存，部门也有一个缓存。

1. 二级缓存的数据来源是一级缓存，一级缓存对应的**会话提交或关闭**后才会保存到二级缓存中的，那新的会话查询信息就可以使用二级缓存。
2. 使用：
   1. 在全局配置文件的settings中开启cacheEnable属性
   2. 在映射文件中mapper标签内配置<cache></cache>标签
      1. eviction:缓存的回收策略
      2. flushInterval：缓存刷新间隔，默认不清空
      3. readOnly:  是否只读。是的话直接将引用交给用户，但是用户可以修改这个引用，不安全，但是快。否则会序列化再给用户。
      4. size
      5. type
   3. POJO需要实现序列化接口

**精确配置**

1. 首先一级缓存是一直开启的。
2. 如果开启了二级缓存，但是<select></select>标签的useCache="false"，那么依然不会使用二级缓存。
3. 增删改标签的flushCache="true"，执行完后会清空一、二级缓存。
4. 查询标签flushCache默认为false,  若flushCache="true"，执行查询后会清空一、二级缓存。
5. sqlSession.clearCache 只会清空一级缓存而不会清空二级缓存。