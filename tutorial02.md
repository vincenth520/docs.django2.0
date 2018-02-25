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
