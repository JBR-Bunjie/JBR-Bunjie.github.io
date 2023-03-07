## 官网原教程：

### PART 2：

[Writing your first Django app, part 2 | Django documentation | Django (djangoproject.com)](https://docs.djangoproject.com/en/4.0/intro/tutorial02/)

#### 概述：

- 建立数据库：基于sqlite数据库，介绍project module下settings中包括如何更换其他数据库的部分内容，并通过migrate命令快速创建所需数据库
- 

#### 复述：

##### 1. 建立数据库

###### a. 确认数据库设置

Django默认使用SQLite作为默认数据库，因为这是Python内置的数据库。

如果需要更换别的数据库的话，首先需要安装合适的database bindings，然后改变project module下settings文件中的设置文件中 `DATABASES` `'default'` 项目中的一些键值，包括：

> - [`ENGINE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std-setting-DATABASE-ENGINE) – Either `'django.db.backends.sqlite3'`, `'django.db.backends.postgresql'`, `'django.db.backends.mysql'`, or `'django.db.backends.oracle'`. Other backends are [also available](https://docs.djangoproject.com/en/4.0/ref/databases/#third-party-notes).
> - [`NAME`](https://docs.djangoproject.com/en/4.0/ref/settings/#std-setting-NAME) – The name of your database. If you’re using SQLite, the database will be a file on your computer; in that case, [`NAME`](https://docs.djangoproject.com/en/4.0/ref/settings/#std-setting-NAME) should be the full absolute path, including filename, of that file. The default value, `BASE_DIR / 'db.sqlite3'`, will store the file in your project directory.

并且如果不使用 SQLite，则必须添加更多内容，比如 [`USER`](https://docs.djangoproject.com/zh-hans/4.0/ref/settings/#std-setting-USER) 、 [`PASSWORD`](https://docs.djangoproject.com/zh-hans/4.0/ref/settings/#std-setting-PASSWORD) 、 [`HOST`](https://docs.djangoproject.com/zh-hans/4.0/ref/settings/#std-setting-HOST) 等。更多内容请查阅文档：[`DATABASES`](https://docs.djangoproject.com/zh-hans/4.0/ref/settings/#std-setting-DATABASES) 。

>如果你使用了 SQLite 以外的数据库，请确认在使用前已经创建了数据库。你可以通过在你的数据库交互式命令行中使用 "`CREATE DATABASE database_name;`" 命令来完成这件事。
>
>另外，还要确保该数据库用户中提供 `mysite/settings.py` 具有 "create database" 权限。这使得自动创建的 [test database](https://docs.djangoproject.com/zh-hans/4.0/topics/testing/overview/#the-test-database) 能被以后的教程使用。
>
>如果你使用 SQLite，那么你不需要在使用前做任何事——数据库会在需要的时候自动创建。

###### b. 按规则设置时区

注意时区属性的格式

###### c. check `INSTALLED_APPS` items

the item, [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std-setting-INSTALLED_APPS) setting at the top of the file, holds the names of all Django applications that are **activated** in this Django instance. 

Apps can be used in multiple projects, and you can package and distribute them for use by others in their projects.

> By default, [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std-setting-INSTALLED_APPS) contains the following apps, all of which come with Django:
>
> - [`django.contrib.admin`](https://docs.djangoproject.com/en/4.0/ref/contrib/admin/#module-django.contrib.admin) – The admin site. You’ll use it shortly.
> - [`django.contrib.auth`](https://docs.djangoproject.com/en/4.0/topics/auth/#module-django.contrib.auth) – An authentication system.
> - [`django.contrib.contenttypes`](https://docs.djangoproject.com/en/4.0/ref/contrib/contenttypes/#module-django.contrib.contenttypes) – A framework for content types.
> - [`django.contrib.sessions`](https://docs.djangoproject.com/en/4.0/topics/http/sessions/#module-django.contrib.sessions) – A session framework.
> - [`django.contrib.messages`](https://docs.djangoproject.com/en/4.0/ref/contrib/messages/#module-django.contrib.messages) – A messaging framework.
> - [`django.contrib.staticfiles`](https://docs.djangoproject.com/en/4.0/ref/contrib/staticfiles/#module-django.contrib.staticfiles) – A framework for managing static files.
>
> These applications are included by default as a convenience for the common case.

Some of these applications make use of at least one database table, though, so we need to create the tables in the database before we can use them. To do that, run the following command:

```python
py manage.py migrate
```

as we said before "the database file will be created automatically when it is needed.", the sqlite database finally appeared with those tables above.

>The [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate) command looks at the [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std-setting-INSTALLED_APPS) setting and creates any necessary database tables according to the database settings in your `mysite/settings.py` file and the database migrations shipped with the app (we’ll cover those later). You’ll see a message for each migration it applies. If you’re interested, run the command-line client for your database and type `\dt` (PostgreSQL), `SHOW TABLES;` (MariaDB, MySQL), `.tables` (SQLite), or `SELECT TABLE_NAME FROM USER_TABLES;` (Oracle) to display the tables Django created.

##### 2. model文件编写

>Now we’ll define your models – essentially, your database layout, with additional metadata.
>
>在 Django 里写一个数据库驱动的 Web 应用的第一步是定义模型 - 也就是数据库结构设计和附加的其它元数据。

###### a. 什么是一个model？

>**Philosophy**
>
>A model is the single, definitive source of information about your data. It contains the essential fields and behaviors of the data you’re storing. Django follows the [DRY Principle](https://docs.djangoproject.com/en/4.0/misc/design-philosophies/#dry). The goal is to define your data model in one place and automatically derive things from it.
>
>This includes the migrations - unlike in Ruby On Rails, for example, migrations are entirely derived from your models file, and are essentially a history that Django can roll through to update your database schema to match your current models.

###### b. examples

In our poll app, we’ll create two models: `Question` and `Choice`. 

- A `Question` has a question and a publication date. 
- A `Choice` has two fields: the text of the choice and a vote tally. Each `Choice` is associated with a `Question`.

These concepts are represented by Python classes. Edit the `polls/models.py` file so it looks like this:

```
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

Here, **each model is represented by a class that subclasses [`django.db.models.Model`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model). Each model has a number of class variables, each of which represents a database field in the model.**

Each field is represented by an instance of a [`Field`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field) class – e.g., [`CharField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.CharField) for character fields and [`DateTimeField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.DateTimeField) for datetimes. This tells Django what type of data each field holds.

**The name of each [`Field`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field) instance (e.g. `question_text` or `pub_date`) is the field’s name**, in machine-friendly format. You’ll use this value in your Python code, and your database will use it as the column name.

You can use an optional first positional argument to a [`Field`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field) to designate a human-readable name. That’s used in a couple of introspective parts of Django, and it doubles as documentation. If this field isn’t provided, Django will use the machine-readable name. In this example, we’ve only defined a human-readable name for `Question.pub_date`. For all other fields in this model, the field’s machine-readable name will suffice as its human-readable name.

**Some [`Field`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field) classes have required arguments.** [`CharField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.CharField), for example, requires that you give it a [`max_length`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.CharField.max_length). That’s used not only in the database schema, but in validation, as we’ll soon see.

A [`Field`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field) can also have various optional arguments; in this case, we’ve set the [`default`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.default) value of `votes` to 0.

Finally, note a relationship is defined, using [`ForeignKey`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ForeignKey). That tells Django each `Choice` is related to a single `Question`. Django supports all the common database relationships: many-to-one, many-to-many, and one-to-one.

##### 3. 激活你的model

>That small bit of model code gives Django a lot of information. With it, Django is able to:
>
>- Create a database schema (`CREATE TABLE` statements) for this app.
>- Create a Python database-access API for accessing `Question` and `Choice` objects.
>

first we need to tell our project that the `polls` app is installed.

- add the `'polls.apps.PollsConfig'` (see `polls/apps.py`) to `INSTALLED_APPS`
- run the command `py manage.py makemigrations polls`, you should see the output like:
>     Migrations for 'polls':
>	polls/migrations/0001_initial.py
>  		\- Create model Question
>    		\- Create model Choice

> > more about command **`makemigrations`**:
> >
> > By running **`makemigrations`**, you’re telling Django that you’ve made some changes to your models (in this case, you’ve made new ones) and that you’d like the changes to be stored as a *migration*.
> >
> > Migrations are how Django stores changes to your models (and thus your database schema) - they’re files on disk. You can read the migration for your new model if you like; it’s the file `polls/migrations/0001_initial.py`. Don’t worry, you’re not expected to read them every time Django makes one, but they’re designed to be human-editable in case you want to manually tweak how Django changes things.

To run the migrations and manage your database schema automatically - using the command [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate) first. let’s see what SQL that migration would run. The [`sqlmigrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-sqlmigrate) command takes migration names and returns their SQL:

- `py manage.py sqlmigrate polls 0001`

  > You should see something similar to the following (we’ve reformatted it for readability):
  >
  > ```sql
  > BEGIN;
  > --
  > -- Create model Question
  > --
  > CREATE TABLE "polls_question" (
  >     "id" serial NOT NULL PRIMARY KEY,
  >     "question_text" varchar(200) NOT NULL,
  >     "pub_date" timestamp with time zone NOT NULL
  > );
  > --
  > -- Create model Choice
  > --
  > CREATE TABLE "polls_choice" (
  >     "id" serial NOT NULL PRIMARY KEY,
  >     "choice_text" varchar(200) NOT NULL,
  >     "votes" integer NOT NULL,
  >     "question_id" integer NOT NULL
  > );
  > ALTER TABLE "polls_choice"
  >   ADD CONSTRAINT "polls_choice_question_id_c5b4b260_fk_polls_question_id"
  >     FOREIGN KEY ("question_id")
  >     REFERENCES "polls_question" ("id")
  >     DEFERRABLE INITIALLY DEFERRED;
  > CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");
  > 
  > COMMIT;
  > ```
  >
  > Note the following:
  >
  > - The exact output will vary depending on the database you are using. The example above is generated for PostgreSQL.
  > - Table names are automatically generated by combining the name of the app (`polls`) and the lowercase name of the model – `question` and `choice`. (You can override this behavior.)
  > - Primary keys (IDs) are added automatically. (You can override this, too.)
  > - By convention, Django appends `"_id"` to the foreign key field name. (Yes, you can override this, as well.)
  > - The foreign key relationship is made explicit by a `FOREIGN KEY` constraint. Don’t worry about the `DEFERRABLE` parts; it’s telling PostgreSQL to not enforce the foreign key until the end of the transaction.
  > - It’s tailored to the database you’re using, so database-specific field types such as `auto_increment` (MySQL), `serial` (PostgreSQL), or `integer primary key autoincrement` (SQLite) are handled for you automatically. Same goes for the quoting of field names – e.g., using double quotes or single quotes.
  > - The [`sqlmigrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-sqlmigrate) command doesn’t actually run the migration on your database - instead, it prints it to the screen so that you can see what SQL Django thinks is required. It’s useful for checking what Django is going to do or if you have database administrators who require SQL scripts for changes.
  >
  > If you’re interested, you can also run [`python manage.py check`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-check); this checks for any problems in your project without making migrations or touching the database.
  >
  > Now, run [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate) again to create those model tables in your database:
- `py manage.py migrate`

  >The [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate) command takes all the migrations that haven’t been applied (Django tracks which ones are applied using a special table in your database called `django_migrations`) and runs them against your database - essentially, synchronizing the changes you made to your models with the schema in the database.
  >
  >Migrations are very powerful and let you change your models over time, as you develop your project, without the need to delete your database or tables and make new ones - it specializes in upgrading your database live, without losing data. We’ll cover them in more depth in a later part of the tutorial, but for now, remember the three-step guide to making model changes:
  >
  >- Change your models (in `models.py`).
  >- Run [`python manage.py makemigrations`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-makemigrations) to create migrations for those changes
  >- Run [`python manage.py migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate) to apply those changes to the database.
  >
  >The reason that there are separate commands to make and apply migrations is because you’ll commit migrations to your version control system and ship them with your app; they not only make your development easier, they’re also usable by other developers and in production.
  >
  >Read the [django-admin documentation](https://docs.djangoproject.com/en/4.0/ref/django-admin/) for full information on what the `manage.py` utility can do.

##### 4. 完善model，初试api

Now, let’s hop into the interactive Python shell and play around with the free API Django gives you. To invoke the Python shell, use this command:

```powershell
py manage.py shell
```

We’re using this instead of simply typing “python”, because `manage.py` sets the [`DJANGO_SETTINGS_MODULE`](https://docs.djangoproject.com/en/4.0/topics/settings/#envvar-DJANGO_SETTINGS_MODULE) environment variable, which gives Django the Python import path to your `mysite/settings.py` file.

You can try these **database api** below:

```python
>>> from polls.models import Choice, Question  # Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
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

我们可以为类内添加\_\_str\_\_()方法来快速查看项目内容

> [Python __str__() 方法 (runoob.com)](https://www.runoob.com/note/41154)

```python
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text # Choice类同理
```

这样当我们查看内容时就能看到

>\>\>\> Question.objects.all()
>\<QuerySet [\<Question: What's up?\>]\>

当然也可以添加别的“正常的功能”，如：

```python
class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```



更多操作:

```python
>>> from polls.models import Choice, Question

# Make sure our __str__() addition worked.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django provides a rich database lookup API that's entirely driven by
# keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith='What')
<QuerySet [<Question: What's up?>]>

# Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Lookup by a primary key is the most common case, so Django provides a
# shortcut for primary-key exact lookups.
# The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Give the Question a couple of Choices. The create call constructs a new
# Choice object, does the INSERT statement, adds the choice to the set
# of available choices and returns the new Choice object. Django creates
# a set to hold the "other side" of a ForeignKey relation
# (e.g. a question's choice) which can be accessed via the API.
>>> q = Question.objects.get(pk=1)

# Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

# Create three choices.
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

# And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# The API automatically follows relationships as far as you need.
# Use double underscores to separate relationships.
# This works as many levels deep as you want; there's no limit.
# Find all Choices for any question whose pub_date is in this year
# (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```



##### 5. 试用admin端

create a admin user

```python
py manage.py createsuperuser
```

Start your server, make sure you have added the admin site to urls.py. Then follow the urls to open the admin page, you will see the login page. After that, you should see the Django admin index page, without your polls app though(if you follow the tutorial completely) 	

>Only one more thing to do: we need to tell the admin that `Question` objects have an admin interface. To do this, open the `polls/admin.py` file, and edit it to look like this:
>
>```
>from django.contrib import admin
>
>from .models import Question
>
>admin.site.register(Question)
>```

Now that we’ve registered `Question`, Django knows that it should be displayed on the admin index page.
