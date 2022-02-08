### Python 的return 和finally

跟java一样，finally一定会执行，python中会以最后一次return的结果为准，因此以finally中的返回值为准

```python
def some_func():
    try:
        return 'from_try'
    finally:
        return 'from_finally'
```



### 为啥会这样？

为什么我执行了一个for循环之后，这个字典就有了值

```python
if __name__ == '__main__':
    some_string = "wtf"
    some_dict = {}
    for i, some_dict[i] in enumerate(some_string):
        pass
    print(some_dict)

{0: 'w', 1: 't', 2: 'f'}
```

因为在python中，for语句的定义是这样的：

```bash
for_stmt: 'for' exprlist 'in' testlist ':' suite ['else' ':' suite]
```

也就是说python的for循环其实是一个迭代器，for循环中不去做退出判断，像上面，`enumerate(some_string)` 每次迭代都会返回一个tuple {1:'w'}，此时我分别用i 和 some_dict[i]去接收。



### 为什么用生成器表达式得不到想要的结果

```python
array = [1, 8, 15]
g = (x for x in array if array.count(x) > 0)
array = [2, 8, 22]

print(list(g))
[8]
# 不应该是 [1, 8, 15]吗？
```

原因是 在**生成器表达式**中，in 子句在声明时执行（即在生成可执行语句的时候我就已经把in语句中的 array的顺序记下来），而条件子句则是在运行时执行。 

可以这样理解，当我们使用一个生成器表达式的时候，其实是保存了一个指针指向原集合 array的地址，而不会随着array指向了另外一个内存地址而改变这个指针。但是条件语句会使用array对象最新的地址。

如果我们修改一下代码变成这样：

```python
array = [1, 8, 15]
g = (x for x in array if array.count(x) > 0)
array[:] = [2, 8, 22]
```

由于使用了切片赋值，所以会直接修改原来的内存地址的内容，g指向的也是[2, 8, 22]，所以会输出 [2, 8, 22]





































