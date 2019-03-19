# [Django去操作已经存在的数据库](https://www.cnblogs.com/smiling-crying/p/9237452.html)

> ## 你有没有遇到过这种情况？

**数据库，各种表结构已经创建好了，甚至连数据都有了，此时，我要用Django管理这个数据库，ORM映射怎么办？？？**

**Django是最适合所谓的green-field开发，即从头开始一个新的项目**

但是呢，Django也支持和以前遗留的数据库和应用相结合的。

Django的数据库层从Python代码生成SQL schemas。但是对于遗留的数据库，你已经用于SQL schemas，这种情况下你需要为你已经存在的数据库表写模型（为了使用数据库的API），幸运的是，Django自带有通过阅读你的数据库表规划来生成模型代码的辅助工具 manage.py inspectdb

###  1.Django默认使用的是sqllit数据库？如何使用MySQL数据库？

 <a title="复制代码">![复制代码](https://common.cnblogs.com/images/copycode.gif)</a>

#修改setting.py文件
```
DATABASE = {
    'default':{
        'ENGINE':'django.db.backends.mysql',
        'NAME':'数据库名',
        'HOST':'数据库地址',
        'PORT':端口,
        'USER':'用户名',
        'PASSWORD':'密码',
    }
}
```

#由于Django内部链接MySQL数据库的时候默认的是使用MySQLdb的
#但是Python3中没有这个模块
#所以我们要去修改他的project同名文件夹下的__init__文件

import pymysql
pymysql.install_as_MySQLdb()
 <a title="复制代码">![复制代码](https://common.cnblogs.com/images/copycode.gif)</a>

 **然后呢，我们就需要根据数据库去自动生成新的models文件**

 python manage.py inspectdb    #简单可以看一下自动映射成的models中的内容

**导出并且去代替models.py**

 python manage.py inspectdb > models.py

这样你就会发现在manage.py的同级目录下生成了一个models.py文件

使用这个models.py文件覆盖app中的models文件。

如果完成了以上的操作，生成的是一个**不可修改/删除的models**，修改meta class中的**managed = True**则可以去告诉django可以对数据库进行操作

![](https://images2018.cnblogs.com/blog/1294578/201807/1294578-20180703082631513-1019513692.png)

**此时，我们再去使models.py和数据库进行同步**

 python manage.py migrate

### 这个时候就已经大功告成了！

 然我们来验证一下：

 python manage.py shell

#一些查询语句

### 嗯.....没毛病。