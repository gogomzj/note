# Django

### 创建基础Django项目

```python
#在pycharm中可直接选Django项目创建
#和flask的app意思相同
django-admin startapp one('one为文件夹名')
```

##### 创建one/views.py中添加

```python
from django.shortcuts import render
from django.http import HttpResponse

def index(request):
    return HttpResponse('Hello Django')
```

##### 在one/urls.py中添加 （使得每个app管理自己的路由）

```python
from django.urls import path
from .views import index
from django.conf.urls import url

urlpatterns = [
    url('index',index)
]
```

##### 在总文件夹的urls.py中

```python
from django.contrib import admin  #系统添加
from django.urls import path  #系统添加
from one.views import index
from django.conf.urls import url,include

urlpatterns = [
    path('admin/', admin.site.urls),  #系统添加
    path('2',include('one.urls')),

]

#最后页面访问路由为172.0.0.1:8000/2/index

#flask中的路由是写在视图函数前，而Django的路由写在urls.py中
```

### Django的GET方式传参

##### 在one/urls.py中

```python
rom django.urls import path
from .views import index,add,sub
from django.conf.urls import url

urlpatterns = [
    url('index',index),
    url('^add/([0-9]+)/([0-9]+)$',add),
    #172.0.0.1:8000/one/2/6
    url('^sub$',sub)
	#172.0.0.1:8000/one/sub?a=5&b=6
]

```

##### 在one/views.py中

```python
def add(request,a,b):
    return HttpResponse(str(int(a) + int(b)))

def sub(request):
    a = request.GET['a']
    b = request.GET['b']
    return HttpResponse(str(int(a) + int(b)))
```

```python
'在Django中传参需要用字典，而且必须为字符串！！！
```

### 模板    (html的使用)

```python
'返回前端的数值必须为字典
'记得在settings.py中添加app
'在HTML中需要传值的地方为 {{}}
'从py文件向html文件传值时必须用字典！
```

### 模板标签(jinja2)

##### if_else判断

```jinja2
if/else 标签
基本语法格式如下：
{% if condition %}
     ... 
{% endif %} 或者：
{% if condition1 %}
   ... 
{% elif condition2 %}
   ... 
{% else %}
   ... 
{% endif %}
    if/else 支持嵌套
    if/else 标签支持and、or 或者 not 关键字来对多个变量做判断
{% if condition1 and condition2 %}
     ...
{% endif %} 
```

##### for循环

```jinja2
#基本语法
{% for X in Y %} 
    ...
{% endfor %} 
#遍历列表
{% for e in listValue %}
    {{ e }}
{% endfor %}
#反向遍历列表
{% for e in listValue reversed %}
    ...
{% endfor %}
#嵌套使用
{% for e in listValue %}
    {{ e.name }}
    {% for s in e.listValue %}
       {{ s }}
    {% endfor %}
{% endfor %} 
```

##### 比较user和current user是否相等

````jinja2
{% ifequal user currentuser %}
    ...   
{% ifequal section 'sitenews' %}
    ...
{% else %}
    ...
{% endifequal %} 
````

##### 注释

```jinja2
{# 这是一个注释 #}
```

##### include

```jinja2
#下面这个例子都包含了 nav.html 模板：
{% include "nav.html" %}
```

### 模板继承

```jinja2
{% extends '想要继承的html' %}
{% block 自定义的块名%}
{% block super %}  #想继承base里的块内容
{% endblock %}
```

### 静态文件(css,js)

```python
'必须要写在static文件中，和templates在一级
'修改HTML中静态文件地址
```

### 模板函数

被@register.simple_tag装饰的为模板函数(函数参数不限制)

被@register.filter(过滤器)装饰的也是装饰器(函数参数不能超过两个)

##### 创建模板函数，在templatetags文件中创建py文件

！必须为templatetags文件！

```python
from django import template

register =template.Library()
@register.simple_tag
def add(a,b):	
	return a+b
	
@register.filter
def sub(a,b):
	return a-b
```

##### 在模板中调用模板函数

```jinja2
a.在模板的最开头加载test模块
	{% load py文件名 %}
b.调用add函数
	{% add 2 3 %}
c.调用sub函数
	{{ 1|sub:2 }}  #1-2
4.把filter当作if条件
    {% if 5|sub:3 == 2 %}    #5-3
    xxx
    {% endif %}
```

### Django数据模型

##### 修改Django数据库配置(settings.py)

```python
DATABASES = {
'default': {
'ENGINE': 'django.db.backends.mysql',
'NAME': 'embsky', #数据库名字
'USER': 'root',
'PASSWORD': '1234563',
'HOST': '127.0.0.1',
'PORT': '3306',
}
}
```

##### 创建数据模型(在one/models.py)

```python
class Student(models.Model) :
    #成员名字=类型
    name = models.CharField(max_length=32)
    age = models.IntegerField()
    salary = models.FloatField()
    password = models.CharField(max_length=256)
     #修改对象的时间
     #Teacher对象也就是数据库中的记录修改的时间
    mt = models.DateTimeField(auto_now=True)
    #创建对象的时间
    #Teacher对象创建并且存放进数据库的时间
    ct =models.DateTimeField(auto_now_add=True) 
   
    #可以直接显示return里的内容
    def __str__(self):
    	return str(self.name + " " + str(self.age) + " " + str(self.salary));
```

##### 更新数据库

```shell
 python manage.py makemigrations
 
 python manage.py migrate
```

##### 数据库数据的增加

```shell
创建数据一：
>>> s = Student(name='zhangsan',age='25')
>>> s.save()
创建数据二：
>>> s1 = Student()
>>> s1.name='wangwu'
>>> s1.age=45
>>> s1.save()
创建数据三：
>>> s2 = Student.objects.create(name='lisi', age=35)
创建数据四：如果name和age有一个不匹配那么就创建一个新的
>>> l =Student.objects.get_or_create(name='123', age=13)
```

##### 数据库数据的查询

```shell
查询数据一：查询所有的数据，返回列表
>>> s = Student.objects.all()
查询数据二：
>>> s = Student.objects.all()[1:3]
查询数据三：
>>> s = Student.objects.get(name='zhangsan')
查询数据四：模糊匹配
>>> s = Student.objects.filter(name='zhangsan')
>>> s = Student.objects.filter(name__iexact='zhangsan')

#不区分大小写
>>> s = Student.objects.filter(name__contains='12')
>>> s = Student.objects.filter(name__icontains='12')

#包含忽略大小写
>>> s = Student.objects.filter(name__startswith='12')
>>> s = Student.objects.filter(name__istartswith='12')

#忽略大小写
>>> s = Student.objects.filter(name__endswith='12')
>>> s = Student.objects.filter(name__iendswith='12') 

#忽略大小写
>>> s = Student.objects.filter(name__isnull=True) #为空
#id在1-20之间的
>>> s = Student.objects.filter(id__range=(1,20)) 
#时间范围
>>> s = Student.objects.filter(ct__range=(datetime(2017,10,22,0,0), datetime(2018,10,22,0.0)))

>>> s = Student.objects.filter(id__in=[1,2,3]) #id在列表中出现的
>>> s = Student.objects.filter(name__regex='^[a-z]+$') #正则
>>> Student.objects.filter(id__lt=3) #小于
>>> Student.objects.filter(id__lte=3) #小于等于
>>> Student.objects.filter(id__gt=3) #大于
>>> Student.objects.filter(id__gte=3) #大于等于
查询数据五：exclude反查询
>>> s = Student.objects.exclude(name__regex='^[a-z]+$')
查询数据六：排序
>>> s = Student.objects.order_by("name") #升序
>>> s = Student.objects.order_by("-name") #降序
查询数据七：filter、order_by和exclude连用
>>> s = Student.objects.filter(name__regex='^[a-z]+$').exclude
(name='zhangsan').exclude(name='lisi').filter(name='wangwu')
>>> s = Student.objects.filter(name__regex='^[a-z]+$').exclude
(name='zhangsan').order_by('-name')
```

##### 数据库数据的修改

```shell
修改数据 一：在查询结果后+update（只能是queryset）
>>> s = Student.objects.filter(name__regex='^[a-z]+$').exclude
(name='zhangsan').order_by('-name').update(name='hao',age=30)
或者：
>>> s = Student.objects.filter(name__regex='^[a-z]+$').exclude
(name='zhangsan').order_by('-name')
>>> s.update(name='hao',age=30)
修改数据二：通过对象修改
>>> s = Student.objects.get(name='12')
>>> s.age=100
>>> s.save()
```

##### 数据库数据的删除

```shell
删除数据一：通过对象删除
>>> s = Student.objects.get(name='12')
>>> s.delete()
>>> s = Student.objects.filter(id__lt=3)
>>> s.delete()
或者：
>>> 
s = Student.objects.filter(id__lt=3).delete()
```

### 数据库中反向关系的创立与外键的书写

在models.py中添加

```python
class Student(models.Model) :
    name = models.CharField(max_length=128)
    age = models.IntegerField()
    salary = models.FloatField()
    #外键及反向关系，访问teacher获得Teacher对象
    #Teacher中可以访问student_set.all()得到QuerySet（该老师的所有学生）
    teacher = models.ForeignKey(Teacher,
    on_delete=models.CASCADE)
    # CASCADE级联删除，老师被删除学生也被删除
   	 # SET_NULL老师删除学生的老师位置为NULL
    	# DO_NOTHING老师被删除，学生内容不变
```

### 表单

```python
'POST方式提交表单必须在HTML的表单中添加令牌
'{{ csrf_tocken}}
method='post'
action=" " #为空的话返回的是当前路由
```

在路由中

```python
def result(request) :
    if request.method == 'POST' :
        d = dict()
        d['name'] = request.POST['name']
        d['age'] = request.POST['age']
        return render(request, 'result.html', d)
    
   '一般为GET方式获取表单，POST方式提交表单
```

### 后端表单

##### 表单创建(forms.py)

```python
from django.forms import Form
from django import forms
from django.core.exceptions import ValidationError
import re

class RegisterForm(Form) :
    
    def is_telephone(value) : #value传入的是用户填写的信息
        mobile_re = re.compile(r'^(13[0-9]|15[012356789]|17[678]|18[0-9]|16[69])[0-9]{8}$')
        if not mobile_re.match(value):
            raise ValidationError('手机号有误')

    telephone = forms.CharField(
        required=True,  #必须填写
        max_length=11,
        min_length=11,
        validators=[
            is_telephone,
        ]
    )

    name = forms.CharField(
        required=True,
        min_length=2,
        max_length=32,
        error_messages={  #错误提示
            'required':'姓名必须填写',
            'min_length':'名字必须大于2个字符',
            'max_length':'名字必须小于32个字符',
        }
    )

    email = forms.EmailField(
        required=True,
        error_messages={
            'invalid':'邮箱格式有误'
        },
        widget=forms.EmailInput(
            attrs={  #表单样式
                'class':'c1',
                'placeholder':'邮箱',
            }
        )
    )

    age = forms.IntegerField(
        required=True,
        max_value=200,
        min_value=75,
        error_messages={
            'max_value':'看来科技发达了',
            'min_value':'不能拉后腿',
        }
    )

    sex = forms.ChoiceField(  #多选
        widget=forms.Select(),
        choices=(
            (1, '男'),
            (2, '女'),
            (3, '保密')
        ),
        initial=(1, '男')
    )
```

##### 表单验证

```python
def register(request) :
    if request.method == 'POST' :
        form = RegisterForm(request.POST)
        if form.is_valid() :
            telephone = form.cleaned_data['telephone']
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            age = form.cleaned_data['age']
            sex = form.cleaned_data['sex']
            print(telephone, name, email, age, sex)
            return redirect(reverse('one_index'))
        else :
            return render(request, 'one/register.html', {'form':RegisterForm(request.POST), 'errors':form.errors}) #request.POST可以将用户填写的信息显示到新页面上
    else :
        return render(request, 'one/register.html', {'form':RegisterForm()})
```

### 页面渲染与重定向

```python
return render(request, 'result.html')
#页面渲染，即将数据显示到页面上
#渲染时若传参必须使用字典！
return redirece(reverse('urls里的路由名'))
#重定向，即页面跳转，url中填的是urls中自定义的name
```

### 内置admin的使用即内置管理后台的创建

##### 在控制台添加管理员

```shell
python  manage.py  creatsuperuser
```

##### 在admin页面显示app中数据(admin.py)

```python
#查看老师的所有学生
class StudentStyle(admin.TabularInline) :
    model = Student
    fields = ['id', 'name', 'sex', 'email', 'age']

@admin.register(Teacher)
class TeacherAdmin(admin.ModelAdmin) :
    list_display = ['id', 'name', 'sex']
    search_fields = ['name']  # 可通过name进行搜索
    inlines = [StudentStyle]
```

​	超级用户拥有所有权限

##### 	可在127.0.0.1:8000/admin中使用Django内置后台

### 用户管理

创建app不能和Django默认app重复(可在数据库中查询)

数据库中表名的建立：app名_models.py中的名字

##### 用户注册(views.py)

```python
from django.contrib.auth.models import User
from django.contrib.auth import login, logout, authenticate
from django.contrib.auth.decorators import login_required

def user_register(requeset) :
    if requeset.method == 'GET' :
        return render(requeset, 'user/user_register.html')
    else :
        username = requeset.POST['name']
        email = requeset.POST['email']
        password = requeset.POST['password']
        password_again = requeset.POST['password_again']
        #print(username, email, password, password_again)
        user = User(username=username, email=email)
        user.set_password(password)  #密码存储，使用SHA256加密方法
        user.is_active = True #是否为用户
        #user.is_staff = True #是否为普通管理员
        user.save()  
		#在注册完跳转的登录页面中显示注册时的用户名
        requeset.session['username'] = username
        return redirect(reverse('user_user_login'))
```

#### cookie与session

##### session 

浏览器访问服务器的时候回建立一个session，在session服务器可以保存状态（比如用户状态）

##### cookie

 保存在浏览器中 服务器把session加密后放到了cookie

```python
def index(request) :
    print(request.COOKIES)  #打印cookie
    return HttpResponse('ok')

def set_cookie(request) :
    res = HttpResponse('ok')
    res.set_cookie('test', 'this is my cookie')  #添加自己的cookie
    return res
```

##### 用户登录(views.py)

````python
def user_login(request) :
    if request.method == 'GET' :
        username = request.session.get('username')
        return render(request, 'user/user_login.html', {'username':username})
    else :
        username = request.POST['name']
        password = request.POST['password']
        user = authenticate(request, username=username, password=password)
        if user :
            login(request, user) #User ---->auth_user  request.user=user
            #next表示上一个用户访问的页面
            next = request.GET.get('next')
            if next :
                print(next)
                return redirect(next)
            return redirect(reverse('user_user_info'))
        return redirect(reverse('user_user_login'))
````

##### 用户注销(views.py)

````python
@login_required(login_url='/user/login') 
#该路由表示登录用户才能访问否则直接跳转到login
def user_logout(request) :
    logout(request)
    return redirect(reverse('user_user_login'))
````

##### html

```jinja2
<ul class="nav navbar-nav navbar-right">
    
    判断是否为管理员,状态都存放在request中可以直接调用
    {% if request.user.is_superuser %}
        <li><a href="/admin">管理中心</a></li>
    {% endif %}
    
    判断是否为用户是否登录
    {% if request.user.is_authenticated %}
        <li><a href="{% url 'user_user_info' %}">
            {{ request.user.username }}
        </a></li>
    {% endif %}
    {% if not request.user.is_authenticated %}
        <li><a href="{% url 'user_user_login' %}">登陆</a></li>
        <li><a href="{% url 'user_user_register' %}">注册</a></li>
    {% else %}
        <li><a href="{% url 'user_user_logout' %}">注销</a></li>
    {% endif %}
</ul>
```

##### 用户登录与登出

```python
#登录用户
from django.contrib.auth import login
login(request, user)
#登出用户
@login_required(login_url='/user/login')
def user_logout(request) :
    logout(request)
    return redirect(reverse('user_user_login'))
```

##### 用户功能扩展

```python
from django.db import models
from django.contrib.auth.models import User #django自带的User

class UserInfo(models.Model) :
    age = models.IntegerField()
    sex = models.IntegerField()
	user = models.ForeignKey(User,related_name='info',on_delete=models.CASCADE)
```

### 邮件

修改配置文件(settings.py)

```python
EMAIL_USE_SSL = True
EMAIL_HOST = 'smtp.163.com' # 如果是 163 改成 smtp.163.com
EMAIL_PORT = 465
EMAIL_HOST_USER = '发送方邮箱' # 帐号
EMAIL_HOST_PASSWORD = 'xxxx' # 密码或授权码
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER
```

##### 普通邮件发送

```python
from django.core.mail import send_mail, send_mass_mail,EmailMultiAlternatives
from django.http import HttpResponse

def sendmail(request):
    subject = '邮件主题'
    content = '邮件内容'
    sender = '邮件发送方'
    recvers = ['邮件接收收方']
    send_mail(subject=subject, message=content, from_email=sender, recipient_list=recvers)
    return HttpResponse('ok')
```

##### 一次发送多封邮件

```python
from django.core.mail import send_mail, send_mass_mail,EmailMultiAlternatives
from django.http import HttpResponse

def sendmail1(request):
    m1 = ('111111', '222222', '邮件发送方', ['邮件接收方1', ])
    m2 = ('222222', '333333', '邮件发送方', ['邮件接收方2', ])
    send_mass_mail((m1, m2))
    return HttpResponse('ok')
```

##### 发送HTML邮件

````python
from django.core.mail import send_mail, send_mass_mail,EmailMultiAlternatives
from django.http import HttpResponse

def sendmail2(request):
    content = '''
        <h1>大标题</h1>
        <h2>次大标题</h2>
        <p>这里是段落</p>
    '''
    msg = EmailMultiAlternatives(subject='页面邮件',
                                 body=content,
                                 from_email='邮件发送方',
                                 to=['邮件接收方'])
    msg.content_subtype = 'html'
    msg.send()
    return HttpResponse('ok')
````

### 用户权限

在数据库auth_permission中可查看所有权限

权限中的content_type_id可以在django_content_type中查看

在auth_user_user_permissions中记录用户所拥有的权限

##### 自定义权限

```python
class Student(models.Model) :
	name = models.CharField(max_length=128)
	age = models.IntegerField()
class Meta :
	permissions = (('edit_student_name', 'can edit student name'),
		('edit_student_age', 'can edit student age'))
```

##### 增加用户权限

```shell
user.user_permissions.add(perm)
```

##### 删除用户权限

```shell
 user.user_permissions.remove(perm)
```

##### 查询权限

````shell
 ps = user.user_permissions.values()
 			或
 ps = user.user_permissions.all()
````

##### 判断用户是否拥有某个权限(装饰器)

````python
@permission_required('student.view_student', login_url='/student/error403')
def students(request) :
    # 或得多有学生
    students = Student.objects.all()
    return render(request, 'student/students.html', {'students':students})
````

### django中间件

在放置配置文件的文件夹中创建middleware文件将想添加的py中间件放在middleware中

![中间件](C:\Users\admin\Desktop\screen capture\Django\中间件.png)

##### 中间件的书写

```python
from django.utils.deprecation import MiddlewareMixin
from django.http import HttpResponse

class MW1(MiddlewareMixin) :
    def process_request(self, request):
        print('MW1 request')
        request.session['hello'] = 'world'
    def process_response(self, request, response):
        print('MW1 response')
        return response
        #return HttpResponse('sorry')

    def process_view(self, request, callback, callback_args, callback_kwargs) :
        print('MW1 view')

class MW2(MiddlewareMixin) :
    def process_request(self, request):
        print('MW2 request')
        #return None
        #return HttpResponse('mw2 error')
    def process_response(self, request, response):
        print('MW2 response')
        return response
    def process_view(self, request, callback, callback_args, callback_kwargs) :
        print('MW2  view')

class MW3(MiddlewareMixin) :
    def process_request(self, request):
        print('MW3 request')
        # if request.path == '/hello' :
        #     return HttpResponse('hello not found')
        # if request.META['REMOTE_ADDR'] == '127.0.0.1' :
        #     return HttpResponse('127.0.0.1 not allow')
        if not request.user.is_authenticated :
            return HttpResponse('please login')
    def process_response(self, request, response):
        print('MW3 response')
        return response
    def process_view(self, request, callback, callback_args, callback_kwargs) :
        print('MW3 view')
```

##### 注册中间件(settings.py)

```python 
MIDDLEWARE = [
    'StudyMiddleWare.middleware.py文件名.MW1', 
    'StudyMiddleWare.middleware.py文件名.MW2',
    'StudyMiddleWare.middleware.py文件名.MW3',
]
```

##### 中间件的作用

过滤URL
做IP限制
做缓存
其他(比如截获未登录/未激活的用户)

### celery+Redis异步框架

pip安装

```linux
celery 

redis==2.10.6 #指定版本

celery-with-redis

```

创建简单例子(新建普通工程)

新建tasks.py

````python
from celery import Celery

app=Celery('tasks',backend='redis://localhost:6379/0',broker='redis://localhost:6379/0')

@app.task #只有添加装饰器才能算是异步
def add(no):
	for i in range(no) :
	print('hello celery')
````



















