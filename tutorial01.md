# 编写你的第一个Django应用程序，第1部分

让我们通过实例来学习。

在本教程中，我们将引导您完成一个基础的问卷调查应用程序

它将由两部分组成：

- 一个让人们查看民意调查并投票的公共网站。
- 一个管理网站，可让您添加，更改和删除民意调查。
我们假设你已经安装Django。
您可以通过在shell提示符下运行以下命令（由$前缀指示）来告诉Django已经安装以及哪个版本：
```
$ python -m django --version
```
如果安装了Django，你应该看到你的安装版本。 如果不是，则会显示“No module named django”错误。

## 创建一个项目
如果这是你第一次使用Django，你需要完成一些初始化设置。 你需要自己用代码来创建一个Django项目 ——一个Django框架开发的网站，创建项目后我们需要的配置的东西，包括数据库的配置、针对Django的配置选项和app的配置选项。

在命令行（终端）中，cd（例如cd code)到你想要用来保存代码的目录，然后运行如下命令：
```
$ django-admin startproject mysite
```
这将在您当前的目录中创建一个mysite目录。 如果它不能正常工作，请查看[运行django-admin遇到的问题](https://docs.djangoproject.com/en/2.0/faq/troubleshooting/#troubleshooting-django-admin)。

> ### 注意

> 您需要避免在内置Python或Django组件之后命名项目。特别是，这意味着你应该避免使用像 django（这将与Django本身冲突）或test（与内置Python包冲突）的名称。

---

> ### 代码应该存在哪里？
> 如果你曾经学过普通的旧式的PHP（没有使用过现代的框架），你可能习惯于将代码放在Web服务器的文档根目录下（例如/var/www）。 但是对于Django，你不该这么做。 将Python代码放在你的Web服务器的根目录不是个好主意，因为它可能会有让别人在网上看到你的代码的风险。 这样不安全

> 将你的代码放置在Web服务器根目录以外的地方，例如/home/mycode。

我们来看一下startproject创建的内容：
```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```

这些文件是：

- 外层的mysite/根目录仅仅是项目的一个容器。 它的命名对Django无关紧要；你可以把它重新命名为任何你喜欢的名字。

- manage.py：一个命令行工具，可以使你用多种方式对Django项目进行交互。 你可以在[django-admin和manage.py](https://docs.djangoproject.com/en/2.0/ref/django-admin/)中读到关于manage.py的所有细节。

- 内层的mysite/目录是你的项目的真正的Python包。 它是你导入任何东西时将需要使用的Python包的名字（例如 mysite.urls）。

- mysite/__init__.py：一个空文件，它告诉Python这个目录应该被看做一个Python包。 （如果你是一个Python初学者，关于[包的更多内容](https://docs.python.org/3/tutorial/modules.html#tut-packages)请阅读Python的官方文档）。

- mysite/settings.py：该Django 项目的设置/配置。 [Django 设置](https://docs.djangoproject.com/en/2.0/topics/settings/) 将告诉你这些设置如何工作。

- mysite/urls.py：该Django项目的URL声明；你的Django站点的“目录”。 你可以在[URL 转发器](https://docs.djangoproject.com/en/2.0/topics/http/urls/) 中阅读到更多关于URL的内容。

- mysite/wsgi.py：用于你的项目的与WSGI兼容的Web服务器入口。 更多细节请参见[如何利用WSGI](https://docs.djangoproject.com/en/2.0/howto/deployment/wsgi/)进行[部署](https://docs.djangoproject.com/en/2.0/howto/deployment/wsgi/)。


## 开发服务器

让我们验证一下Django项目是否工作。 如果还没有，请转到外部mysite目录，然后运行以下命令：
```
$ python manage.py runserver
```

您将在命令行上看到以下输出：
```
Performing system checks...

System check identified no issues (0 silenced).

You have unapplied migrations; your app may not work properly until they are applied.
Run 'python manage.py migrate' to apply them.

January 17, 2018 - 15:50:53
Django version 2.0, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

> ###  注意
> 现在忽略关于未应用数据库迁移的警告；我们很快就会处理数据库。

这说明你已经启动了Django开发服务器，一个用纯Python写的轻量级Web服务器。 我们在Django中内置了它，这样你就可以在不配置用于生产环境的服务器 —— 例如Apache的情况下快速开发出产品，直到你准备好上线。

现在需要注意的是：不要在任何生产环境使用这个服务器。 它仅用于开发时使用。 （我们的重点是编写Web框架，非Web服务器。）

既然服务器已经运行，请用你的浏览器访问 http://127.0.0.1:8000/  在淡蓝色背景下，你将看到一个“Welcome to Django”的页面。 它成功运行了！

> ### 改变端口
> 默认情况下，runserver命令在内部IP的8000端口启动开发服务器。

> 如果要更改服务器的端口，请将其作为命令行参数传递。 
> 例如，这个命令在端口8080上启动服务器：

```
$ python manage.py runserver 8080
```
> 如果要更改服务器的IP，请将其与端口一起传递。  
> 例如，要收听所有可用的公共IP（如果您正在运行Vagrant或想要在网络上的其他计算机上显示您的工作，这是非常有用的），请使用：
```
$ python manage.py runserver 0:8000
```
> 0是0.0.0.0的快捷方式。 完整的开发服务器文档可以在runserver参考中找到。

---

> runserver的自动重载
> 开发服务器根据需要自动为每个请求重新加载Python代码。您无需重新启动服务器以使代码更改生效。但是，某些操作（比如添加文件）不会触发重新加载，因此在这种情况下您必须重新启动服务器。

## 创建投票应用程序
既然你的环境已经建立起来了，你就开始工作了。

您在Django中编写的每个应用程序都包含遵循特定约定的Python包。Django带有一个实用程序，可以自动生成应用程序的基本目录结构，因此您只需要专心于编写代码而不是创建目录。

> ### 项目与应用程序

> 项目和应用程序有什么区别？应用程序是一种Web应用程序，它可以执行某些操作，例如Weblog系统，公共记录数据库或简单的轮询应用程序。项目是特定网站的配置和应用程序的集合。项目可以包含多个应用程序。一个应用程序可以在多个项目中。

您的应用程序可以存放在[Python path](https://docs.python.org/3/tutorial/modules.html#tut-searchpath)的任何目录。在本教程中，我们将在您的manage.py 文件旁边创建我们的投票应用程序，以便它可以作为自己的顶级模块导入，而不是导入为子模块mysite。

要创建您的应用程序，请确保您与manage.py位于同一个目录中，然后输入以下命令：
```
$ python manage.py startapp polls
```

这将创建一个目录polls，其目录结构如下所示：
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

### 编写你的第一个视图
我们来写第一个视图。打开文件polls/views.py 并将下面的Python代码放入其中：
> polls/views.py
```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

这是Django中最简单的视图。要调用视图，我们需要将它映射到一个URL - 为此我们需要一个URLconf。
要在polls目录中创建URLconf，请创建一个名为的文件urls.py。你的app目录现在应该如下所示：
```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    urls.py
    views.py
```

在polls/urls.py文件中包含以下代码：
> polls/urls.py
```
from django.urls import include, path
from django.contrib import admin

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

[include()](https://docs.djangoproject.com/en/2.0/ref/urls/#django.urls.include)函数允许引用其他URLconf。每当Django遇到时include()，它会截断与该点匹配的URL的任何部分，并将剩余的字符串发送到包含的URLconf以供进一步处理。

include()背后的想法是为了方便即插即用的URL。由于polls是在他们自己的URLconf（polls/urls.py）中，他们可以放在“/ polls /”下，或者放在“/ fun_polls /”下，或者放在“/ content / polls /”下，或者任何其他路径下,他将依然正常工作。


> ### 何时使用include()

> 当包括其他URL模式时你应该使用include()。唯一的例外就是admin.site.urls。

你现在已经将一个index视图定义到了URLconf。让我们验证它是否有效，运行以下命令：
```
$ python manage.py runserver
```

浏览器访问http://localhost:8000/polls/ 你将看到一行文字 “Hello, world. You’re at the polls index.”, 这是你的index视图里面定义的内容

path()函数传递四个参数，其中两个是必需的： route和view，以及两个可选的：kwargs，和name。在这一点上，值得回顾一下这些参数是什么。

### path()参数：route 
route是一个包含URL模式的字符串。在处理请求时，Django从第一个模式开始urlpatterns并在列表中向下，将所请求的URL与每个模式进行比较，直到找到匹配的模式。

模式不搜索GET和POST参数或域名。例如，在请求中https://www.example.com/myapp/ URLconf将查找 myapp/。在请求中https://www.example.com/myapp/?page=3 URLconf也会查找myapp/。


### path() 参数: view

When Django finds a matching pattern, it calls the specified view function with an HttpRequest object as the first argument and any “captured” values from the route as keyword arguments. We’ll give an example of this in a bit.
当Django找到匹配的模式时，它会以HttpRequest对象作为第一个参数和路由中的任何“捕获”值作为关键字参数来调用指定的视图函数。 我们将举一个例子。

### path() 参数: kwargs

任意关键字参数可以在字典中传递给目标视图。 我们不打算在教程中使用Django的这个特性。

### path() 参数: name

命名您的URL可以让您从Django的其他地方明确地引用它，特别是在模板中。 这个强大的功能使您可以对项目的URL模式进行全局更改，而只触摸单个文件。

当您熟悉基本的请求和响应流程时，请阅读本教程的[part 2 of this tutorial](tutorial02.md)开始使用数据库。