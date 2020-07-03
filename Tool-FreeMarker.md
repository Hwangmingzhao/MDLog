一般是用来生成**静态文件**的，不止可以生成html文件还可以生成java文件



### 标签

1. 遍历集合<#list collection as  var>   </#list> 
2. <#if   condition>  <#else>  </#if>
3. <#include> </#include>
4. <#assign number2 = 5>   赋值

### 访问变量

1. ${varname}
2. ${date?date} ${date?time}  ${date?datetime}
3.  ${"Hello " + name + " !"} 输出字符串
4. ${name[2]}  ${name[0..5]}截取字符串



#### 变量空判断

 !  　　指定缺失变量的默认值；一般配置变量输出使用
??   　判断变量是否存在。一般配合if使用 <#if value??></#if>

### 内建函数的使用

<#assign data = "abcd1234">
第一个字母大写：${data?cap_first}
所有字母小写：${data?lower_case}
所有字母大写：${data?upper_case}

<#assign floatData = 12.34>
数值取整数：${floatData?int}
获取集合的长度：${users?size}
时间格式化：${dateTime?string("yyyy-MM-dd")}



### Spring使用

将FreeMarkerConfigurer添加到容器内

```xml
<bean id="freemarkerConfig"
        class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    	<!--模板位置-->
        <property name="templateLoaderPath" value="/WEB-INF/ftl/" />
        <property name="defaultEncoding" value="UTF-8" />
        <property name="freemarkerSettings">
            <!-- 设置默认的编码方式，原先是GBK，需要设置成utf-8 -->
            <props>
                    <!--用于解决前端报空指针问题 不用再空值后面+ ！号-->
                <prop key="classic_compatible">true</prop>
            </props>
        </property>
    </bean>
```

处理请求并将数据封装给模板

```java
public String genHtml()throws Exception {
        // 1、从spring容器中获得FreeMarkerConfigurer对象。
        // 2、从FreeMarkerConfigurer对象中获得Configuration对象。
        Configuration configuration = freeMarkerConfigurer.getConfiguration();
        // 3、使用Configuration对象获得Template对象。这里的hello.ftl就是一个模板，加上数据就变成html或.java了
        Template template = configuration.getTemplate("hello.ftl");
        // 4、创建数据集
        Map dataModel = new HashMap<>();
        dataModel.put("hello", "1000");
        // 5、创建输出文件的Writer对象。
        Writer out = new FileWriter(new File("D:/temp/term197/out/spring-freemarker.html"));
        // 6、调用模板对象的process方法，生成文件。
        template.process(dataModel, out);
        // 7、关闭流。
        out.close();
        return "OK";
    }
```

