# Django教程

# 1.安装

我是在docker 最简洁的ubuntu镜像中开发的，所以首先需要安装python3

```shellscript
apt install python3
```



然后安装pip

```shellscript
apt install pip
```



安装django

```shellscript
pip install django
```



测试安装成功,创建一个新的python文件，如下代码运行，看打印版本号

```python
import django

if __name__=="__main__":
    print(django.__version__)
```



# 2.创建项目

通过命令创建一个django项目my\_project

```shellscript
django-admin startproject my_project
```



项目自动创建多个路径和文件

manage.py---命令行工具

/my\_project/\_\_init\_\_.py---初始化文件

/my\_project/asgi.py---用于异步编程

/my\_project/settings.py---项目配置文件

/my\_project/urls.py---项目url设置

/my\_project/wsgi.py---python服务器网关接口



项目中创建应用,进入到项目根目录,创建个index应用

```shellscript
python3 manage.py startapp index
```

在index里面，生成一些文件夹和文件

migrations文件夹---用于数据库迁移

\_\_init\_\_.py---初始化文件

admin.py---当前app的后台管理系统

apps.py---当前app的配置信息

models.py---定义映射类关联数据库

tests.py---自动化测试模块

views.py---逻辑处理模块



在完成index应用创建后，在my\_project模块根目录运行

```shellscript
 python3 manage.py runserver 80
```

这样可以启动服务，通过浏览器就可以访问服务



# 3.基本用法

## 3.1创建静态资源引用显示demo

web项目中静态资源存储于固定路径，示例代码

[https://github.com/mingtiancai/django\_learn/tree/main/static\_project](file:///workspace/AHQ8KS56BzCKekLqcfm9D/KuS8IzOE59)

核心是在项目settings.py中加上静态资源

```python
# 静态文件（CSS, JavaScript, Images）
STATIC_URL = '/static/'

# 静态文件的位置
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),  # 创建的静态文件目录
]
```



然后在app应用中，的views.py中写具体显示的html，在app路径下创建templates文件夹，在里面加入home.html,在这个html文件中引用静态图片资源。



在项目根目录下，创建static文件夹里面存储静态资源，给整个项目引用。一般在这个路径下创建的是给整个项目所有应用引用的资源，也可以在内部引用的路径下给格子应用创建静态资源。



在html中如果引用静态资源，记得加

```html
{% load static %}
```



## 3.2.视图

django里面的视图就是用来处理前端请求的逻辑。



在urls.py文件中定义相关接口对应的处理函数后，如下

```python
from django.urls import path,re_path
from . import views

urlpatterns=[
    path("",views.home),
    path("hello/",views.hello),
    re_path('(?P<year>[0-9]{4}).html',views.year,name='my_year'),
    re_path('dict/(?P<year>[0-9]{4}).htm',views.my_dict,{'month':'05'},name='my_dict'),
    path("download.html",views.download),
    path("google/",views.google),
    path("database/",views.get_data),
    path('login.html',views.login)
]
```

path或者re\_path第一个参数是接口，第二个参数是定义在views.py中的处理函数

```python
from django.shortcuts import render, redirect
from django.http import HttpResponse
import csv
from . import models


def home(request):
    return render(request, "home.html")


def hello(request):
    return HttpResponse("hello world")


def date(request, year, month, day):
    return HttpResponse(str(year) + "/" + str(month) + "/" + str(day))


def year(request, year):
    return render(request, "myyear.html")


def my_dict(request, year, month):
    return render(request, "myyear_dict.html", {"month": month})


def download(request):
    response = HttpResponse(content_type="text/csv")
    response["Content-Disposition"] = 'attachment;filename="somefilename.csv"'
    writer = csv.writer(response)
    writer.writerow(["First row", "A", "B", "C"])
    return response


def google(request):
    return render(request, "Google.html")


def get_data(request):
    print("----------------------")
    items = models.Product.objects.all()
    print(items)
    print("cookies", request.COOKIES)
    return render(request, "database.html", {"items": items})


def login(request):
    if request.method == "POST":
        name = request.POST.get("name")
        return redirect("/")
    else:
        if request.GET.get("name"):
            name = request.GET.get("name")
        else:
            name = "everyone"
    return HttpResponse("username is " + name)

```



基本上就是常规逻辑，如果返回html显示页面就返回render函数，在这个函数中把相关的html文件以及传入到html文件中的参数带上即可。当然也可以连接数据库获取数据库中表中数据返回前端显示。数据库对应的数据对象定义在models.py中，可以在views.py文件中引用。也可以在逻辑中查看request中的数据参数，自己定义一些逻辑。



## 3.3.模板

### vue做前端，django做后端demo

vue除了自定义的前端界面，其实会在后端自己也起一个server，然后在前端代码中通过http协议跟django的进程进行通信。



项目地址:

[https://github.com/mingtiancai/django\_learn/tree/main/vue\_project](file:///workspace/AHQ8KS56BzCKekLqcfm9D/e871phMkZ2)

不过这个做的有点失败，vue动不动就把我docker弄死机且这个demo前端设置的数据其实不能真正写进数据库中。我估计把后面学完就能知道怎么写进去了。



## 3.4.模型与数据库

数据库就是真实的数据库比如sqlite、postgresql、mysql等。而django里面的ORM框架，就是在程序中构建虚拟数据库，程序代码通过跟虚拟数据库操作，再由虚拟数据库操作真实数据库。这个虚拟对象数据库就是django中的模型。



示例代码地址:

[https://github.com/mingtiancai/django\_learn/tree/main/db\_project](file:///workspace/AHQ8KS56BzCKekLqcfm9D/GAMR9xuHY7)

首先要在项目的settings.py中设置数据库相关配置信息

```json
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

```



然后创建app子应用，然后再app的models.py文件中定义数据类型

```python
from django.db import models


# Create your models here.
class Type(models.Model):
    id = models.AutoField(primary_key=True)
    type_name = models.CharField(max_length=20)


class Product(models.Model):
    id = models.AutoField(primary_key=True)
    name = models.CharField(max_length=50)
    weight = models.CharField(max_length=20)
    size = models.CharField(max_length=20)
    type = models.ForeignKey(Type, on_delete=models.CASCADE)

```

这些数据类型可以通过django的工具去产生数据库的表，同时在python程序中也就是django的views.py文件中可以引用这些类结构的相关方法，对数据库进行增删改查。



在定义模型后，生成迁移文件

```shellscript
python manage.py makemigrations
```

接着生成真正的数据库表

```shellscript
python manage.py migrate
```

这样就可以生成真正的数据库，里面除了我们自定义的数据表，还有一些django框架生成的表。django框架对sql语句进行了封装，可以不直接使用底层的sql语句写业务逻辑。而是直接调用框架相关的方法即可，都是增删改查的。



## 3.5.表单

表单就是在前端界面设置的参数数据，通过表单，传递到后端进行解析。我自己写的demo里面是通过在前端注册一个用户，然后把用户的数据写进数据库里面存储。

代码地址:

[https://github.com/mingtiancai/django\_learn/tree/main/form\_project](file:///workspace/AHQ8KS56BzCKekLqcfm9D/7DB1BFOmZd)

之前用vue写的前端没设置成数据库数据，这次可以了。都是第一次使用，不容易啊。

django的表单有一些内置格式，比如邮箱、日期之类的。可以比较方便直接调用。



## 3.6.admin后台

就是网站的后台管理，可以直接进入到后台设置相关信息。可以通过django的命令设置超级用户

```shellscript
python manage.py createsuperuser
```

按照提示设置参数即可

这种admin后台就类似于wifi后台，进入里面之后可以设置各种参数。



## 3.7.Auth认证系统

就是网站常规注册、用户权限管理等功能。django在创建数据库的时候除了会创建用户自定义的数据库表之外，还会创建一些额外的表，这里面就有用于用户相关的。比如auth\_user、auth\_permission、auth\_group等。如果你需要设置相关功能，就可以自己写逻辑进去去做增删改查。另外如果django提供的这些表的字段不足以满足现实需求，可以自定义去做拓展。就是自定义数据库表，然后配置到配置文件中就行。然后在业务逻辑中引用就行了。



## 3.8.其他

session对话控制，用于维持会话信息

cache缓存，对于相同的输入，可以直接调用缓存来提高性能。包含分布式缓存，数据库缓存，文件系统缓存，本地内存缓存，虚拟缓存

CSRF防护，这是为了应对一些网络攻击。当表单访问网页的时候，会在后端生成一个随机数，当用户提交表单的时候，后台会验证这个随机数客户端和服务端是否一致。以此来判断表单是否合法



# 4.总结

整体感觉这个框架不是很难，不过如果想弄得很熟练，学习这种框架的细节还是需要两三周时间才能比较熟练。我几乎从来不写web的，只不过最近在做ai agent，这个领域几乎所有的开源框架在软件体系上都是采用web技术栈的方式。这迫使我不得不把web这块恶补一下。先找个可以快速入门且功能还算完善的web后端框架学习一下，接下来就去学一下前端，估计也就一两天配合AI辅助编程，就可以写出能看的一些前后端应用了。这样再去看和改一些开源得AI Agent源码就比较得心应手了。毕竟我自己如果当CTO，自己一点不懂web也很影响技术选型，不能让自己有技术短板，否则一定会影响战略决策的准确性。



坦率来讲，web的信息非常多，我也的确不是专家，仅仅自己写个留个成长记录吧，我的核心焦点还是会集中到AI上，只是抽一周时间扫盲一下。











