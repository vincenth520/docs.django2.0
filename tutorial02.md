# 编写你的第一个Django应用程序，第2部分

本教程从[Tutorial 1](tutorial02.md)中断的地方开始。 我们将设置数据库，创建您的第一个模型，并快速介绍Django自动生成的管理站点。

## 数据库设置

现在，打开**mysite/settings.py**。 这是一个普通的Python模块，代表Django设置。

默认情况下，配置使用SQLite。 如果你是数据库新手，或者你只是想尝试Django，这是最简单的选择。 SQLite包含在Python中，所以你不需要安装任何东西来支持你的数据库。 但是，当开始你的第一个真正的项目时，你可能想要使用像PostgreSQL这样更具可伸缩性的数据库，以避免数据库切换的麻烦。

如果您希望使用其他数据库，请安装相应的[database bindings](https://docs.djangoproject.com/en/2.0/topics/install/#database-installation)，并在[**DATABASES**](https://docs.djangoproject.com/en/2.0/ref/settings/#std:setting-DATABASES) '**default**'项中更改以下键以与数据库匹配连接设置：


- **Either** - '**django.db.backends.sqlite3**', '**django.db.backends.postgresql**', '**django.db.backends.mysql**', 或者 '**django.db.backends.oracle**'中的一种. [其他后端也可用](https://docs.djangoproject.com/en/2.0/ref/databases/#third-party-notes)。
- **NAME** – 你数据库的名称 如果你使用的是的SQLite, 数据库就是你电脑上的一个文件; 在这种情况下, NAME 应该设置为一个完整的绝对路径, 包括文件名称的绝对路径 默认值是**os.path.join（BASE_DIR， 'db.sqlite3'）**，将会把文件存储在您的项目目录中。

> ### 对于SQLite以外的数据库

> 如果你使用SQLite之外的数据库，请确保你已经创建了一个数据库。在数据库的交互式提示符下用`CREATE DATABASE database_name`来创建数据库。

> 还要确保提供的数据库用户mysite/settings.py 具有“创建数据库”权限。这允许自动创建 测试数据库，这将在稍后的教程中需要。

> 如果您使用的是SQLite，则无需事先创建任何内容 - 数据库文件将在需要时自动创建。

在编辑时**mysite/settings.py**，请设置**TIME_ZONE**为您的时区。

另外，请注意文件顶部**INSTALLED_APPS**的设置。它包含在此Django实例中激活的所有Django应用程序的名称。应用程序可以用于多个项目，您可以打包并分发这些应用程序以供他人在其项目中使用。

默认情况下，**INSTALLED_APPS**包含以下应用程序，所有这些应用程序都附带Django：

- **django.contrib.admin** - 管理网站。你会很快使用它。
- **django.contrib.auth** - 一个认证系统。
- **django.contrib.contenttypes** - 内容类型的框架。
- **django.contrib.sessions** - 会话框架。
- **django.contrib.messages** - 消息传递框架。
- **django.contrib.staticfiles** - 一个管理静态文件的框架。
默认情况下包含这些应用程序，以方便常见情况。

但是其中一些应用程序至少使用了一个数据库表，所以我们需要在数据库中创建表格，然后才能使用它们。为此，请运行以下命令：
```
python manage.py migrate
```
该**migrate**命令查看INSTALLED_APPS设置并根据mysite/settings.py文件中的数据库设置以及应用程序随附的数据库迁移（稍后将介绍这些数据库迁移）创建任何必需的数据库表。您会看到每个迁移应用的消息。如果您有兴趣，运行你的数据库命令行客户端并且输入命令
- **\dt (PostgreSQL)**
- **SHOW TABLES;(MySQL)** 
- **.schema (SQLite)** 
- **SELECT TABLE_NAME FROM USER_TABLES; (Oracle)**
来显示Django创建的表

> ### 对于极简主义者

> 就像我们上面所说的那样，默认应用程序将会全部安装，但不是每个人都需要它们。如果您不需要其中的任何一个或全部，请在运行之前在**INSTALLED_APPS**里面注释或删除相应的行 migrate。该 migrate命令将仅运行INSTALLED_APPS里的迁移 。

## 创建模型
现在我们将定义您的模型 - 实质上是您的数据库结构，并附加元数据。

> ### 哲学
> 模型是数据的唯一、可靠的来源。它包含存储的数据的基本字段和行为。Django遵循[DRY原则](https://docs.djangoproject.com/en/2.0/misc/design-philosophies/#dry)。我们的目标是在一个地方定义数据模型并自动从中获取数据。

> 迁移模式与Ruby On Rails不同，例如，迁移是完全来源于您的模型文件，实际上只是Django可以滚动更新数据库模式以匹配当前模型的历史记录。

在我们的简单投票应用程序中，我们将创建两个模型：**Question** 和**Choice**。**Question**有一个问题内容和一个提问时间。**Choice**有选项和投票记录。每一个**Choice**都与一个**Question**有关。

这些概念用简单的Python类表示。编辑**polls/models.py**文件，它看起来像这样：
> polls/models.py
```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

```

代码很简单。每个模型都是由一个子类django.db.models.model表示。每个模型都有许多类变量，每个变量代表模型中的一个数据库字段。

每个字段都由一个字段类的实例Field表示，例如，用于字符字段的CharField和DateTimeField。 这告诉Django每个字段包含什么类型的数据。

每个Field实例（如question_text或pub_date）是字段的名字。您将在Python代码中使用此值，您的数据库将使用它作为列名。

您可以使用一个可选的第一个位置参数来指定一个可读的名字。Field 这在Django的一些内省部分中被使用，而且它也被用作文档。 如果未提供此字段，则Django将使用机器可读的名称。 在这个例子中，我们只为Question.pub_date定义了一个人类可读的名字。 对于此模型中的所有其他字段，该字段的机器可读名称就足以作为其人类可读的名称。

一些Field类具有必需的参数。 CharField, 例如，需要你给它一个 max_length. 这不仅在数据库模式中使用，而且在验证中使用，我们将很快看到。

一个Field也可以有各种可选的参数；在这种情况下，我们将votes的default值设置为0。

最后，注意使用ForeignKey定义关系。 这告诉Django每个Choice都与一个Question有关。 Django支持所有常见的数据库关系：多对一，多对多和一对一。

### 激活模型
这一小部分模型代码给Django提供了很多信息。 有了它，Django能够：

为这个应用程序创建一个数据库模式（CREATE TABLE语句）。
创建一个Python数据库访问API来访问Question和Choice对象。
但首先我们需要告诉我们的项目安装了polls应用程序。


> ### 哲学
> Django应用程序是“可插入的”：您可以在多个项目中使用应用程序，并且可以分发应用程序，因为它们不必绑定到给定的Django安装。

要将该应用程序包含在我们的项目中，我们需要在**INSTALLED_APPS**设置中添加对其配置类的引用。 **PollsConfig**类位于**polls/apps.py**文件中，所以它的虚线路径是'**polls.apps.PollsConfig**'。 编辑**mysite/settings.py**文件并将该虚线路径添加到**INSTALLED_APPS**设置中。 它看起来像这样：

> mysite/settings.py
```python
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

现在Django知道包含polls应用程序。 让我们运行另一个命令：
```
$ python manage.py makemigrations polls
```

您应该看到类似于以下内容的内容：
```
Migrations for 'polls':
  polls/migrations/0001_initial.py:
    - Create model Choice
    - Create model Question
    - Add field question to choice
```

通过运行**makemigrations**，您告诉Django您已经对模型进行了一些更改（在这种情况下，您已经创建了新的模型），并且希望将更改存储为一个迁移。

迁移是Django如何将更改存储到模型（以及数据库模式） - 它们只是磁盘上的文件。 如果你喜欢，你可以阅读你的新模型的迁移。它是文件polls/migrations/0001_initial.py。 不用担心，Django每次创建时都不会读取它们，但是如果您想手动调整Django如何改变内容，那么它们的设计是可以人为编辑的。

有一个命令可以为你运行迁移并自动管理你的数据库模式 - 这就是所谓的migrate，我们马上就会谈到 - 但是首先让我们看看迁移的SQL将运行什么。 **sqlmigrate**命令使用迁移名称并返回它们的SQL：
```
$ python manage.py sqlmigrate polls 0001
```
您应该看到类似于以下的内容（为了便于阅读，我们将其重新格式化）：
```
BEGIN;
--
-- Create model Choice
--
CREATE TABLE "polls_choice" (
    "id" serial NOT NULL PRIMARY KEY,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL
);
--
-- Create model Question
--
CREATE TABLE "polls_question" (
    "id" serial NOT NULL PRIMARY KEY,
    "question_text" varchar(200) NOT NULL,
    "pub_date" timestamp with time zone NOT NULL
);
--
-- Add field question to choice
--
ALTER TABLE "polls_choice" ADD COLUMN "question_id" integer NOT NULL;
ALTER TABLE "polls_choice" ALTER COLUMN "question_id" DROP DEFAULT;
CREATE INDEX "polls_choice_7aa0f6ee" ON "polls_choice" ("question_id");
ALTER TABLE "polls_choice"
  ADD CONSTRAINT "polls_choice_question_id_246c99a640fbbd72_fk_polls_question_id"
    FOREIGN KEY ("question_id")
    REFERENCES "polls_question" ("id")
    DEFERRABLE INITIALLY DEFERRED;

COMMIT;
```

请注意以下几点：

- 实际的输出将取决于您使用的数据库。 上面的例子是为PostgreSQL生成的。
- 表名是通过结合应用程序的名称（polls）和模型的小写名称> question和choice自动生成的。 （您可以覆盖此行为。）
- 主键（ID）自动添加。 （你也可以重写这个。）
- 按照惯例，Django将"_id"附加到外键字段名称。 （是的，你也可以重写这个。）
- 外键关系通过FOREIGN KEY约束来显式化。
- 不要担心DEFERRABLE部分；这只是告诉PostgreSQL在事务结束之前不执行外键。
- 自动适配你所使用的的数据库，数据库特定的字段类型如auto_increment (MySQL), serial (PostgreSQL), 或者 integer主键autoincrement (SQLite) 都可以自动处理.字段名称的引用也是如此 - 例如使用双引号或单引号。
- sqlmigrate命令实际上并不在数据库上运行迁移 - 它只是将其打印到屏幕上，以便您可以看到SQL。 Django认为是必需的。 这对于检查Django将要做什么或者如果您有需要SQL脚本进行更改的数据库管理员很有用。
如果你感兴趣，你也可以运行`python manage.py check；`这将检查您的项目中的任何问题，而无需进行迁移或触摸数据库。

现在，再次运行**migrate**以在您的数据库中创建这些模型表：
```
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK
```

**migrate**命令采用所有尚未应用的迁移（Django使用名为django_migrations的数据库中的特殊表来跟踪哪些应用），并根据数据库运行它们本质上，将您对模型所做的更改与数据库中的模式同步。

迁移是非常强大的，随着时间的推移，您可以随着时间的推移来改变模型，而不需要删除数据库或表格，也不需要删除数据库或表格，而是专门用于实时升级数据库，而不会丢失数据。 我们将在本教程的后面部分更深入地介绍它们，但现在请记住进行模型更改的三步指南：

- 更改模型（在**models.py**中）。
- 运行`python manage.py makemigrations`来为这些更改创建迁移
- 运行`python manage.py migrate`以将这些更改应用到数据库。
有单独的命令来进行和应用迁移的原因是因为您将提交迁移到您的版本控制系统，并将其与您的应用程序一起发货；它们不仅使您的开发更容易，而且还可以被其他开发人员和生产使用。

请阅读[django-admin文档](https://docs.djangoproject.com/en/2.0/ref/django-admin/)了解**manage.py**实用程序可以执行的操作的完整信息。

### 玩转API
现在，让我们进入交互式Python shell并使用Django提供的API。 要调用Python shell，请使用以下命令：
```
$ python manage.py shell
```

我们使用这个命令而不是简单地输入"python"命令是因为 **manage.py** 将 **DJANGO_SETTINGS_MODULE** 设置为环境变量, 它将Python 文件mysite/settings.py 的引入路径告诉Django.

一旦你在shell中，浏览[database API](https://docs.djangoproject.com/en/2.0/topics/db/queries/)：

```
>>> from polls.models import Question, Choice   # 将我们刚刚写好的模型类引入.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# 创建一个新的调查问题.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```
等一下。 Question object （1）> 不是这个对象的有效表现形式。 我们通过编辑Question模型（**polls/models.py**文件中）并添加一个`__str__()`方法来修复Question和Choice：

polls/models.py
```python
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```
将`__str__()`方法添加到模型中很重要，不仅为了您在处理交互提示时的方便，还因为在Django自动生成的管理中使用了对象表示。

请注意，这些是普通的Python方法。 让我们添加一个自定义的方法，只是为了演示：
> polls/models.py
```
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

```
请注意添加`import datetime` 和 `from django.utils import timezone`， 分别引用Python的标准datetime模块和django.utils.timezone Django的时区相关。如果您不熟悉Python中的时区处理，您可以在[时区支持文档](https://docs.djangoproject.com/en/2.0/topics/i18n/timezones/)中了解更多信息。

再次运行`python manage.py shell`，保存这些更改并启动一个新的Python交互式shell：
```
>>> from polls.models import Question, Choice

# 测试刚才添加的 __str__() 是否正常工作
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django提供了丰富的数据库查找API
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith='What')
<QuerySet [<Question: What's up?>]>

# 获取今年的question
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# 查找不存在的id，将抛出一个异常
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# 获取主键为1的值
>>> Question.objects.get(pk=1)
<Question: What's up?>

# 测试用户自定义方法是否正常执行
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# 给这个Question两个选择。创建调用构建一个新的Choice对象
# INSERT语句，增加了选择集
# 可用的选择和返回新的选择对象。
# Django创建一套持“一个外键关系的另一面（如一个问题的选择）可以通过API访问。

>>> q = Question.objects.get(pk=1)

# 显示相关对象集中的所有选项——到目前为止没有。
>>> q.choice_set.all()
<QuerySet []>

# 创建3个choices.
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# Choice对象对其相关Question对象具有API访问权限。
>>> c.question
<Question: What's up?>

# 反之亦然：Question对象获得对Choice对象的访问权。
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# API会自动按照你需要的关系进行跟踪。
# 使用双下划线分开的关系。
#找到任何问题的pub_date是今年所有的选择
#（重用我们上面创建的current_year变量）。
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# 让我们删除一个列，使用delete()。
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```

有关模型关系的更多信息，请参阅[访问相关对象](https://docs.djangoproject.com/en/2.0/ref/models/relations/)。有关如何使用双下划线通过API执行字段查找的更多信息，请参阅[字段查找](https://docs.djangoproject.com/en/2.0/topics/db/queries/#field-lookups-intro)。有关数据库API的完整详细信息，请参阅我们的[数据库API参考](https://docs.djangoproject.com/en/2.0/topics/db/queries/)。

### 介绍Django Admin

> ### 哲学
> 为您的员工或客户生成管理网站来添加，更改和删除内容是一件繁琐的工作，不需要太多的创造力。 出于这个原因，Django完全自动为模型创建管理界面。

> Django是在新闻编辑环境下编写的，“内容发布者”和“公共”网站之间有着非常明确的分离。 网站管理员使用该系统添加新闻故事，事件，体育比分等，并将该内容显示在公共场所。 Django解决了为站点管理员创建统一界面来编辑内容的问题。

> 管理界面不向普通用户开放。 只提供给网站管理者


### 创建管理员账户

首先，我们需要创建一个可以登录管理网站的用户。 运行以下命令：
```
$ python manage.py createsuperuser
```
输入你想要的用户名，然后按回车。
```
Username: admin
```
您将被提示输入您想要的电子邮件地址：
```
Email address: admin@example.com
```
最后一步是输入你的密码。 您将被要求输入两次密码，第二次作为第一次确认。
```
Password: **********
Password (again): *********
Superuser created successfully.
```

### 启动开发服务器

Django管理站点默认是激活的。 让我们开始开发服务器，并探索它。

如果服务器没有运行，就像这样启动：
```
$ python manage.py runserver
```
现在，打开Web浏览器并转到本地域的“/admin/” - 例如http://127.0.0.1:8000/admin/。 您应该看到管理员的登录屏幕：
![](https://docs.djangoproject.com/en/2.0/_images/admin01.png)

由于自动[翻译](https://docs.djangoproject.com/en/2.0/topics/i18n/translation/)默认情况下处于打开状态，因此登录界面可能会以您自己的语言显示，具体取决于您的浏览器设置，以及Django是否具有该语言的翻译。

### 输入管理网站
现在，试着使用您在上一步中创建的超级用户帐户登录。 你应该看到Django的管理索引页面：
![](https://docs.djangoproject.com/en/2.0/_images/admin02.png)

您应该看到几种类型的可编辑内容：Groups和Users。 它们由Django提供的认证框架django.contrib.auth提供。

### 在admin中修改poll应用程序

但是我们的poll程序在哪里？ 它并没有显示在管理索引页面上。

为此需要做的是: 我们需要网站控制台 **Question** 对象有一个管理接口， 要做到这点，打开**polls/admin.py**文件，并编辑成：

> polls/admin.py
```
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

### 探索自由的管理功能
现在我们已经注册了Question，Django知道它应该显示在管理索引页面上：
![](https://docs.djangoproject.com/en/2.0/_images/admin03t.png)

点击“**Questions**”。 现在，您可以在“change”页面寻求问题。 此页面显示数据库中的所有问题，并让您选择一个来更改它。 我们之前创建的“What's up？”问题是：
![](https://docs.djangoproject.com/en/2.0/_images/admin04t.png)

点击“What's up？”问题进行编辑：
![](https://docs.djangoproject.com/en/2.0/_images/admin05t.png)

这里需要注意的是：

- 表单是从Question模型自动生成的。
- 不同的模型字段类型（DateTimeField，CharField）对应于相应的HTML输入小部件。 每种类型的领域都知道如何在Django管理中显示自己。
- 每个DateTimeField获得免费的JavaScript快捷方式。 日期得到一个“Today”的快捷方式和日历弹出窗口，时间得到一个“Now”的快捷方式和一个方便的弹出窗口，列出通常输入的时间。

页面的底部给你几个选项：

- 保存 - 保存更改并返回到此类型对象的更改列表页面。
- 保存并继续编辑 - 保存更改并重新加载此对象的管理页面。
- 保存并添加另一个 - 保存更改并为此类型的对象加载一个新的空白表单。
- 删除 - 显示删除确认页面。

如果“发布日期”的值与您在[Tutorial 1](tutorial01.md)中创建问题的时间不匹配，则可能意味着您忘记为TIME_ZONE设置正确的值设置。 更改它，重新加载页面，并检查出现正确的值。

通过单击“Today”和“Now”快捷方式更改“Date published”。 然后点击“保存并继续编辑”，然后点击右上角的“历史记录”。 您将看到一个页面，其中列出了通过Django管理员对此对象所做的所有更改，以及进行更改的人员的时间戳和用户名：
![](https://docs.djangoproject.com/en/2.0/_images/admin06t.png)
当你尝试了模型提供的一些API，熟悉了你的管理页面的, 就可以看一看[Tutorial 3](tutorial03.md) ，去学习如何给你的polls应用添加更多Views。