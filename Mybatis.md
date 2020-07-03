逆向工程总结

准备工作，创建一个maven项目。

编写generatorConfig.xml文件，包含内容有，指定数据库连接器，与数据库的连接，配置typeresolver，

```xml
<commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true" />
        </commentGenerator>
        <!--数据库连接的信息：驱动类、连接地址、用户名、密码 ,加上“useSSL=false”是因为我SSL连接数据库出现了错误 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://192.168.20.101:3306/risk?serverTimezone=GMT&amp;useUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false"
                        userId="bqs" password="123456">
        </jdbcConnection>
        <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL
            和 NUMERIC 类型解析为java.math.BigDecimal -->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
```



然后重要的地方来了。

配置javaModelGenerator， 即生成pojo对象

```xml
<!-- targetProject:生成pojo类的位置 -->
        <javaModelGenerator targetPackage="pojo"
                            targetProject=".\src\main\java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
            <!-- 从数据库返回的值被清理前后的空格 -->
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
```

配置sqlMapGenerator，即生成映射文件

```xml
 <!-- targetProject:mapper映射文件生成的位置 -->
        <sqlMapGenerator targetPackage="mapper" targetProject=".\src\main\java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </sqlMapGenerator>
```

配置javaClientGenerator，即接口  

```xml
<!-- targetPackage：mapper接口生成的位置 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="mapper" targetProject=".\src\main\java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>
```

最后配置要生成的表信息

```xml
<table tableName="usr_partner" domainObjectName="Partner"
               enableCountByExample="true" enableUpdateByExample="true" enableDeleteByExample="true" enableSelectByExample="true" selectByExampleQueryId="true"
               enableSelectByPrimaryKey="true" enableDeleteByPrimaryKey="true"
        >
        </table>
```

然后参照官方文档解析配置文件，生成所需要的内容。

