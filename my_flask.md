# Flask

### 创建基础Flask项目

一个项目一般只包含一个app，一个app中包含多个蓝本(每个a蓝本都是一个包)，不同的功能可以写在不同蓝本里，这样可以使代码更好的维护，建议不使用太多app

flask包含：run.py ，config.py(配置文件)和app (app为py文件夹，必须含有__init__.py)

一个app可以包含多个蓝本，根据实际需求吧功能分配到不同的蓝本中

每个蓝本都包含：views.py,forms.py，models.py等

### 模板继承

- 不完整的HTML页面
- 模版引擎Jinja2（render_template)
- 模版变量{{var_name}} 
- 模版标签{% %}
- 模版继承（基模版， 一改全改）
- 模版宏定义（替换）

### MVC

- M 数据模型
- C 控制器（Python写的代码）
- V 视图（模版）

### MVT

- M 数据模型
- C 视图函数（Python写的代码）
- T 模版

### Flask与HTTP

#### 常用的http方法

| 方法   | 说明               |
| ------ | ------------------ |
| GET    | 获取资源           |
| POST   | 传输数据，提交数据 |
| PUT    | 传输文件           |
| DELETE | 删除资源           |
| HEAD   | 获取文件报文首部   |
| OPTION | 询问支持方法       |

#### 常用http状态码

|    类型    | 状态码 |    状态码英文名称     |                             说明                             |
| :--------: | :----: | :-------------------: | :----------------------------------------------------------: |
|    成功    |  200   |          OK           |                        请求被正常处理                        |
|    成功    |  201   |        Created        |                 请求被处理，并创建一个新资源                 |
|    成功    |  204   |      No Content       |                 请求处理成功，但没有内容返回                 |
|   重定向   |  301   |   Moved Permanently   |                          永久重定向                          |
|   重定向   |  302   |         Found         |                          临时重定向                          |
|   重定向   |  304   |     Not Modified      |           请求的资源未被修改，重定向到换成你的资源           |
| 客户端错误 |  400   |      Bad Request      |              表示请求无效，即请求报文中存在错误              |
| 客户端错误 |  401   |     Unauthorized      | 类似403，表示请求的资源需要获取授权信息，在浏览器中会弹出认证弹窗 |
| 客户端错误 |  403   |       Forbidden       |                表示请求的资源被服务器拒绝访问                |
| 客户端错误 |  404   |       Not Found       |           表示服务器上无法找到请求的资源或URL无效            |
| 服务器错误 |  500   | Internal Server Error |                      服务器内部发生错误                      |

#### cookie与session

cookie 在浏览器。浏览器每次访问服务器都会带过去cookie，因此服务器可以修改浏览器的cookie

session  在服务器。浏览器访问服务器的时候，服务器就创建一个session对应浏览器，而且一个浏览器在一个服务器上有一个session

对于Flask来说，每一个浏览器访问服务器的时候，服务器都会创建一个session(字典)

当用户登录后，服务器需要记住用户的登陆状态，这里使用session记录，在session中可以添加一个键值对来标记用户的登陆状态，如果用户注销则把这个状态删掉。对于Flask来说这个过程不需要我们手动操作session，只需要调用login_user和logout_user就可以实现。

##### session

浏览器访问服务器的时候回建立一个session，在session服务器可以保存状态（比如用户状态）

##### cookie

 保存在浏览器中 服务器把session加密后放到了cookie

### 简单的Flask页面

##### 在app中views.py中

```python
from app import app

@app.route('/')  #路由
def index():  #视图函数
    return 'hello  world!'  #必须返回字符串
```

##### 在run.py(和app是一个级别)中

```python
from app import app

app.run(debug = True)
```

##### 在app的__init__.py中

```python
from flask import Flask

app = Flask(__name__）
            
from . import views
```

##### 运行

```shell
 python   run.py   runserver
 "运行成功可在浏览器的127.0.0.1:5000中查看"
```

### 模板(HTML使用)

在app中创建templates文件，将需要的HTML文件放在这个文件夹中

``` python
'在views.py中导入render_template
'使用jinja2的过程中一定要闭合'
```

### 模板继承

```jinja2
{% extends '想要继承的HTML' %}
{% block 自定义的块名%}
{{super()}} #想继承base里的块内容
{% endblock %}
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

### 静态文件(css,js)

```python
'必须要写在static文件中，和templates在一级
'修改HTML中静态文件地址
```

### 宏

类似函数的使用

在templates中创建mecro.HTML(可以有多个)

```jinja2
{% macro 自定义宏(所需参数) %}
经常判断的函数
{% endmacro %}
```

在需要宏的HTML中

```
{% import 'macro.HTML' as macro %}
{{macro.自定义宏(所需参数)}}
```

### 表单

安装flask-wtf

#### 请求表单

##### 在config.py配置文件中

```pyhon 
WTF_CSRF_ENABLED=True
SECRET_KEY='表单秘钥'
```

##### 在app的__init__.py中添加

```python
app.config.from_object('config')
```

创建forms.py

```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField, PasswordField, BooleanField
# 创建表单类
class LoginForm(FlaskForm) :
name = StringField(label='姓名')
password = PasswordField(label='密码')
remember_me = BooleanField(label='记住我')
submit = SubmitField(label='提交')
```

在templates下创建HTML

```html
<HTML>
<head>
</head>
<body>
<!--注意：这里提交表单用post方法-->
<form action="" method="post">
 <!--添加令牌-->
{{form.hidden_tag()}}

{{form.name(placeholder="账号",class="c1",style="")}}<br>
{{form.password(placeholder="密码")}}<br>
{{form.remember_me}}&nbsp;记住密码<br>
{{form.submit}}<br>
</form>
</body>
</HTML>
```

##### 在views.py中

```python 
from app import app
from .forms import LoginForm
from flask import render_template

@app.route('/login')
def login() :
# 创建表单对象
form = LoginForm()
# 渲染模板
return render_template('需要渲染的HTML', form=form)
```

#### 提交表单

##### 在views.py中

```python
from app import app
from .forms import LoginForm
from flask import render_template
# 请求表单：以GET⽅方式访问login路路由
# 提交表单：以POST⽅方式访问login路路由
#-------------------修改---------------------#
@app.route('/login', methods=['GET','POST'])
def login() :
form = LoginForm()
#-------------------------新加---------------------------#
# 判断表单的合法性（验证⽤用户填写是否正确）
if form.validate_on_submit() :
name = form.name.data
password = form.password.data
remember_me = form.remember_me.data
print(name, password, remember_me)
form.name.data = None
form.remember_me.data = False
#-------------------------新加---------------------------#
return render_template('需要渲染的表单', form=form)
```

#### 表单字段类型

````python
StringField 文本字段
TextAreaField 多行文本字段
PasswordField 密码文本字段
HiddenField 隐藏文本字段
DateField 文本字段,值为 datetime.date 格式
DateTimeField 文本字段,值为 datetime.datetime 格式
IntegerField 文本字段,值为整数
DecimalField 文本字段,值为 decimal.Decimal
FloatField 文本字段,值为浮点数
BooleanField 复选框,值为 True 和 False
RadioField 一组单选框
SelectField 下拉列列表
SelectMultipleField 下拉列列表,可选择多个值
FileField 文件上传字段
SubmitField 表单提交按钮
FormField 把表单作为字段嵌入另一个表单
FieldList 一组指定类型的字段
````

#### 表单验证(判断用户是否输入正确)

##### 在forms.py中

```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField, PasswordField, BooleanField
from wtforms.validators import Email, DataRequired, Length, Regexp
# DataRequired 要求用户必须填写
# Email 要求⽤用户输入必须是邮箱格式
# Length 限制用户输入长度
# Regexp 限制用户的输入必须符合正则表达式

# 创建表单类
class LoginForm(FlaskForm) :
	email = StringField(label='邮箱', validators=[DataRequired(message='邮箱不能空'),Email(message='必须是邮箱格式'),Length(6,64, message='长度必须是6-16')])
	name = StringField(label='姓名', validators=[DataRequired(message='名字必须填写'),Length(2, 32,message='长度必须是2-32')])
	password = PasswordField(label='密码', validators=[DataRequired(message='密码不不能为空'),
Length(6, 16,message='长度必须是6-16')])
	remember_me = BooleanField(label='记住我', default=False)
	submit = SubmitField(label='提交')
```

#### flash

有后端传入的内容的弹框

在HTML中

```html
{% for m in get_flashed_messages() %}
{{m}}
{% endfor %}
```

在views.py中

```python
# 新加 导入flash函数
from flask import flash

# 新加 发送flash消息
flash('恭喜！提交成功!' + name + ':' + password)
```

### 页面渲染与重定向

#### 页面渲染

```python 
from flask import render_template
return render_template('需要渲染的表单', form=form)
#页面渲染，即将数据显示在页面上
```

#### 重定向

```python 
#用代码的方式实现视图函数间的跳转，进而实现页面的切换
from flask import url_for,redirect
return redirect(url_for('.user_info', 需要传给下个页面的参数)
```

### Flask的数据模型

安装 flask-sqlalchemy 和 pymysql

使用ORM对象映射

在MySQL中创建库

```sql
CREATE DATABASE IF NOT EXISTS 数据库名称 DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

#### 表的创建与删除

##### 在config.py配置文件中

```python
SQLALCHEMY_DATABASE_URI='mysql://数据库用户名:密码@localhost/数据库名称'
#如果设置成 True (默认情况)，Flask-SQLAlchemy将会追踪对象的修改并且发送信号。
#注意:这需要额外的内存，如果不不必要的可以禁⽤用它。
SQLALCHEMY_TRACK_MODIFICATIONS = False
```

##### 在__init__.py中创建访问数据库的对象

```python
# 创建访问数据库的对象
db = SQLAlchemy(app)
```

##### 在models.py中

```python
from app import db
# 创建商品类 继承数据模型类的基类
class Shop(db.Model) :
    # 指定表名
    __tablename__ = 'shops'
    # 定义主键
    id = db.Column(db.Integer(), primary_key=True)
    # 定义其他字段(表属性)
    name = db.Column(db.String(32), default='商品')
    price = db.Column(db.Float(), default=1.1)
```

##### 在run.py中

```python
from app import db
# 注意：一定要导入数据模型类
from app import models

# 根据模型类创建表
db.create_all()
# 根据模型删除表
db.drop_all(
```

#### 表关系

```python
class User(db.Model,UserMixin):
     role_id = db.Column(db.Integer,db.ForeignKey('roles.id'))
class Role(db.Model):
	users = db.relationship('User',backref='role')

```

在user表中有role_id属性，但Role表中没有users属性

这两句话就是使用Flask的语句，数据还是有SQL语句查询出来的

db.relationship()中的backref参数向User模型中添加role属性，从而定义反向关系。这个属性可以代替Role模型，此时获取的是模型对象，而不是外键的值

#### 数据库的增删改查

##### 添加数据

```python
 t = Teacher()
 t.name = 'xxxx'
 t.age = 35
 t.salary = 12.25
 db.session.add(t)
 db.session.commit()
```

##### 删除数据

```python
 db.session.delete(t)
 db.session.commit()
```

##### 修改数据

````python
 t.name='yyyyyy'
 db.session.add(t)
 db.session.commit()
````

##### 查询数据

```shell
>>>Teacher.query.all()   #查询表中所有数据
[<Teacher 1>, <Teacher 2>, <Teacher 3>, <Teacher 4>, <Teacher 5>, <Teacher9>]
>>> Teacher.query.first()  #查询表中第一个数据
<Teacher 1>
```

|      方法      |                             描述                             |
| :------------: | :----------------------------------------------------------: |
|     all()      |                 以列表形式返回查询的所有结果                 |
|    first()     |        返回查询的第一个结果,如果没有结果,则返回 None         |
| first_or_404() | 返回查询的第一个结果,如果没有结果,则终止请求,返回 404 错误响应 |
|     get()      |      返回指定主键对应的行,如果没有对应的行,则返回 None       |
|    count()     |                      返回查询结果的数量                      |
|   paginate()   | 返回一个 Paginate 对象,它包含指定范围内的结果 (一般用于分页) |

```python
# 使用query对象中的filter方法进行数据筛选，返回一个新的query对象
>>> l = Student.query.filter(Student.name=='s1').all()
>>> l = Student.query.filter(Student.name=='s1', Student.id>5).all ()
>>> l = Student.query.filter(Student.name >'s3', Student.id>5).all()
>>> l = Student.query.filter(Student.name <'s3', Student.id>5).all()
>>> l = Student.query.filter(Student.name !='s3', Student.id>5).all()
>>> l = Student.query.filter(Student.name.like('s3'), Student.id>5).all()
# %代表0个或者多个字符
>>> l = Student.query.filter(Student.name.like('s%'), Student.id>5).all()
>>> l = Student.query.filter(Student.name.like('%s1%'), Student.id>5).all()
# _代表一个字符
>>> l = Student.query.filter(Student.name.like('_s1_'), Student.id>5).all()
# 把集合作为查询条件
>>> d = {Student.name.like('s1'), Student.id>5}
>>> l = Student.query.filter(*d).all()
# or not
>>> from sqlalchemy import not_, or_，and_
>>> s = Student.query.filter(or_(Student.id>2, Student.id<5)).all()
>>> s = Student.query.filter(not_(Student.id>2)).all()
>>> s = Student.query.filter(and_(Student.id>2, Student.id<5)).all()
# 注意:可以连续调用多次filter，相当于and
>>> l = Student.query.filter(Student.name=='s1').filter(Student.id>5).all()
# 注意:如果要多个条件或的话，可以先按照各⾃自的条件查询，然后把返回的列列 表相加再变成集合去重使用query对象中的filter_by方法进行数据筛选，返回一个新的query对象
# 列名不需要写类名
# 只能写等于判断
>>> l = Student.query.filter_by(name='s3').all()
>>> l = Student.query.filter_by(name='s3',id=5).all()
>>> l = Student.query.filter_by(name='s3',id=6).all()
# 把字典作为查询条件
>>> d = {'name':'s3', 'id':6}
>>> l = Student.query.filter_by(**d).all()
```

##### 排序/限制/偏移

````python
# 使用query对象的limit方法限制查询的记录条数，返回一个新的query对象
# 最多返回3条记录
>>> l = Student.query.limit(3).all()
>>> l = Student.query.filter(*d).limit(3).all()
# 使用query对象的offset方法限制查询的偏移，返回一个新的query对象
# 查询到的数据从偏移为2的数据开始显示
>>> l = Student.query.filter_by(name='s1').offset(2).all()
# 使用query对象的order_by⽅方法进行排序，返回一个新的query对象
>>> l = Student.query.order_by(Student.name).all()
````

##### 部分查询

````python
#查询全部列
db.session.query(Teacher).all()
#查询id列name列
db.session.query(Teacher.id, Teacher.name.filter(Teacher.id<5).all()
````

缺点是只能访问被查询的这⼏几列列，如果想访问其他列列得重新查询。

#### 聚合函数

##### 求平均值

```python
db.session.query(func.avg(Teacher.age)).first()
```

##### 求最大值

```python
db.session.query(func.max(Teacher.age)).first()
```

##### 求最小值

```python
db.session.query(func.min(Teacher.age)).first()
```

##### 求和

````python
db.session.query(func.sum(Teacher.age)).first()
````

##### 统计行数

```python
Teacher.query.count()  #表中所有行数
Teacher.query.filter(Teacher.id > 3).count()  #也可以带条件查询行数

from sqlalchemy import func
db.session.query(func.count(Teacher.id))  #返回一个query对象
db.session.query(func.count(Teacher.id)).first()   #返回一个元祖

```

#### 延迟加载(查询不显示)

##### 在models.py中使用defferred方法

```python
from sqlalchemy.orm import deferred
class Student(db.Model) :
	__tablename__ = 'students'
	id = db.Column(db.Integer, primary_key=True)
	name = db.Column(db.String(32))
	teacher_id = db.Column(db.Integer, db.ForeignKey('teachers.id'))
	#使用deferred修饰的列属性，在使用query查询的过程中不不会加载到对象中
	#直到直接访问这些列列属性时候才会从数据库中加载到对象中
	max_score = deferred(db.Column(db.Integer, default=0))
	mix_score = deferred(db.Column(db.Integer, default=0))
	#延迟加载的字段可以分组，当有任何⼀一个列列属性被直接访问时，同组的都会被加载到对象中
	avr_score = deferred(db.Column(db.Integer, default=0), group='score')
	now_score = deferred(db.Column(db.Integer, default=0), group='score')
```

##### 使用query对象的option方法进行延迟加载

````python
#导入方法
from sqlalchemy.orm import defer, undefer,load_only
#延迟加载
Student.query.options(defer('name')).filter_by(id=1).all()
#延迟加载方法混合使用
Student.query.options(defer('name').undefer('max_score')).filter_by(id>1).all()
#加载指定列
Student.query.options(load_only('id','name')).filter_by(id=1).all()
````

#### 数据库迁移

安装flask-migrate

##### 在run.py中添加

````python
#新增加
from flask_migrate import Migrate, MigrateCommand
migrate = Migrate(app, db)
......
#新增加
manager.add_command('db', MigrateCommand)
````

```shell
python run.py db init     #初始化数据库
python run.py db migrate  #创建数据库迁移脚本
python run.py db upgrade  #提交数据库更新

'每次修改完数据模型类后执行migrate和upgrade即可更改表结构并且不破环数据'
```

### 集成Python  Shell

安装flask-script

##### 在run.py中添加

```python
#导入相应模块
from flask_script import Shell, Manager
from app.models import Student, Teacher
from app import app
#创建Manager对象
manager = Manager(sqlApp)

def make_shell_context():
	return dict(app=app, db=db, Student=Student, T=Teacher)
#添加命令shell
	manager.add_command("shell", Shell(make_context=make_shell_context))
	manager.run()
```

以后可用python  run.py  shell进入环境，不需要导入app，db

### 邮件

安装flask-mail

##### 在config.py配置文件中

````python
MAIL_SERVER='smtp.163.com'
MAIL_PORT='25'
MAIL_USE_TLS=True
MAIL_USERNAME='xxxxxx@163.com'
MAIL_PASSWORD='xxxxxx
````

##### 发送普通邮件

```python
from flask_mail import Mail, Message
from app import app
mail = Mail(app)

msg = Message('没有主题',
sender = 'xxxx@163.com',
recipients = ['xxxxxx@163.com'])
msg.body = '来自Flask邮件'

with app.app_context():
	mail.send(msg)
```

##### 发送HTML邮件

创建一个TXT文件和HTML文件

````python
from flask_mail import Mail, Message
from app import app
from flask import render_template
mail = Mail(app)
#把发邮件的功能封装成一个函数
def send_mail(subject, sender, recvers, body, html) :
	msg = Message(subject, sender = sender, recipients = recvers)
	msg.body = body
	msg.html = html
    
    with mailApp.app_context():
		mail.send(msg)
        
@app.route('/')
def index() :
    #文本模板
	body = render_template('mail/test.txt', sender='嵌⼊入式天空')
	#html模板
	html = render_template('mail/test.html', sender = '嵌⼊入式天空')
	send_mail('垃圾邮件', '发送方',['接收方'],body, html)
	return 'ok'
app.run()
````

当文本邮件和HTML邮件冲突时，HTML优先选择

##### 异步邮件

````python
from flask_mail import Mail, Message
from app import app
from flask import render_template
from threading import Thread

mail = Mail(app)

#线程处理函数
def send_fun(app, msg) :
	with app.app_context():
		mail.send(msg)
	
def send_mail(subject, sender, recvers, body, html) :
	msg = Message(subject, sender = sender, recipients = recvers)
	msg.body = body
	msg.html = html
	#创建新线程
	thread = Thread(target=send_fun, args=[mailApp, msg])
	thread.start()
    
@app.route('/')
def index() :
	body = render_template('mail/test.txt', sender='嵌入式天空')
	html = render_template('mail/test.html', sender = '嵌入式天空')
    send_mail('垃圾邮件', '发送方',['接收方'], body, html)
	return 'ok'

app.run()
````



























