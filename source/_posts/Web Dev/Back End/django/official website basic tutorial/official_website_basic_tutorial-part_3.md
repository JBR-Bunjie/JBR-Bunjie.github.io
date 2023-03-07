

## 官网原教程：

### PART 3：

[Writing your first Django app, part 3 | Django documentation | Django (djangoproject.com)](https://docs.djangoproject.com/en/4.0/intro/tutorial03/#raising-a-404-error)

#### 概述：

- 编写处理http请求的函数
- 在urls中，添加视图内容
- 

#### 复述：

##### 1. 编写处理http请求的函数：

示例函数：

```python
from django.http import HttpResponse
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
# 我们也可以使用这样的方式：在html文件中编写特殊语句并通过render函数及HttpResponse返回给前端
# 需要额外引入：`from django.template import loader`
#   template = loader.get_template('polls/index.html')
#   context = {
#       'latest_question_list': latest_question_list,
#   }
#   return HttpResponse(template.render(context, request))

# 当然也可以使用shortcuts中的render函数：https://docs.djangoproject.com/zh-hans/4.0/intro/tutorial03/#a-shortcut-render
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

这些函数实际需要做的只有两件事：

- 按需求处理请求，或者抛出异常
- 返回处理结果或者错误信息

在处理的过程中，我们从数据库里读取必要信息以参与处理，以达成我们的目的。以上方代码中的`index`函数为例，我们读取了数据库中以日期为序的最近5个问题，并在后续处理中进行合并并以逗号`,`分割，最后返回一个`HttpResponse`对象来完成本次请求。

如果收到的请求无效，我们应当以适当的方式抛出错误，请注意在处理时将model层和view层相分离：

>为什么我们使用辅助函数 [`get_object_or_404()`](https://docs.djangoproject.com/zh-hans/4.0/topics/http/shortcuts/#django.shortcuts.get_object_or_404) 而不是自己捕获 [`ObjectDoesNotExist`](https://docs.djangoproject.com/zh-hans/4.0/ref/exceptions/#django.core.exceptions.ObjectDoesNotExist) 异常呢？还有，为什么模型 API 不直接抛出 [`ObjectDoesNotExist`](https://docs.djangoproject.com/zh-hans/4.0/ref/exceptions/#django.core.exceptions.ObjectDoesNotExist) 而是抛出 [`Http404`](https://docs.djangoproject.com/zh-hans/4.0/topics/http/views/#django.http.Http404) 呢？
>
>因为这样做会增加模型层和视图层的耦合性。指导 Django 设计的最重要的思想之一就是要保证松散耦合。一些受控的耦合将会被包含在 [`django.shortcuts`](https://docs.djangoproject.com/zh-hans/4.0/topics/http/shortcuts/#module-django.shortcuts) 模块中。
>
>[编写你的第一个 Django 应用，第 3 部分 | Django 文档 | Django (djangoproject.com)](https://docs.djangoproject.com/zh-hans/4.0/intro/tutorial03/#a-shortcut-get-object-or-404)

为了便于构建views，django提供了一套template系统:，可至[模板 | Django 文档 | Django (djangoproject.com)](https://docs.djangoproject.com/zh-hans/4.0/topics/templates/)查看





##### 2. 编写urls

编写好对应请求的处理函数后，自然要把它们挂载到对应的url路径上，此时便需要修改project module和当前app中的urls文件

project module内urls - 示例内容：

```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls'))
]
```

app内urls - 示例内容：

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

如果你转到 "/polls/34/" ，Django 将会运行 `detail()` 方法并且展示你在 URL 里提供的问题 ID。也就是说当某人请求你网站的某一页面时——比如"/polls/34/"，那么Django 将会根据配置项 [`ROOT_URLCONF`](https://docs.djangoproject.com/zh-hans/4.0/ref/settings/#std-setting-ROOT_URLCONF) 中的设置载入 `mysite.urls` 模块，然后寻找名为 `urlpatterns` 变量并且按序匹配正则表达式。在找到匹配项 `'polls/'`，它切掉了匹配的文本（`"polls/"`），将剩余文本——`"34/"`，发送至 'polls.urls' URLconf 做进一步处理。在这里剩余文本匹配了 `'<int:question_id>/'`，使得我们 Django 以如下形式调用 `detail()`:

```python
detail(request=<HttpRequest object>, question_id=34)
```

这样便完成了基本的前后端交互
