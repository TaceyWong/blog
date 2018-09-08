+++
title = "Python中通过lambda抛异常的奇技淫巧"
date = 2018-07-04T19:28:11+08:00
toc = true
tags = ["Python"]
categories = ["编程语言"]
banner = "img/avatar.png"
author = "Tacey Wong"
+++

假设我们需要一个函数什么事都不干，只是抛出异常（在某些系统中有些handler就是干这事的），我们可以很直观的写出下面的代码：
```python
def func():
    raise Exception("this is a exception")
```
就这么一个简单的功能，我们更希望用lambda实现，自然就写下了下面的代码：
```python
lambda :raise  Exception("this is a exception")
```
但遗憾的是这样是不行的~~~会出现`SyntaxError: invalid syntax`的错误。具体原因可以看[Python Lambda](https://docs.python.org/3/reference/expressions.html#lambda)

下面搜集实践了几种可用的奇技淫巧：

#### 方法一

```python
func = lambda: (_ for _ in ()).throw(Exception('this is an exception'))
```

#### 方法二

如果不在乎异常信息是什么：
```python
func = lambda: 1/0
```
不难理解，这个函数会抛出`ZeroDivisionError`。这种方法其实代表了一类，比如也可以写成：
```python
func = lambda : [][0]
```
这类实现就是在lambda后面写一定会抛出异常的表达式

#### 方法三

一种非常阴霸的方式,只适合python3.x
```python
func = lambda : exec('raise(Exception("this is an exception"))')
```

#### 方法四：
尚未看懂的
```python
# python2.x
type(lambda:0)(type((lambda:0).func_code)(
  1,1,1,67,'|\0\0\202\1\0',(),(),('x',),'','',1,''),{}
)(Exception())
```
或
```python
# python3.x
type(lambda: 0)(type((lambda: 0).__code__)(
    1,0,1,1,67,b'|\0\202\1\0',(),(),('x',),'','',1,b''),{}
)(Exception())
```


---

最后忠告:***玩玩可以，慎用!***