# AOP
#### 这种在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程。
> 面向对象的特点是继承、多态和封装。而封装就要求将功能分散到不同的对象中去，这在软件设计中往往称为职责分配。实际上也就是说，让不同的类设计不同的方法。这样代码就分散到一个个的类中去了。这样做的好处是降低了代码的复杂程度，使类可重用。

> 但是人们也发现，在分散代码的同时，也增加了代码的重复性。什么意思呢？比如说，我们在两个类中，可能都需要在每个方法中做日志。按面向对象的设计方法，我们就必须在两个类的方法中都加入日志的内容。也许他们是完全相同的，但就是因为面向对象的设计让类与类之间无法联系，而不能将这些重复的代码统一起来。

> 也许有人会说，那好办啊，我们可以将这段代码写在一个独立的类独立的方法里，然后再在这两个类中调用。但是，这样一来，这两个类跟我们上面提到的独立的类就有耦合了，它的改变会影响这两个类。那么，有没有什么办法，能让我们在需要的时候，随意地加入代码呢？这种在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程。

> 一般而言，我们管切入到指定类指定方法的代码片段称为切面，而切入到哪些类、哪些方法则叫切入点。有了AOP，我们就可以把几个类共有的代码，抽取到一个切片中，等到需要时再切入对象中去，从而改变其原有的行为。

> 这样看来，AOP其实只是OOP的补充而已。OOP从横向上区分出一个个的类来，而AOP则从纵向上向对象中加入特定的代码。有了AOP，OOP变得立体了。如果加上时间维度，AOP使OOP由原来的二维变为三维了，由平面变成立体了。从技术上来说，AOP基本上是通过代理机制实现的。      

> AOP在编程历史上可以说是里程碑式的，对OOP编程是一种十分有益的补充。


---
# 装饰器
http://kissg.me/2016/07/16/translation-about-python-decorator/
https://foofish.net/python-decorator.html

装饰器是一个很著名的设计模式，经常被用于有切面需求的场景，较为经典的有插入日志、性能测试、事务处理等。装饰器是解决这类问题的绝佳设计，有了装饰器，我们就可以抽离出大量函数中与函数功能本身无关的雷同代码并继续重用。
装饰器的作用就是为已经存在的对象添加额外的功能。

#### 装饰器就是”包装纸(wrapper)”, 也就是说, 装饰器允许你在被装饰的函数前后执行代码, 而不对函数本身做任何修改。

* ##### 装饰器示例

```
def use_logging(func):

    def wrapper():
        logging.warn("%s is running" % func.__name__)
        return func()
    return wrapper

def foo():
    print("i am foo")

a = use_logging(foo)
a()
```

* ##### 使用@语法糖

```
def use_logging(func):

    def wrapper():
        logging.warn("%s is running" % func.__name__)
        return func()
    return wrapper

@use_logging
def foo():
    print("i am foo")

foo()
```
use_logging 就是一个装饰器，它一个普通的函数，它把执行真正业务逻辑的函数 func 包裹在其中，看起来像 foo 被 use_logging 装饰了一样，use_logging 返回的也是一个函数，这个函数的名字叫 wrapper。在这个例子中，函数进入和退出时 ，被称为一个横切面，这种编程方式被称为面向切面的编程。

* ##### 装饰器可以带参数：
这里wrapper接收的参数就是foo函数的参数，可以任意自定义，这里用*args, **kwargs定义了一个通用的装饰器。
#### 这里是一个装饰器的装饰器。一个能接收任意参数的装饰器代码片段(decorator)。 然后, 我们用另一个函数(use_logging)来创建我们的装饰器, 以接收参数。
@use_logging(level="warn")等价于@decorator (当然还是会优先执行use_logging的内容)

```
def use_logging(level):
    def decorator(func):
        def wrapper(*args, **kwargs):
            if level == "warn":
                logging.warn("%s is running" % func.__name__)
            elif level == "info":
                logging.info("%s is running" % func.__name__)
            return func(*args)
        return wrapper

    return decorator

@use_logging(level="warn")
def foo(name='foo'):
    print("i am %s" % name)

foo()
```

* ##### 类装饰器：装饰器不仅可以是函数，还可以是类，相比函数装饰器，类装饰器具有灵活度大、高内聚、封装性等优点。使用类装饰器主要依靠类的__call__方法，当使用 @ 形式将装饰器附加到函数上时，就会调用此方法。

```
class Foo(object):
    def __init__(self, func):
        self._func = func

    def __call__(self):
        print ('class decorator runing')
        self._func()
        print ('class decorator ending')

@Foo
def bar():
    print ('bar')

bar()
'''输出
class decorator runing
bar
class decorator ending
'''
```

* ##### 引入functools模块, 该模块包含的funtools.wraps()函数, 可以将被装饰函数名称, 模块, 文档字符串(docstring)拷贝到它的包装纸中。
wraps本身也是一个装饰器。
如果没有wraps装饰器，这里会输出 'with_logging' 和 None。

```
from functools import wraps
def logged(func):
    @wraps(func)
    def with_logging(*args, **kwargs):
        print func.__name__      # 输出 'f'
        print func.__doc__       # 输出 'does some math'
        return func(*args, **kwargs)
    return with_logging

@logged
def f(x):
   """does some math"""
   return x + x * x

```

* ##### 装饰器顺序：一个函数还可以同时定义多个装饰器

```
@a
@b
@c
def f ():
    pass

# 它的执行顺序是从里到外，最先调用最里层的装饰器，最后调用最外层的装饰器，它等效于
f = a(b(c(f)))
```

* ##### 如何使用装饰器
    + 经典的用法是, 拓展一个外部库函数(你无法修改它)的功能, 或者用于调试(因为它是临时的, 你并不想进行修改).
    + 当然, 装饰器的好处是, 对于几乎所有东西, 在不重写的情况, 马上可用.