1. 启动Arthas

```shell
java -jar arthas-boot.jar
```

2. 搜索所有被JVM加载的类

```shell
sc -d 类名
```

![1616740132974](C:\Users\H30008~1\AppData\Local\Temp\1616740132974.png)

3. 反编译类，这样就可以在线看你这个类在环境里到底是以什么样的代码在跑了

```
jad 全路径类名
```

4. 随时监控一个方法的返回值(参数)是啥

```shell
watch 类的全路径名 方法名 returnObject（param）
```

![1616740581457](C:\Users\H30008~1\AppData\Local\Temp\1616740581457.png)

![1616741263054](C:\Users\H30008~1\AppData\Local\Temp\1616741263054.png)

5. 打印所有系统属性（某个属性）（设置某个属性）参数名支持自动补齐

```
sysprop   |   sysprop 参数名    |  sysprop 参数名 参数值
```

6. 打印jvm信息

```
jvm
```

7. 快捷键映射

![1616741457835](C:\Users\H30008~1\AppData\Local\Temp\1616741457835.png)

8. sc 详细用法，如果搜索接口还会展示实现类

![1616741596443](C:\Users\H30008~1\AppData\Local\Temp\1616741596443.png)

支持通配符   sc *StringUtils

9. 查找类的具体函数

```
sm 类的全路径名 
```

10. watch 的够用用法

```
watch com.example.demo.arthas.user.UserController * '{params, returnObj, throwExp}' -x 2
```

可以展示某个方法的入参、返回值、和异常栈

```
watch com.example.demo.arthas.user.UserController * "{params[0],throwExp}" -e
```

只有捕获异常的时候才打印出来



11. 一个超强用法，热更新代码

    1. 首先反编译你想要的类，使用--source-only参数 ，重定向到某个文件里

    2. 查询原本的类的类加载器

       ```bash
       sc -d *UserController | grep classLoaderHash
       ```

    3. 反编译

       ```console
       mc -c 1be6f5c3 /tmp/UserController.java -d /tmp
       ```

    4. 使用`redefine`命令重新加载新编译好的`UserController.class`：

       ```bash
       redefine /tmp/com/example/demo/arthas/user/UserController.class
       ```



12. 当在本地启动时，可以访问 http://127.0.0.1:8563/ ，通过浏览器来使用Arthas。
13. 清理屏幕 cls