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

---
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
这些目录和文件的用处是：
* 最外层的:file: mysite/ 根目录只是你项目的容器， Django 不关心它的名字，你可以将它重命名为任何你喜欢的名字。
* manage.py: 一个让你用各种方式管理 Django 项目的命令行工具。你可以阅读 django-admin and manage.py 获取所有 manage.py 的细节。
* 里面一层的 mysite/ 目录包含你的项目，它是一个纯 Python 包。它的名字就是当你引用它内部任何东西时需要用到的 Python 包名。 (比如 mysite.urls).
* mysite/__init__.py：一个空文件，告诉 Python 这个目录应该被认为是一个 Python 包。如果你是 Python 初学者，阅读官方文档中的 更多关于包的知识。
* mysite/settings.py：Django 项目的配置文件。如果你想知道这个文件是如何工作的，请查看 Django settings 了解细节。
* mysite/urls.py：Django 项目的 URL 声明，就像你网站的“目录”。阅读 URL调度器 文档来获取更多关于 URL 的内容。
* mysite/wsgi.py：作为你的项目的运行在 WSGI 兼容的Web服务器上的入口。阅读 如何使用 WSGI 进行部署 了解更多细节。

##### 用于开发的简易服务器
`python manage.py runserver 8001(默认端口是8000)`
输出如下：
```
Performing system checks...

System check identified no issues (0 silenced).

You have unapplied migrations; your app may not work properly until they are applied.
Run 'python manage.py migrate' to apply them.

二月 28, 2019 - 15:50:53
Django version 2.1, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```





