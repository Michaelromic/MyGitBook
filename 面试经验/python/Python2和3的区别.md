# Python2和3的区别
https://chenqx.github.io/2014/11/10/Key-differences-between-Python-2-7-x-and-Python-3-x/

* ### __future__模块
Python 3.x 介绍的 一些Python 2 不兼容的关键字和特性可以通过在 Python 2 的内置 __future__ 模块导入。
例如，如果我想要 在Python 2 中表现 Python 3.x 中的整除，我们可以通过如下导入：
`from __future__ import division`
可以导入的模块有：
nested_scopes、generators、division、absolute_import、with_statement、print_function、unicode_literals

* ### print函数
Python 2 的 print 声明已经被 Python 3中的 print() 函数取代了。
print 在 Python 2 中是一个声明，而不是一个函数调用。

* ### 整除
当2个**整数**相除时，/ 在 python2 和 python3 中有区别： 
python2是整除，结果是int
python3不是整除，结果是float。

```
# Python2
print '3 / 2 =', 3 / 2  # 1
print '3 // 2 =', 3 // 2  # 1
print '3 / 2.0 =', 3 / 2.0  # 1.5
print '3 // 2.0 =', 3 // 2.0  # 1.0

# Python3
print('4 / 2 =', 4 / 2)  # 2.0
print('3 / 2 =', 3 / 2)  # 1.5
print('3 // 2 =', 3 // 2)  # 1
print('3 / 2.0 =', 3 / 2.0)  # 1.5
print('3 // 2.0 =', 3 // 2.0)  # 1.0
```

* ### Unicode

python2 默认编码是系统默认编码(如linux是utf-8，windows是gbk)，并且有单独的 unicode类型。
python3 默认编码就是unicode，unicode编码会被直接认为是字符串类型。

Python3有两种表示字符序列的类型：bytes和str。前者的实例包含原始的8位值，后者的实例包含Unicode字符。
Python2也有两种表示字符序列的类型，分别叫做str和Unicode。与Python3不同的是，str实例包含原始的8位值；而unicode的实例，则包含Unicode字符。
编写Python程序的时候，一定要把编码和解码操作放在界面最外围来做。程序的核心部分应该使用Unicode字符类型（也就是Python3中的str、Python2中的unicode）。

```
# python 3
print(type('\u03BC\u0394'))  # <class 'str'>
print(type(b' bytes for storing data'))  # <class 'bytes'>，这个在python2中是str类型
```

* ### xrange模块

在 Python 2 中 xrange() 创建迭代对象的用法是非常流行的。比如： for 循环或者是列表/集合/字典推导式。
因为直接创建一个迭代器，不用生成list，所以比range()执行更快。

python3中去除了range()，直接将xrange()重命名为range()。
但实际上 Python 3 的 range() 比 Python 2 的 xrange() 运行慢。Python 3 倾向于比 Python 2 运行的慢一点。
```
# python2
x = xrange(1,3)
print type(x)  # <type 'xrange'>

x = range(1,3)
print type(x)  # <type 'list'>


# python3
x = range(1,3)
print(type(x))  # <class 'range'>
```

* ### Python3中的range对象的__contains__方法
Python 3 中 range 有一个新的 __contains__ 方法，__contains__ 方法可以加速 “查找” 在 Python 3.x 中显著的整数和布尔类型。

* ### Raising exceptions
Python 2 接受新旧两种语法标记，在 Python 3 中如果我不用小括号把异常参数括起来就会阻塞（并且反过来引发一个语法异常）。

```
# python2，以下两种写法都支持
raise IOError, "file error"
raise IOError("file error")

# python3，只支持一种写法
raise IOError("file error")
```

* ### Handling exceptions
在 Python 3 中处理异常也轻微的改变了，在 Python 3 中我们现在使用 as 作为关键词。

```
# python2
try:
    let_us_cause_a_NameError
except NameError, err:
    print err, '--> our error message'

# python3
try:
    let_us_cause_a_NameError
except NameError as err:
    print(err, '--> our error message')

```

* ### next()函数 and .next()方法
python3 删除了.next()方法
```
# python3
my_generator = (letter for letter in 'abcdefg')
next(my_generator)  # a
```

* ### For循环变量和全局命名空间泄漏
在 Python 3.x 中 for 循环变量不会再导致命名空间泄漏。
```
# python 2 中使用列表推导会导致循环控制变量泄露进周围的作用范围域
print 'Python', python_version()
i = 1
print 'before: i =', i  # before: i = 1
print 'comprehension: ', [i for i in range(5)]  # comprehension: [0, 1, 2, 3, 4]
print 'after: i =', i  # after: i = 4
```

    列表推导不再支持 [... for var in item1, item2, ...] 这样的语法。使用 [... for var in (item1, item2, ...)] 代替
```
# python 3
print([i for i in range(5)])  # [0, 1, 2, 3, 4]
print([i for i in [1,2,3]])  # [1, 2, 3]
print([i for i in (1,2,3)])  # [1, 2, 3]
```

* ### 比较不可排序类型
在 Python 3 中的另外一个变化就是当对不可排序类型做比较的时候，会抛出一个类型错误。
而python 2中比较 [1,2] 和 'foo'的大小不会报错。

* ### 通过input()解析用户的输入
在 Python 3 中已经解决了把用户的输入存储为一个 str 对象的问题。
为了避免在 Python 2 中的读取非字符串类型的危险行为，我们不得不使用 raw_input() 代替。
    在python2中使用input()读取数字，会存储为int类型，如果使用raw_input()读取数字，会存储为str类型。

* ### 返回可迭代对象，而不是列表
假如只遍历一次，我们用可迭代对象可以节约内存。
如果需遍历多次，可以用list()函数把迭代对象转换成一个列表。
```
# python2
print range(3)  # [0, 1, 2]
print type(range(3))  # < type ‘list’>

# python3
print(range(3))  # range(0, 3)
print(type(range(3)))  # < class ‘range’>
print(list(range(3)))  # [0, 1, 2]
```

在 Python 3 中一些经常使用到的不再返回列表的函数和方法：
zip()
map()
filter()
dictionary’s .keys() method
dictionary’s .values() method
dictionary’s .items() method