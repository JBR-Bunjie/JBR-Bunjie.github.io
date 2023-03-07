# Django Model

## 参考文档

- [模型 | Django 文档 | Django (djangoproject.com)](https://docs.djangoproject.com/zh-hans/4.1/topics/db/models/)
- [详解Django的Models（django基础四）_Charles-Su的博客-CSDN博客_django model](https://blog.csdn.net/happygjcd/article/details/102649947)
- 



## 内容：

### Model在项目中的意义

模型准确且唯一的描述了数据。它包含储存数据的重要字段和行为。

一般来说，每一个模型都映射一张数据库表。

基础：

- 每个模型都是一个 Python 的类，这些类继承 [`django.db.models.Model`](https://docs.djangoproject.com/zh-hans/4.1/ref/models/instances/#django.db.models.Model)
- 模型类的每个属性都相当于一个数据库的字段。
- 利用这些，Django 提供了一个自动生成访问数据库的 API；请参阅 [执行查询](https://docs.djangoproject.com/zh-hans/4.1/topics/db/queries/)。



### Model的构建

#### 继承来源：

我们可以继承 `models.Model` ,  `AbstractUser` 等来构建我们的新Model类

其中

##### models.Model:

Models则是通用的模型类，自定义模型都需要继承这个



##### AbstractUser:

AbstractUser要记得在setting.py里面加上AUTH_USER_MODEL = ‘users.UserProfile’

AbstractUser是内置的用户类，当要继承内置的用户模型并进行扩展时，就使用它





#### Inner Class -- Meta：

Model metadata is “anything that’s not a field”, such as ordering options ([`ordering`](https://docs.djangoproject.com/en/4.1/ref/models/options/#django.db.models.Options.ordering)), database table name ([`db_table`](https://docs.djangoproject.com/en/4.1/ref/models/options/#django.db.models.Options.db_table)), or human-readable singular and plural names ([`verbose_name`](https://docs.djangoproject.com/en/4.1/ref/models/options/#django.db.models.Options.verbose_name) and [`verbose_name_plural`](https://docs.djangoproject.com/en/4.1/ref/models/options/#django.db.models.Options.verbose_name_plural)). None are required, and adding `class Meta` to a model is completely optional.

A complete list of all possible `Meta` options can be found in the [model option reference](https://docs.djangoproject.com/en/4.1/ref/models/options/).



#### 基本属性与字段：

##### 1.field类型

AutoField:一个自动递增的整形字段，通常用于主键

CharField：字符串字段，用于输入较短的字符，对应与HTML里面\<input type='text'\>

TextField：文本字段，用于输入较多的字符，对应html标签 \<input type = "textarea"\>；

EmailField：邮箱字段，用于输入带有Email格式的字符

DateFiled

TimeFiled

DateTimeField：日期字段，支持时间输入

ImageField：用于上传图片并验证图片合法性，需定义upload_to参数，使用本字段需安装python pillow等图片库

IntegerField：整数字段，用于保持整数信息

##### field属性

primary_key：设置True or False，定义此字段是否为主键

default：设置默认值，可以设置默认的文本、时间、图片、时间等

null：设置True or False，是否允许数据库字段为Null，默认为False

blank：设置True or False，定义是否运行用户不输入，默认为False；若为True，则用户可以不输入此字段

max_length：设置默认长度，一般在CharField、TextField、EmailField等文本字段设置

verbose_name：设置该字段的名称，所有字段都可以设置，在Web页面会显示出来（例如将英文显示为中文）

choices：设置该字段的可选值，本字段的值是一个二维元素的元祖；元素的第1个值为实际存储的值，第2个值为HTML页面显示的值

upload_to：设置上传路径，ImageField和FileField字段需要设置此参数,如果路径不存在，会自动创建

##### Meta类属性

verbose_name：设置对象名称（例如usecms），若没有设置，则默认为该类名的小写分词形式，例如类名为CamelCase会被转换为camel case；

verbose_name_plural：设置对象名称复数（例如usercms），一般设置跟verbose_name一样，

verbose_name_plural=verbose_name否则会默认加s；

db_table：设置映射的数据表名，默认为“应用名_模型名”，即用该模型所在app的名称加本模型类的名称

proxy：设置True or False，设置本模型及所有继承本模型的子模型是否为代理模型；

abstract：设置True or False，设置本模型类是否为抽象基类；如果是抽象基类，那么是不会创建这张表的，这张表用来作为基类被其他的表继承

##### model层的命令详解

python manage.py makemigrations+名字:# 生成数据库表的初始化文件initial.py文件

python manage.py migrate# # 基于数据库表初始化文件initial.py文件，正式生成数据表

python manage.py sqlmigrate polls 0001查看数据库的生成语句，因为initial.0001是数据库表的初始化文件



#### 基本设计思路：

