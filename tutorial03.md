# 编写你的第一个Django应用程序，第3部分

本教程从[Tutorial 2](tutorial02.md)停止的地方开始。 我们正在继续Web调查应用程序，并将重点放在创建公共接口 - “视图”上。

## 概述

视图是Django应用程序中网页的“类型”，它通常用于特定的功能并具有特定的模板。 例如，在博客应用程序中，您可能有以下视图：

- 博客主页 - 显示最新的几个条目。
- 输入“详细信息”页面 - 永久链接页面为单个条目。
- 基于年份的存档页面 - 显示给定年份中所有条目的月份。
- 基于月份的存档页面 - 显示给定月份中所有条目的日期。
- 基于日期的归档页面 - 显示给定日期的所有条目。
- 评论操作 - 处理对给定条目的发布评论。
在我们的投票应用程序中，我们将有以下四个视图：

- 问题“索引”页面 - 显示最新的几个问题。
- 问题“详细”页面 - 显示一个问题文本，没有结果，但有一个表格投票。
- 问题“结果”页面 - 显示特定问题的结果。
- 投票行动 - 处理针对特定问题的特定选择的投票。
在Django中，网页和其他内容由视图传递。 每个视图都由一个简单的Python函数（或基于类的视图的方法）表示。 Django将通过检查请求的URL（准确地说，域名后的URL部分）来选择一个视图。

现在在网络上你可能遇到过"ME2/Sites/dirmod.asp?sid=&type=gen&mod=Core+Pages&gid=A6CD4967199A42D9B65B1B"等。 令人庆幸的是Django允许我们使用比这更优雅的URL模式。

URL模式是URL的一般形式，例如：/newsarchive/<year>/<month>/。

为了从一个URL获得一个视图，Django使用了所谓的“URLconf”。 URLconf将URL模式映射到视图。

本教程提供了使用URLconf的基本说明，您可以参考[URL dispatcher](https://docs.djangoproject.com/en/2.0/topics/http/urls/)了解更多信息。

## 设置更多的视图

现在我们再添加一些视图到polls/views.py。 这些视图略微不同，因为各自名称不同：
> polls/views.py
```python
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

通过添加以下**path()**调用，将这些新视图连接到**polls.urls**模块：
> polls/urls.py
```python
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
当你在浏览器访问
看看你的浏览器，在'`/polls/34/`'。 它开始运行 **detail()** 方法和你在URL中提供的ID. 尝试“/polls/34/results/”和“/polls/34/vote/” - 这些将显示占位符结果和投票页面。

当有人从你的网站上请求一个页面 - 比如说“/polls/34/”时，Django将加载mysite.urls Python模块，因为它是由**ROOT_URLCONF**设置指向的。 它找到名为**urlpatterns**的变量并按顺序遍历这些模式。 在'**polls/**'找到匹配项后，将匹配的文本（"**polls/**"）分离并发送剩余的文本 - "**34/**" 在那里匹配'**<int:question_id>/**'，导致对**detail()**视图的调用如下所示：
```
detail(request=<HttpRequest object>, question_id=34)
```
**question_id=34**部分来自**<int:question_id>**。 使用尖括号“捕获”部分URL并将其作为关键字参数发送到视图函数。 字符串的`:question_id>`部分定义了将用于标识匹配模式的名称，而 `<int:` 部分是一个转换器，它决定了哪些模式应该匹配这部分URL路径。

没有必要添加额外的URL后缀，如**.html** - 除非你想，在这种情况下你可以这样做：

path('polls/latest.html', views.index),
但是，不要这样做。 这很愚蠢。

## 编写实际执行某些操作的视图

每个视图负责执行以下两种操作之一：返回包含所请求页面内容的HttpResponse对象，或引发Http404等异常。 其余的由你决定。

您的视图可以从数据库中读取记录，也可以不读取。 它可以使用一个模板系统，比如Django或者第三方的Python模板系统。 它可以生成一个PDF文件，输出XML，创建一个ZIP文件，你可以使用想要的Python库。

所有的Django想要的是HttpResponse。 或者是一个异常。

由于方便，我们使用Django自己的数据库API，我们在[Tutorial 2](tutorial02.md)中介绍了它。 下面是一个新的index()视图，根据出版日期,它显示系统中最新的5个轮询问题，用逗号分隔：
> polls/views.py
```python
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# detail, results, vote等页面代码不改变
```
这里有一个问题：页面的设计在视图中是硬编码的。 如果你想改变页面的外观，你必须编辑这个Python代码。 所以让我们使用Django的模板系统，通过创建视图可以使用的模板来将设计从Python中分离出来。

首先，在您的polls目录中创建一个名为**templates**的目录。 Django将在那里寻找模板。

你的项目的[**TEMPLATES**](https://docs.djangoproject.com/en/2.0/ref/settings/#std:setting-TEMPLATES)设置描述了Django如何加载和渲染模板。 默认设置文件配置一个**DjangoTemplates**后端，其**APP_DIRS**选项设置为**True**。 按照惯例**DjangoTemplates**在每个**INSTALLED_APPS**中查找“**templates**”子目录。

在刚创建的**templates**目录中，创建另一个名为**polls**的目录，并在其中创建一个名为**index.html**的文件。 换句话说，你的模板应该在**polls/templates/polls/index.html**。 因为**app_directories**的模板加载器的工作方式如上所述，因此您可以在Django中简单地引用该模板**polls/index.html**

> # 模板命名空间
> 现在我们可以避免将模板直接放入 **polls/templates**（而不是创建另一个**polls**子目录），但这实际上是一个糟糕的主意。Django会选择找到的名称匹配的第一个模板，如果在不同的应用程序中有同名的模板，Django将无法区分它们。我们需要能够将Django指向正确的位置，并且最简单的方法是通过对它们进行命名来确保它是正确的。也就是说，将这些模板放在为应用程序本身命名的另一个目录中。

将下面的代码放入templates中：
> polls/templates/polls/index.html
```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```
现在,让我们修改**polls/views.py**来使用模板：
> polls/views.py
```python
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```
该代码加载名为**polls/index.html**的模板并将其传递给上下文。 上下文是一个将模板变量名映射到Python对象的字典。

通过将浏览器指向"/polls/"来加载页面，您应该看到一个包含[Tutorial 2](tutorial02.md)中“What's up”问题列表。 链接指向问题的详细信息页面。

## 快捷方式：[render()](https://docs.djangoproject.com/en/2.0/topics/http/shortcuts/#django.shortcuts.render) 
加载模板，填充上下文并返回一个HttpResponse对象与渲染模板的结果是非常普遍的习惯用法。 Django提供了一个捷径。 下面是完整的index()视图，重写了：

> polls/views.py
```python
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```
请注意，一旦我们在所有这些视图中完成了这些操作，我们就不再需要导入**loader**和**HttpResponse**（您将需要保留**HttpResponse**如果仍然有**detail**，**results**和**vote**的方法）。

[**render()**](https://docs.djangoproject.com/en/2.0/topics/http/shortcuts/#django.shortcuts.render) 函数将request对象作为第一个参数，将模板名称作为第二个参数，将字典作为可选的第三个参数。 它返回给定上下文给定模板的**HttpResponse**对象。

## 编写一个 404 ( 页面未找到 ) 视图
现在，让我们解决问题详细信息视图 - 显示给定投票的问题文本的页面。 这是视图内容：

> polls/views.py
```python
from django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```
这里的新概念：如果所请求的ID的问题不存在，则该视图引发Http404异常。

稍后我们将讨论可以放在**polls/detail.html**模板中的内容，但是如果您希望快速获得上述示例的工作方式，则只需包含以下内容的文件：

> polls/templates/polls/detail.html
```html
{{ question }}
```

### 快捷方式：[get_object_or_404()](https://docs.djangoproject.com/en/2.0/topics/http/shortcuts/#django.shortcuts.get_object_or_404)
如果对象不存在，那么使用[**get()**](https://docs.djangoproject.com/en/2.0/ref/models/querysets/#django.db.models.query.QuerySet.get)并提升[Http404](https://docs.djangoproject.com/en/2.0/topics/http/views/#django.http.Http404)是一个非常常见的习惯用法。 Django提供了一个捷径。 这里是**detail()**视图，重写了：

> polls/views.py
```python
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```
[`get_object_or_404()`](https://docs.djangoproject.com/en/2.0/topics/http/shortcuts/#django.shortcuts.get_object_or_404)函数将Django模型作为其第一个参数和任意数量的关键字参数传递给模型管理器的`get()`函数。 如果对象不存在，它会引发**Http404**。

> # 哲学

> 为什么我们使用助手函数[`get_object_or_404()`](https://docs.djangoproject.com/en/2.0/topics/http/shortcuts/#django.shortcuts.get_object_or_404) 而不是自动捕获**ObjectDoesNotExist**更高级别的 异常，或者使用模型API **Http404**而不是 **ObjectDoesNotExist**？

> 因为那会将模型图层耦合到视图图层。Django最重要的设计目标之一是保持松耦合。**django.shortcuts**模块中引入了一些受控耦合。

还有一个[`get_list_or_404()`](https://docs.djangoproject.com/en/2.0/topics/http/shortcuts/#django.shortcuts.get_list_or_404)功能，其功能就像[`get_object_or_404()`](https://docs.djangoproject.com/en/2.0/topics/http/shortcuts/#django.shortcuts.get_object_or_404)- 除了使用 [`filter()`](https://docs.djangoproject.com/en/2.0/ref/models/querysets/#django.db.models.query.QuerySet.filter)而不是 [`get()`](https://docs.djangoproject.com/en/2.0/ref/models/querysets/#django.db.models.query.QuerySet.get)。[**Http404**](https://docs.djangoproject.com/en/2.0/topics/http/views/#django.http.Http404)如果列表为空，则会引发 此问题。

### 使用模板系统
回到**detail()**我们的投票应用程序的视图。鉴于上下文变量question，以下是**polls/detail.html**模板的样子：

> polls/templates/polls/detail.html

```python
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

模板系统使用点查找语法来访问变量属性。 在{{ question.question_text }}这个例子中，Django首先在question对象上做字典查询。 否则，它会尝试一个属性查找 如果属性查找失败，它会尝试一个列表索引查找。

方法调用发生在**{% for %}**循环中：**question.choice_set.all**被解释为Python的代码**question.choice_set.all()**，它返回一个由Choice对象组成的可迭代对象，并将其用于**{% for %}**标签。

有关模板的更多信息，请参阅[template guide](https://docs.djangoproject.com/en/2.0/topics/templates/)。

## 删除模板中的硬编码URL

记得吗？在**polls/index.html**模板中，我们连接到polls的链接是硬编码成这个样子的：
```
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```
问题出在硬编码，紧耦合使得在大量的模板中修改 URLs 成为富有挑战性的项目。 然而，因为你在**polls.urls**模块的`url()`函数中定义了**name** 参数，你可以通过使用**{% url %}**模板标签来移除对你的URL配置中定义的特定的URL的依赖：
```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```
工作方式是通过查找**polls.urls**模块中指定的URL定义。 您可以在下面定义具体的“**detail**”的URL名称：
```
...
# the 'name' value as called by the {% url %} template tag
path('<int:question_id>/', views.detail, name='detail'),
...
```
如果你想修改polls应用的detail视图的URL, 类似于**polls/specifics/12/**的形式，你可以修改**polls/urls.py**中的内容:
```
...
# added the word 'specifics'
path('specifics/<int:question_id>/', views.detail, name='detail'),
```

## URL名称的命名空间
教程项目只有一个应用程序，polls。 在真正的Django项目中，可能会有五个，十个，二十个应用程序或更多。 Django如何区分它们之间的URL名称？ 例如，polls应用程序具有detail视图，因此同一个项目中的应用程序可能会用于博客。 如何让Django在使用{％ url ％}时知道为网址创建哪个应用视图模板标签？

答案是将命名空间添加到您的**URLconf**。 在**polls/urls.py**文件中，继续添加**app_name**来设置应用程序名称空间：

> polls/urls.py
```
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
然后修改**polls/index.html**模板中的：

> polls/templates/polls/index.html
```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```
修改为

> polls/templates/polls/index.html
```
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```
编写完上述内容后，请阅读本教程的[part 4 of this tutorial](tutorial04.md)以了解简单表单处理和通用视图。










