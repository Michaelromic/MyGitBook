2、Python中的元类(metaclass)
* 类也是对象：
```
>>> print ObjectCreator     # 你可以打印一个类，因为它其实也是一个对象
<class '__main__.ObjectCreator'>
>>> def echo(o):
…       print o
…
>>> echo(ObjectCreator)                 # 你可以将类做为参数传给函数
<class '__main__.ObjectCreator'>
>>> print hasattr(ObjectCreator, 'new_attribute')
Fasle
>>> ObjectCreator.new_attribute = 'foo' #  你可以为类增加属性
>>> print hasattr(ObjectCreator, 'new_attribute')
True
>>> print ObjectCreator.new_attribute
foo
>>> ObjectCreatorMirror = ObjectCreator # 你可以将类赋值给一个变量
>>> print ObjectCreatorMirror()
<__main__.ObjectCreator object at 0x8997b4c>
```

* 动态地创建类：
a、可以在函数中创建类，使用class关键字即可。
b、type可以接受一个类的描述作为参数，然后返回一个类。
type(类名, 父类的元组（针对继承的情况，可以为空），包含属性的字典（名称和值）)
例如：
```
>>> MyShinyClass = type('MyShinyClass', (), {})  # 返回一个类对象
>>> print MyShinyClass
<class '__main__.MyShinyClass'>
>>> print MyShinyClass()  #  创建一个该类的实例
<__main__.MyShinyClass object at 0x8997cec>
```
用字典定义属性 + 继承 + 增加方法
```
>>> Foo = type('Foo', (), {'bar':True})
>>> FooChild = type('FooChild', (Foo,),{})
>>> print FooChild
<class '__main__.FooChild'>
>>> print FooChild.bar   # bar属性是由Foo继承而来
True
>>> def echo_bar(self):
…       print self.bar
…
>>> FooChild = type('FooChild', (Foo,), {'echo_bar': echo_bar})
>>> hasattr(Foo, 'echo_bar')
False
>>> hasattr(FooChild, 'echo_bar')
True
>>> my_foo = FooChild()
>>> my_foo.echo_bar()
True
```

##### 总结：在Python中，类也是对象，你可以动态的创建类。这就是当你使用关键字class时Python在幕后做的事情，而这就是通过元类来实现的。
元类就是用来创建这些类（对象）的，元类就是类的类。
MyClass = type('MyClass', (), {})
type就是Python在背后用来创建所有类的元类。
任何一个__class__的__class__属性是type：
```
>>> class Bar(object): pass
>>> b = Bar()
>>> b.__class__
<class '__main__.Bar'>
>>> b.__class__.__class__
<type 'type'>
```
+ type就是Python的内建元类，当然了，你也可以创建自己的元类。
```
class Foo(object):
	__metaclass__ = something…
[…]
```
你首先写下class Foo(object)，但是类对象Foo还没有在内存中创建。Python会在类的定义中寻找__metaclass__属性，如果找到了，Python就会用它来创建类Foo。
如果Python没有找到__metaclass__，它会继续在Bar（父类）中寻找__metaclass__属性，并尝试做和前面同样的操作。如果Python在任何父类中都找不到__metaclass__，它就会在模块层次中去寻找__metaclass__，并尝试做同样的操作。如果还是找不到__metaclass__,Python就会用内置的type来创建这个类对象。

#### 元类的作用：
1、拦截类的创建
2、修改类
3、返回修改之后的类

例如：构造一个元类，使所有属性都改成大写形式
```
class UpperAttrMetaclass(type):
    def __new__(cls, name, bases, dct):
        attrs = ((name, value) for name, value in dct.items() if not name.startswith('__'))
        uppercase_attr = dict((name.upper(), value) for name, value in attrs)
        uppercase_attr['say_' + name] = lambda self, value, saying=name: print(saying + ',' + value + '!')
        return super(UpperAttrMetaclass, cls).__new__(cls, name, bases, uppercase_attr)

# python3 中取消了 __metaclass__属性，只能在定义类的时候声明metaclass参数来创建元类
class MyClass(object, metaclass=UpperAttrMetaclass):
    bar = 1

print(hasattr(MyClass, 'bar'))
# 输出: False
print(hasattr(MyClass, 'BAR'))
# 输出: True

f = MyClass()
print(f.BAR)
# 输出：1
f.say_MyClass('Hello')
# 输出：MyClass,Hello!
```
