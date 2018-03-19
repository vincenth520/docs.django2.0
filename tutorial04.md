# 编写你的第一个Django应用程序，第4部分

本教程从[Tutorial 3](tutorial03.md)停止的地方开始。 我们正在继续Web调查应用程序，将重点放在简单的表单处理和削减我们的代码

## 编写一个简单的表单

让我们把在上一节教程中的（"polls / detail.html"）内容更新下，在模板中添加一个一个 <form>组件：

> polls/templates/polls/detail.html
```
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
{% endfor %}
<input type="submit" value="Vote" />
</form>
```

简要说明：

上面的模板为每个问题选项显示一个单选按钮。 每个单选按钮的**value**是关联的问题选项的ID。 每个单选按钮的**name**是"**choice**"。 这意味着，当有人选择其中一个单选按钮并提交表单时，它将发送POST数据choice=#，其中＃是所选选项的ID。 这是HTML表单的基本概念。
我们将表单的action设置为`{% url {poll：'vote' question.id ％}`，然后我们设置method ="post"。 使用`method="post"`（与`method="get"`相对）非常重要，因为提交此表单的行为将会改变数据服务器端。 无论何时您创建一个可以改变数据服务器端的表单，都可以使用`method="post"`。 这个技巧并不是针对Django的。这是HTML表单的基本概念。
`forloop.counter`表示for标签经过了多少次循环
由于我们正在创建一个POST表单（这可能会影响修改数据），所以我们需要担心跨站点请求伪造。 值得庆幸的是，您不必太担心，因为Django带有一个非常容易使用的系统来保护它。 简而言之，所有以内部URL为目标的POST表单都应该使用`{% csrf_token %}`模板标记。
现在，我们来创建一个Django视图来处理提交的数据，并执行一些操作。 请记住，在[Tutorial 3](tutorial03.md)中，我们为polls应用程序创建了一个包含以下行的URLconf：
> polls/urls.py
```
path('<int:question_id>/vote/', views.vote, name='vote'),
```
我们还创建了`vote()`函数的虚拟实现。 我们来创建一个真正的版本。 将以下内容添加到**polls/views.py**中：
> polls/views.py
```
from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect, HttpResponse
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

这段代码包含了本教程中尚未涉及的一些内容：

**request.POST**是一个类似字典的对象，允许您通过键名访问提交的数据。 在这例子中， **request.POST ['choice']**以字符串形式返回所选选项的ID。 **request.POST**值始终是字符串。

请注意，Django还提供了**request.GET**来以相同的方式访问GET数据 - 但是我们在代码中显式使用了**request.POST**，以确保数据是唯一的通过POST调用修改。

如果POST数据中没有提供**choice**，**request.POST['choice']**将会引发**KeyError** 如果没有给出**choice**，上面的代码检查**KeyError**并重新显示问题表单。

递增选择计数后，代码将返回**HttpResponseRedirect**而不是正常的**HttpResponse**。 **HttpResponseRedirect**接受一个参数：用户将被重定向到的URL（在这种情况下，我们如何构造URL）。

正如上面的Python注释所指出的那样，在成功处理POST数据之后，您应该始终返回一个**HttpResponseRedirect**。 这个技巧并不是针对Django的。这只是一个很好的Web开发实践。

在这个例子中，我们在**HttpResponseRedirect**构造函数中使用`reverse()`函数。 该功能有助于避免在视图功能中硬编码URL。 给出了我们想要传递控制权的视图的名称以及指向该视图的URL模式的可变部分。 在这种情况下，使用我们在[Tutorial 3](tutorial03.md)中设置的URLconf，这个`reverse()`调用会返回一个字符串
`'/polls/3/results/'`
其中3是**question.id**的值。 然后这个重定向的URL将调用'**results**'视图来显示最终页面。

如[Tutorial 3](tutorial03.md)所述，**request**是一个**HttpRequest**对象。 有关HttpRequest对象的更多信息，请参阅[request and response documentation](https://docs.djangoproject.com/en/2.0/ref/request-response/)。

有人在问题中投票后，`vote()`视图重定向到问题的**results**页面。 我们来修改下这个视图：

> polls/views.py
```
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```
这与[Tutorial 3](tutorial03.md)中的`detail()`视图几乎完全相同。 唯一的区别是模板名称。 我们稍后会修复这个冗余。

现在，创建一个**polls/results.html**模板：

> polls/templates/polls/results.html
```
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```
现在，在浏览器中转到[/polls/1/](/polls/1/)，然后在问题中投票。 您应该会看到每次投票时更新的结果页面。 如果您提交表单没有选择一个选项，您应该看到错误信息。

> 注意
我们的`vote()`视图的代码确实有一个小问题。 它首先从数据库获取**selected_choice**对象，然后计算**votes**的新值，然后将其保存回数据库。 如果您的网站的两个用户尝试在完全同时投票，这可能会出错：相同的值，比如42，将被检索为votes。 然后，为两个用户计算并保存43的新值，但44将是期望值。
这被称为竞赛条件。 如果您有兴趣，可以阅读[Avoiding race conditions using F()](https://docs.djangoproject.com/en/2.0/ref/models/expressions/#avoiding-race-conditions-using-f)避免竞争条件以了解如何解决此问题。

## 使用通用视图：优化代码

`detail()`（来自[Tutorial 3](tutorial03.md)）和`results()`视图非常简单 - 如上所述，是冗余的。 显示轮询列表的`index()`视图是相似的。

这些视图代表了基本Web开发的常见情况：根据URL中传递的参数从数据库获取数据，加载模板并返回呈现的模板。 因为这很常见，所以Django提供了一个叫做“**通用视图**”系统的快捷方式。

通用视图抽象常用模式，甚至不需要编写Python代码来编写应用程序。

让我们完善我们的投票应用程序使用通用的意见系统，所以我们可以删除一堆我们已有的代码。 我们只需要采取几个步骤来进行转换。 我们会：

- 修改URLconf。
- 删除一些旧的不需要的视图。
- 基于Django的通用视图引入新的视图。
请阅读详细信息。

> 为什么要重构？
通常，在编写Django应用程序时，您将评估通用视图是否适合您的问题，从一开始就使用它们，而不是在中途重构代码。 但是这个教程故意把重点放在写“观点”，直到现在，专注于核心概念。
在开始使用计算器之前，你应该知道基础数学。

## 修改URLconf 
首先，打开**polls/urls.py** URLconf并将其更改如下：

> polls/urls.py
```python
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
请注意，第二和第三模式的路径字符串中的匹配模式的名称已从**<question_id>**改变为**<pk>**。

## 修改视图
接下来，我们将删除旧的**index**，**detail**和**results**视图，并使用Django的通用视图。 为此，请打开**polls/views.py**文件，并将其更改如下：

> polls/views.py
```python
from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'


class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'


def vote(request, question_id):
    ... # same as above, no changes needed.

```
我们在这里使用两个通用视图：**ListView**和**DetailView**。 这两个视图分别抽象出“**显示对象列表**”和“**显示特定类型对象的详细页面**”的概念。

每个通用视图都需要知道它将采取何种模式。 这是使用model属性提供的。
**DetailView** 通用视图从URL获取一个键值("pk"), 所以我们把 **question_id**改成**pk**
默认情况下，DetailView通用视图使用名为**app名称/model名称_detail.html的模板**。 在我们的例子中，它将使用模板"**polls/question_detail.html**"。 **template_name**属性用于告诉Django使用特定的模板名称，而不是自动生成的默认模板名称。 我们还为结果列表视图指定了**template_name** - 这确保了结果视图和详细视图在呈现时具有不同的外观，即使它们都是** DetailView**幕后。

类似地，**ListView**使用一个叫做**app name/model name_list.html**的默认模板；我们使用**template_name** 来告诉ListView 使用我们自己已经存在的"**polls/index.html**"模板。

在本教程的前几部分中，模板已经提供了一个包含**question**和**latest_question_list**上下文变量的上下文。 对于**DetailView** ，**question**变量会自动提供—— 因为我们使用Django 的模型 (**Question**)， Django 能够为**context** 变量决定一个合适的名字。 但是，对于**ListView**，自动生成的上下文变量是**question_list**。 为了覆盖这个，我们提供**context_object_name**属性，指定我们要使用**latest_question_list**来代替。 作为一种替代方法，您可以更改模板以匹配新的默认上下文变量 - 但只要告诉Django使用您想要的变量就容易多了。

运行服务器，并根据通用视图使用新的轮询应用程序。

有关通用视图的完整详细信息，请参阅[generic views documentation](https://docs.djangoproject.com/en/2.0/topics/class-based-views/)。

当您熟悉表单和通用视图时，请阅读[part 5 of this tutorial](tutorial05.md)了解测试我们的投票应用程序。