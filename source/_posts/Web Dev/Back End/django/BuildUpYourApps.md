> remeber: we always do `makemigrations` and `migrate` when we change the models

### 



### Step 1. create an app

```powershell
django-amdin startapp <YourAppName>
```

Then we got an essential app with the structure like below:

> \- *\<YourAppName\>*:
>   \- *migrations*
>   \- \_\_init\_\_.py
>   \- admin.py
>   \- apps.py
>   \- models.py
>   \- tests.py
>   \- views.py



### Step 2. change your models and save the changes

obviously, if we want a good backend, we need a database. And in that case, we need to write a good model file







### 



### sideshow: Use The Python Shell to Control Django

use command `python manage.py shell` to enter the special python shell

for example: 

we have a model in an app named product like this:

```python
>>> from products.models import product as pd
>>> pd.objects.all()  # check all the data related with class product, using the method __str__
# add:
>>> pd.objects.create(<pd_attr1>="...", [...])  # add a new data into the table
# delete
>>> pd.objects.
# change
>>>
# search
>>>
```



if we want to change the fields of vars in models, we must make sure that the data is fixable in new field:
In certain time, we need to change the data:

of course you can change it in software like sqlitestudio, but how about with shell?

