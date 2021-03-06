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
> 如果这是你第一次使用 Django 的话，你需要一些初始化设置。也就是说，你需要用一些自动生成的代码配置一个 Django project —— 即一个 Django 项目实例需要的设置项集合，包括数据库配置、Django 配置和应用程序配置。

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

---
### 创建应用
我们在的 manage.py 同级目录下创建投票应用。这样它就可以作为顶级模块导入，而不是 mysite 的子模块。
`py manage.py startapp polls`
创建了如下目录：
```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```
##### 项目 VS 应用
> 项目和应用有啥区别？应用是一个专门做某件事的网络应用程序——比如博客系统，或者公共记录的数据库，或者简单的投票程序。项目则是一个网站使用的配置和应用的集合。项目可以包含很多个应用。应用可以被很多个项目使用。

##### 编写第一个视图
打开 polls/views.py
```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
> 这是 Django 中最简单的视图。如果想看见效果，我们需要将一个 URL 映射到它——这就是我们需要 URLconf 的原因了。
为了创建 URLconf，请在 polls 目录里新建一个 urls.py 文件。

打开 polls/urls.py
```
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```
下一步是要在根 URLconf 文件中指定我们创建的 polls.urls 模块。在 mysite/urls.py 文件的 urlpatterns 列表里插入一个 include()
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```
> 函数 include() 允许引用其它 URLconfs。每当 Django 遇到 :func：~django.urls.include 时，它会截断与此项匹配的 URL 的部分，并将剩余的字符串发送到 URLconf 以供进一步处理。
我们设计 include() 的理念是使其可以即插即用。因为投票应用有它自己的 URLconf( polls/urls.py )，他们能够被放在 "/polls/" ， "/fun_polls/" ，"/content/polls/"，或者其他任何路径下，这个应用都能够正常工作。
当包括其它 URL 模式时你应该总是使用 include() ， admin.site.urls 是唯一例外。

现在把 index 视图添加进了 URLconf。可以验证是否正常工作，运行下面的命令:
`py manage.py runserver`
访问 http://localhost:8000/polls/

> 函数 path() 具有四个参数，两个必须参数：route 和 view，两个可选参数：kwargs 和 name。
> * route 是一个匹配 URL 的准则（类似正则表达式）。当 Django 响应一个请求时，它会从 urlpatterns 的第一项开始，按顺序依次匹配列表中的项，直到找到匹配的项。
这些准则不会匹配 GET 和 POST 参数或域名。例如，URLconf 在处理请求 https://www.example.com/myapp/ 时，它会尝试匹配 myapp/ 。处理请求 https://www.example.com/myapp/?page=3 时，也只会尝试匹配 myapp/。
> * view：
当 Django 找到了一个匹配的准则，就会调用这个特定的视图函数，并传入一个 HttpRequest 对象作为第一个参数，被“捕获”的参数以关键字参数的形式传入。稍后，我们会给出一个例子。
> * kwargs：
任意个关键字参数可以作为一个字典传递给目标视图函数
> * name：
为你的 URL 取名能使你在 Django 的任意地方唯一地引用它，尤其是在模板中。这个有用的特性允许你只改一个文件就能全局地修改某个 URL 模式。

---
### 使用mysql
`pip install pymysql`
在Django的settings.py文件中设置如下：
```
import pymysql         # 一定要添加这两行！           
pymysql.install_as_MySQLdb()
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',   # 数据库引擎
        'NAME': 'mysite',  # 数据库名，先前创建的
        'USER': 'root',     # 用户名，可以自己创建用户
        'PASSWORD': '****',  # 密码
        'HOST': '192.168.1.121',  # mysql服务所在的主机ip
        'PORT': '3306',         # mysql服务端口
    }
}
```

