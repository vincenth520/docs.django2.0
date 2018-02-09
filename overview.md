## Django 初探

由于Django是在一个快节奏的新闻编辑室环境下开发出来的，因此它被设计成让普通的网站开发工作简单而快 捷。以下简单介绍了如何用 Django 编写一个数据库驱动的Web应用程序。

本文档的目标是给你描述足够的技术细节能让你理解Django是如何工作的，但是它并不表示是一个新手指南或参考目录 – 其实这些我们都有! 当你准备新建一个项目，你可以 从新手指南开始 或者 深入阅读详细的文档.

## 设计你的模型
虽然你可以在不使用数据库的情况下使用Django，但是它带有一个对象关系映射器，用于在Python代码中描述数据库布局。

数据模型语法提供了很多丰富的表示模型的方法 - 到目前为止，它已经解决了数年来数据库模式问题。 这里有一个简单的例子：

> mysite/news/models.py
```python
from django.db import models

class Reporter(models.Model):
    full_name = models.CharField(max_length=70)

    def __str__(self):
        return self.full_name

class Article(models.Model):
    pub_date = models.DateField()
    headline = models.CharField(max_length=200)
    content = models.TextField()
    reporter = models.ForeignKey(Reporter, on_delete=models.CASCADE)

    def __str__(self):
        return self.headline
```
### 安装这个模型
接下来，运行Django命令行实用程序自动创建数据库表：
```
python manage.py migrate
```
这个migrate命令将查看所有可用的模型，并为您的数据库中的表创建任何不存在的表，并且可以选择提供更丰富的[模式控制](https://docs.djangoproject.com/en/2.0/topics/migrations/)。

### 享受免费的API
有了这个，你就有了一个免费的，丰富的Python API来访问你的数据。 API是即时创建的，不需要代码生成：

```shell
# Import the models we created from our "news" app
>>> from news.models import Reporter, Article

# No reporters are in the system yet.
>>> Reporter.objects.all()
<QuerySet []>

# Create a new Reporter.
>>> r = Reporter(full_name='John Smith')

# Save the object into the database. You have to call save() explicitly.
>>> r.save()

# Now it has an ID.
>>> r.id
1

# Now the new reporter is in the database.
>>> Reporter.objects.all()
<QuerySet [<Reporter: John Smith>]>

# Fields are represented as attributes on the Python object.
>>> r.full_name
'John Smith'

# Django provides a rich database lookup API.
>>> Reporter.objects.get(id=1)
<Reporter: John Smith>
>>> Reporter.objects.get(full_name__startswith='John')
<Reporter: John Smith>
>>> Reporter.objects.get(full_name__contains='mith')
<Reporter: John Smith>
>>> Reporter.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Reporter matching query does not exist.

# Create an article.
>>> from datetime import date
>>> a = Article(pub_date=date.today(), headline='Django is cool',
...     content='Yeah.', reporter=r)
>>> a.save()

# Now the article is in the database.
>>> Article.objects.all()
<QuerySet [<Article: Django is cool>]>

# Article objects get API access to related Reporter objects.
>>> r = a.reporter
>>> r.full_name
'John Smith'

# And vice versa: Reporter objects get API access to Article objects.
>>> r.article_set.all()
<QuerySet [<Article: Django is cool>]>

# The API follows relationships as far as you need, performing efficient
# JOINs for you behind the scenes.
# This finds all articles by a reporter whose name starts with "John".
>>> Article.objects.filter(reporter__full_name__startswith='John')
<QuerySet [<Article: Django is cool>]>

# Change an object by altering its attributes and calling save().
>>> r.full_name = 'Billy Goat'
>>> r.save()

# Delete an object with delete().
>>> r.delete()
```

### 一个动态的管理界面：它不只是脚手架 - 它是整个房子
一旦你的模型被定义，Django可以自动创建一个专业的，生产就绪的[管理界面](https://docs.djangoproject.com/en/2.0/ref/contrib/admin/) - 一个网站，让认证的用户添加，更改和删除对象。这与在管理站点注册您的模型一样简单：
> mysite/news/models.py
```python
from django.db import models

class Article(models.Model):
    pub_date = models.DateField()
    headline = models.CharField(max_length=200)
    content = models.TextField()
    reporter = models.ForeignKey(Reporter, on_delete=models.CASCADE)
```
> mysite/news/admin.py
```python
from django.contrib import admin
from . import models
admin.site.register(models.Article)
```
这里的哲学是，你的网站是由工作人员或客户编辑的，或者也许只是你 - 你不想为了管理内容而创建后端接口。

创建Django应用程序的一个典型工作流程是创建模型并使管理站点尽快启动并运行，以便您的员工（或客户端）可以开始填充数据。 然后，开发数据提供给公众的方式。


## 设计你的网址
一个干净优雅的URL方案是高质量Web应用程序中的一个重要细节。 Django鼓励美丽的URL设计，并且不会在URL中放置任何东西，如.php或.asp。

要为应用程序设计URL，您需要创建一个名为URLconf的Python模块。 您的应用程序的内容列表，它包含URL模式和Python回调函数之间的简单映射。 URLconf也用于从Python代码中分离URL。

下面是上面的Reporter / Article示例的URLconf：
> mysite/news/urls.py
```python
from django.urls import path

from . import views

urlpatterns = [
    path('articles/<int:year>/', views.year_archive),
    path('articles/<int:year>/<int:month>/', views.month_archive),
    path('articles/<int:year>/<int:month>/<int:pk>/', views.article_detail),
]
```
上面的代码将URL路径映射到Python回调函数（“视图”）。 路径字符串使用参数标签来“捕捉”来自URL的值。 当用户请求一个页面时，Django依次遍历每个路径，并停在第一个匹配请求URL的地方。 （如果没有匹配的话，Django会调用特殊的404视图。） 这是非常快的，因为路径在加载时被编译成正则表达式。

一旦一个URL模式匹配，Django将调用给定的视图，这是一个Python函数。 每个视图都会传递一个请求对象 - 其中包含请求元数据 - 以及模式中捕获的值。

例如，如果用户请求URL“/articles/ 2005/05/39323/”，Django会调用函数news.views.article_detail（request， year = 2005 ， month = 5， pk = 39323）。

## 视图
每个视图负责做以下两件事之一：返回包含请求页面内容的HttpResponse对象，或引发Http404之类的异常。 其余的由你决定。

通常，视图根据参数检索数据，加载模板并使用检索的数据呈现模板。 下面是上面的year_archive的示例视图：

> mysite/news/views.py
```python
from django.shortcuts import render

from .models import Article

def year_archive(request, year):
    a_list = Article.objects.filter(pub_date__year=year)
    context = {'year': year, 'article_list': a_list}
    return render(request, 'news/year_archive.html', context)
```
这个例子使用了Django的[模板系统](https://docs.djangoproject.com/en/2.0/topics/templates/)，它有几个强大的功能，但努力保持简单，非程序员使用。

## 设计您的模板
上面的代码加载了`news/year_archive.html`模板。

Django有一个模板搜索路径，它允许你最小化模板间的冗余。 在您的Django设置中，您可以指定一个目录列表来检查包含DIRS的模板。 如果第一个目录中不存在模板，则会检查第二个目录，依此类推。

比方说找到了`news/year_archive.html`模板。 以下是可能的样子：
> mysite/news/templates/news/year_archive.html
```html
{% extends "base.html" %}

{% block title %}Articles for {{ year }}{% endblock %}

{% block content %}
<h1>Articles for {{ year }}</h1>

{% for article in article_list %}
    <p>{{ article.headline }}</p>
    <p>By {{ article.reporter.full_name }}</p>
    <p>Published {{ article.pub_date|date:"F j, Y" }}</p>
{% endfor %}
{% endblock %}
```

变量被双曲花括号包围。 {{ article.headline }} 意思是输出文章的标题属性的值，但这点并不仅用于属性查找. 他们也可以做字典键查找，索引查找和函数调用。

注意{{ article.pub_date | date：“F j Y } }使用Unix风格的“管道”（“|”字符）。 这被称为模板过滤器，它是一种过滤变量值的方法。 在这种情况下，日期过滤器以给定的格式格式化Python日期时间对象（如PHP日期函数中所示）。

你可以链接尽可能多的过滤器，只要你愿意。 您可以编写[自定义模板过滤器](https://docs.djangoproject.com/en/2.0/howto/custom-template-tags/#howto-writing-custom-template-filters)。 您可以编写[自定义模板标签](https://docs.djangoproject.com/en/2.0/howto/custom-template-tags/)，后者在后台运行自定义Python代码。

最后，Django使用“模板继承”的概念。 这就是{% extends "base.html" %} 这意味着“首先加载名为'base'的模板，它定义了一堆块，并用下面的块填充块。”简而言之，这可以大大减少模板中的冗余：每个模板只能定义该模板的独特之处

以下是“base.html”模板，包括使用静态文件，可能如下所示：

> mysite/templates/base.html
```python
{% load static %}
<html>
<head>
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    <img src="{% static "images/sitelogo.png" %}" alt="Logo" />
    {% block content %}{% endblock %}
</body>
</html>
```

简单地说，它定义了站点的外观（带有站点的标识），并为子模板填充提供了“接口”。这使得重新设计一个网站就像更改一个文件 - 基本模板一样简单。

它还允许您在重复使用子模板的同时使用不同的基本模板创建网站的多个版本。Django的创建者已经使用这种技术创建了截然不同的移动版本的站点 - 只需创建一个新的基本模板即可。

请注意，如果您更喜欢其他系统，则不必使用Django的模板系统。虽然Django的模板系统与Django的模型层结合得非常好，但是没有任何东西强迫你使用它。对于这个问题，你也不必使用Django的数据库API。你可以使用另一个数据库抽象层，你可以读取XML文件，你可以从磁盘读取文件，或者任何你想要的。每一个Django模型，视图，模板都与下一个解耦。

