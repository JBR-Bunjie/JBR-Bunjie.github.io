

## 官网原教程：

### PART 1：

[Writing your first Django app, part 1 | Django documentation | Django (djangoproject.com)](https://docs.djangoproject.com/en/4.0/intro/tutorial01/)

#### 概述：

- 概述教程内容
- 使用`startproject`命令创建的项目的基本内容介绍
- 通过`mange.py`启动刚刚创建的新项目
- 通过`startapp`命令新建应用，并对`project`与`app`作区分
- 介绍url与view

#### 复述：

##### 1. quick the installation and version of django

```python
python -m django --version
```

If Django is installed, you should see the version of your installation. If it isn’t, you’ll get an error telling “No module named django”.

##### 2. creating a project

Use the command line to auto-generate some code that establishes a Django [project](https://docs.djangoproject.com/en/4.0/glossary/#term-project) – a collection of settings for an instance of Django, including database configuration, Django-specific options and application-specific settings to finish some initial setup.

```powershell
django-admin startproject <mysite>
```

This will create a `mysite` directory in your current directory.

>And the folder created by startproject looks like:
>
>>mysite/
>>manage.py
>>mysite/
>>   __init__.py
>>   settings.py
>>   urls.py
>>   asgi.py
>>   wsgi.py
>
>These files are:
>
>- The outer `mysite/` root directory is a container for your project. Its name doesn’t matter to Django; you can rename it to anything you like.
>- `manage.py`: A command-line utility that lets you interact with this Django project in various ways. You can read all the details about `manage.py` in [django-admin and manage.py](https://docs.djangoproject.com/en/4.0/ref/django-admin/).
>- The inner `mysite/` directory is the actual Python package for your project. Its name is the Python package name you’ll need to use to import anything inside it (e.g. `mysite.urls`).
>- `mysite/__init__.py`: An empty file that tells Python that this directory should be considered a Python package. If you’re a Python beginner, read [more about packages](https://docs.python.org/3/tutorial/modules.html#tut-packages) in the official Python docs.
>- `mysite/settings.py`: Settings/configuration for this Django project. [Django settings](https://docs.djangoproject.com/en/4.0/topics/settings/) will tell you all about how settings work.
>- `mysite/urls.py`: The URL declarations for this Django project; a “table of contents” of your Django-powered site. You can read more about URLs in [URL dispatcher](https://docs.djangoproject.com/en/4.0/topics/http/urls/).
>- `mysite/asgi.py`: An entry-point for ASGI-compatible web servers to serve your project. See [How to deploy with ASGI](https://docs.djangoproject.com/en/4.0/howto/deployment/asgi/) for more details.
>- `mysite/wsgi.py`: An entry-point for WSGI-compatible web servers to serve your project. See [How to deploy with WSGI](https://docs.djangoproject.com/en/4.0/howto/deployment/wsgi/) for more details.

##### 3. start your project

```powershell
python manage.py runserver
# you can pass server's port and so on as a command-line argument behind 'runserver'
```

You'll see the output which includes:

> ...
> Starting development server at http://127.0.0.1:8000/
> Quit the server with CONTROL-C.

##### 4. create new app

> What’s the difference between a project and an app? An app is a web application that does something – e.g., a blog system, a database of public records or a small poll app. A project is a collection of configuration and apps for a particular website. A project can contain multiple apps. An app can be in multiple projects.

```powershell
py manage.py startapp polls
```

That will create a new directory like:

>polls/
>    __init__.py
>    admin.py
>    apps.py
>    migrations/
>        __init__.py
>    models.py
>    tests.py
>    views.py

This directory structure will house the poll application.

##### 5. write your own view and set it url

Open the file `polls/views.py` and put the following Python code in it:

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

This is the simplest view possible in Django. To call the view, we need to map it to a URL - and for this we need a URLconf.

To create a URLconf in the polls directory, create a file called `urls.py`. Your app directory should now look like:

>polls/
>    __init__.py
>    admin.py
>    apps.py
>    migrations/
>        __init__.py
>    models.py
>    tests.py
>    urls.py
>    views.py

In the `polls/urls.py` file include the following code:

```python
from django.urls import path
from . import views

urlpatterns = [path('', views.index, name='index'),]
```

The next step is to **point the root URLconf at the `polls.urls` module.** In `mysite/urls.py`, add an import for `django.urls.include` and insert an [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include) in the `urlpatterns` list, so you have:

```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

The [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include) function allows referencing other URLconfs. Whenever Django encounters [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include), it chops off whatever part of the URL matched up to that point and sends the remaining string to the included URLconf for further processing. See more about -> [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include)

You should always use `include()` when you include other URL patterns. `admin.site.urls` is the only exception to this.

Now restart your server and go to http://localhost:8000/polls/ in your browser, and you should see the text “*Hello, world. You’re at the polls index.*”, which you defined in the `index` view.

See more about `path()`

>[`path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path) argument: `route`[¶](https://docs.djangoproject.com/en/4.0/intro/tutorial01/#path-argument-route)
>
>`route` is a string that contains a URL pattern. When processing a request, Django starts at the first pattern in `urlpatterns` and makes its way down the list, comparing the requested URL against each pattern until it finds one that matches.
>
>Patterns don’t search GET and POST parameters, or the domain name. For example, in a request to `https://www.example.com/myapp/`, the URLconf will look for `myapp/`. In a request to `https://www.example.com/myapp/?page=3`, the URLconf will also look for `myapp/`.

>[`path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path) argument: `view`[¶](https://docs.djangoproject.com/en/4.0/intro/tutorial01/#path-argument-view)
>
>When Django finds a matching pattern, it calls the specified view function with an [`HttpRequest`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest) object as the first argument and any “captured” values from the route as keyword arguments. We’ll give an example of this in a bit.

>[`path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path) argument: `kwargs`[¶](https://docs.djangoproject.com/en/4.0/intro/tutorial01/#path-argument-kwargs)
>
>Arbitrary keyword arguments can be passed in a dictionary to the target view. We aren’t going to use this feature of Django in the tutorial.

>[`path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path) argument: `name`[¶](https://docs.djangoproject.com/en/4.0/intro/tutorial01/#path-argument-name)
>
>Naming your URL lets you refer to it unambiguously from elsewhere in Django, especially from within templates. This powerful feature allows you to make global changes to the URL patterns of your project while only touching a single file.
>
>When you’re comfortable with the basic request and response flow, read [part 2 of this tutorial](https://docs.djangoproject.com/en/4.0/intro/tutorial02/) to start working with the database.



- 