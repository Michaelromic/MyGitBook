### 安装Django
`pip install Django`

###### 验证
若要验证 Django 是否能被 Python 识别，可以在 shell 中输入 python。 然后在 Python 提示符下，尝试导入 Django：
```
>>> import django
>>> print(django.get_version())
2.1
```
或者
`python -m django --version`

### 创建项目
如果这是你第一次使用 Django 的话，你需要一些初始化设置。也就是说，你需要用一些自动生成的代码配置一个 Django project —— 即一个 Django 项目实例需要的设置项集合，包括数据库配置、Django 配置和应用程序配置。

打开命令行，cd 到一个你想放置你代码的目录，然后运行以下命令：
`django-admin startproject mysite`
创建了如下目录：
```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```