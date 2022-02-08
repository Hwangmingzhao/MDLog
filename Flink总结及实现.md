首先，API算子的功能和处理过程分为以下四大步，然后我们再细分各种细节：



### 初始化->采数据->解析数据->往下走



#### 1. 首先我们说一下初始化都初始化了什么

首先，作为一个API算子，核心就是我要调用什么请求，它的请求体是啥，请求头是啥，然后如果在调用接口之前需要先认证，那么大概率我还需要密码，还有用户信息。

（RestApiClient.java 198~205）

然后我们需要识别这个接口所使用的分页模式，分页模式，通俗来讲，就是这个接口要怎么样去调用才能把所有数据全部拿回来。这个识别是通过我们设置好的几个分页宏变量实现的，共有五种：pageNo、seqNo、lastId、firstId、offset 如果都没有使用，那就是callOnce，就是只会查询一次。

（RestApiClient.java 213~253）

在初始化的过程中我们需要保存一些标志位：

1. needAppendUpstreamRow：当前算子的输出是否需要把上游来的数据也拼到一起
2. isInit：是否初始化完成

此外我们还需要将当前算子输出的字段按照是否有jsonPath，区分为上游来的字段还是当前算子的字段（即通过自己的json返回体解析出来的）

（RestApiClient.java 272~290）

最后在我们要去查询数据之前，我们还需要确定当前API算子使用的是哪一种API认证方式，并确定要使用哪一个tokenHandler去获取token。然后配置一下连接管理器的参数什么的。

（RestApiClient.java 311~315）



#### 2. 如何通过API方式查询数据

- 首先我们要初始化一个httpClient对象
- 然后构造我们的请求体 httpRequest
- 最后httpclient.excute(httpRequest)，然后数据就出来了。

先看到buildHttpRequest(Row rowInput) 方法，在构造请求体的时候，我们首先从配置文件中读取请求的信息，第一个是URL，我们需要转换为一个URI对象，这里可以对特殊字符做转义，也适配了历史任务。 第二个是要确定是get还是post请求，如果是post的话还要把请求体带上。 第三个是请求头，首先把用户填写的请求头加上去，一般来说认证信息也是放在请求头中的，我们上一步初始化的tokenHandler会把其对应的请求头参数名和 token返回，追加到请求头参数中。最后我们就把一个请求体构造好了。

接着我们看一下如何取出返回体CloseableHttpResponse 中的数据，很简单 `EntityUtils.toString(CloseableHttpResponse.getEntity())`，将这些数据转换为Flink中的一个个Row之后，我们就可以**开始准备下一次的调用和解析**啦



#### 3. 如何解析JSON类型的返回体作为Row

1. 将JSON体解析为一棵树，从根到叶子节点的树
2. 遍历根节点到叶子节点的每一个路径，并将该路径经过的所有节点保存下来，保存为一个Map，那么一棵树就会变成一个List<Map< String, Object >>，形式就类似于关系型数据库中每行数据一个字段对应一个值。
3. 然后根据FieldConfig，将这些数据按照顺序保存到一个个Row中，最终形成一个List< Row >

由于过程比较复杂，在这里讲一个第二步中的一个结论，如果说一棵树中，某个叶子节点是一个空数组，那么处于同一棵子树中的其他数据将会被置空，在以前我们就是利用这样的特性去判断本次返回的数据是否为空，比如：

```json
{
    "result": "success",
    "member": []
    "returnCode": 123
}
```

像这种情况下，如果说我们理解为，返回的member已经没有了，那么是不是就意味着没有更多数据了，那我们应该结束继续调用这个接口。 但是万一我关注的数据其实是returnCode呢，那是不是还有数据呢，我还需要继续调用呢？   所以为了解决这个问题，我们会有条件地将数据中的空数组修改为 null，这样的话，就不会有空数组情况下数据丢失的问题了。 我们可以对比一下这样修改过后，上述这个json被解析成Map的前后对比

```shell
前：()
后：(result: success, member: null, returnCode: 123)
```

那么疑问又来了：如果说我就是希望没有member了就代表数据采集完了，结果你给我判断一下returnCode还有值，你还一直往下翻页插数据，那不对啊。  为此我们引入了**主键字段**这个概念，主键字段是用来标识 对于你来说，你认为什么样的数据才是有效数据的字段，比如说作为一个member，一定要有一个ID，不然这个数据就不是我想要的一条数据。我们扩展一下上面的json来看下怎么解释

```json
{
    "result": "success",
    "member": [
        {
            "name": "abc",
            "ID": "123",
            "school": "ttt"
        },
        {
            "name": "def",
            "school": "ttt"
        },
        {
            "ID": "456",
            "school": "ttt"
        }
    ]
    "returnCode": 123
}
```

看一下这样的数据会被解析成什么样子：

```shell
(result: success, member.name: abc ,member.ID: 123 ,member.school: ttt,returnCode: 123)
(result: success, member.name: def ,member.ID: null, member.school: ttt,returnCode: 123)
(result: success, member.ID: 123 ,member.name: null, member.school: ttt,returnCode: 123)
```

好，我们看一下，因为我们确定了ID是主键字段，因此我在遇到第二条数据的时候，尽管他有名字和学校，但是ID是空的，那么我将会抛弃这条数据。 那如果我认为一个member的标准是，有ID或者name都算一条合法数据，那我可以选择这两个字段为主键字段，此时三条数据都会被接收。
