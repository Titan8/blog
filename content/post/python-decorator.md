+++
draft = false
date = "2017-01-22T13:20:36+08:00"
title = "Python Decorator"
categories = [
    "backend"
]
tags = ["Python"]

+++
## 什么是装饰器

在coding的时候，你经常会需要计算一个函数的执行时间，最容易想到的方法是在每个函数内部进行计算。

```python
def hello():
    start = time.time()
    print 'Hello'
    print 'Spend ', time.time() - start
```

也有可能是在函数外部计算时间。

<!--more-->

```python
def hello():
    print 'Hello'

start = time.time()
hello()
print 'Spend ', time.time() - start
```

但是，一个hello还好，如果还有hello1、 hello2、hello3，甚至更多的时候你就懵了。这时候，你就会想有没有不需要写这么多重复代码的方法。

要解决这个问题，你首先需要知道在Python中，`function is the first class`。有一种意译是，“函数是一等公民”。你想啊，函数都是一等公民了，那它肯定可以像基本类型那样作为函数参数喽。于是呢，就有了下面这种方法。



```python
def spend(func):
    def _spend():
        start = time.time()
        func()
        print 'Spend ', time.time() - start
    return _spend

def hello():
    print 'Hello'

hello = spend(hello)
hello()
```



这样是不是少写了很多代码，但是呢，还有一个问题，一旦需要计算时间的函数多了，就有可能漏掉一个两个。那么，将时间计算写在函数定义的时候总不会漏了吧。于是乎，Python从2.4版本开始引入一个特殊的语法糖标记`@`。



```python
@spend
def hello():
    print 'Hello'
```



这个spend呢，我们就将其称之为装饰器（Decorator）。

##  装饰器的种类

### 无参数装饰器

```python
def spend(func):
    def _spend(*args, **kwargs):
        start = time.time()
        func(*args, **kwargs)
        print 'Spend ', time.time() - start
    return _spend

@spend
def hello(name):
    print 'Hello', name

# 等同于
def hello(name):
    print 'Hello', name
hello = spend(hello)
```

### 有参数装饰器

```python
def spend(start):
    def _spend(func):
        def __spend(*args, **kwargs):
            func(*args, **kwargs)
            print 'Spend ', time.time() - start
        return __spend
    return _spend

@spend(time.time())
def hello(name):
    print 'Hello', name

# 等同于
def hello(name):
    print 'Hello', name
hello = spend(time.time())(hello)
```

这里有个问题，为什么spend的参数start能在hello中使用？



我们从作用域的角度分析这个问题，举一个另外的例子。



```python
def outer(arg):
    print locals()
    def inner():
        x = arg * arg
        print locals()
        print x
    return inner

>>> f = outer(2)
{'arg': 2}
>>> f()
{'x': 4, 'arg': 2}
4
>>>
```

locals函数的结果是局部命名空间的内容。在outer中，局部命名空间只有一个键arg，而在inner内部也有arg。这就引出了另外一个术语`闭包`。在维基中，闭包的解释是这样的：

> 在计算机科学中，闭包（Closure）是词法闭包（Lexical Closure）的简称，是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。所以，有另一种说法认为闭包是由函数和与其相关的引用环境组合而成的实体。闭包在运行时可以有多个实例，不同的引用环境和相同的函数组合可以产生不同的实例。



> 我的理解是，Python的装饰器就是一种闭包。

这样，我们就能明白为什么spend的参数start能在hello中使用了。



### 类装饰器

```python
def deco(*args, **kwargs):
    def _deco(cls):
        cls.x = 12
        return cls
    return _deco

@deco('hello')
class A(object):
    pass

>>> A.x
12

# 等同于
class A(object):
    pass
A = deco('hello')(A)
```

### 装饰器类

类作为装饰器，分为有参数和无参数。同时，需要装饰的是类方法时，需要用到**get**。

#### 无参数

```python
class Deco(object):
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print 'call Deco'
        self.func(*args, **kwargs)

@Deco
def test():
    print 'call test'

# 等同于
test = Deco(test)
```

#### 有参数

```python
class Deco(object):
    def __init__(self, *args, **kwargs):
        print args, kwargs

    def __call__(self, func):
        def _deco(*args, **kwargs):
            print 'call Deco'
            func(*args, **kwargs)
        return _deco

@Deco('hello')
def test():
    print 'call test'

# 等同于
test = Deco('hello')(func)
```


## 内置装饰器

Python中有三个常用的内置装饰器，`staticmethod`，`classmethod`，`functools.wraps`。

### staticmethod

staticmethod返回一个静态方法。

```python
class Time(object):

    @staticmethod
    def now():
        print time.time()

t = Time()
t.now()
```



### classmethod

classmethod返回一个类方法。



```python
class Time(object):

    @classmethod
    def now():
        print time.time()

Time.now()
```



### functools.wraps



在使用装饰器的时候，返回的新函数的名称并不是原函数的。



```python
def spend(func):
    def _spend(*args, **kwargs):
        start = time.time()
        func(*args, **kwargs)
        print 'Spend ', time.time() - start
    return _spend

@spend
def hello(name):
    print 'Hello', name

>>> print hello.__name__
_spend
```



如果需要新函数的名称和原函数相同，functools.wraps就派上用场了。

```python
def spend(func):
    @functools.wraps(func)
    def _spend(*args, **kwargs):
        start = time.time()
        func(*args, **kwargs)
        print 'Spend ', time.time() - start
    return _spend

@spend
def hello(name):
    print 'Hello', name

>>> print hello.__name__
hello
```



## 总结

假如有新的功能需要实现，但是又不想改变已有的逻辑代码，这时候就需要考虑考虑装饰器了。



参考： [PEP 318 -- Decorators for Functions and Methods](https://www.python.org/dev/peps/pep-0318/#warningwarningwarning)
