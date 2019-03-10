# Celery+Redis



## 基本概念 

#### Brokers

brokers 中文意思为中间人，在这里就是指任务队列本身，Celery 扮演生产者和消费者的角色，brokers 就是生产者和消费者存放/拿取产品的地方(队列)，常见的 brokers 有 rabbitmq、redis、Zookeeper 等。

#### Result Stores / backend

顾名思义就是结果储存的地方，队列中的任务运行完后的结果或者状态需要被任务发送者知道，那么就需要一个地方储存这些结果，就是 Result Stores 了，常见的 backend 有 redis、Memcached 甚至常用的数据都可以。

#### Workers

就是 Celery 中的工作者，类似与生产/消费模型中的消费者，其从队列中取出任务并执行。

#### Tasks

就是我们想在队列中进行的任务，一般由用户、触发器或其他操作将任务入队，然后交由 workers 进行处理。

## Redis

#### Centos7下安装Redis

yum install redis

systemctl start redis

## Celery

### 安装celery

pip install redis

pip install celery

### 一个简单的例子

###### 创建一个tasks.py文件

```python
from celery import Celery

app = Celery('tasks',  backend='redis://localhost:6379/0', broker='redis://localhost:6379/0')

@app.task
def add(no):
	for i in range(no) :
		print('hello celery')
```

###### 启动celery工作者

(venv) [embsky@localhost Test]$ celery -A tasks worker --loglevel=info

###### 创建一个test.py

```python
from tasks import add
import time
#调度任务
result = add.delay(2)  #2是add的参数
#等待任务完成
while not result.ready():
	time.sleep(1)
#获取任务结果
print('task done: {0}'.format(result.get()))
```

###### 调度任务

(venv) [embsky@localhost Test]$ python test.py 

### 任务继承

利用任务继承可以实现任务结束回调或者任务失败回调

###### 构建任务类（tasks.py）

```python
from celery import Celery

app = Celery('tasks',  backend='redis://localhost:6379/0', broker='redis://localhost:6379/0')

from celery import Task
class MyTask(Task):
    def on_success(self, retval, task_id, args, kwargs):
        print ('=================task done: {0}'.format(retval))
        return super(MyTask, self).on_success(retval, task_id, args, kwargs)
    
    def on_failure(self, exc, task_id, args, kwargs, einfo):
        print ('=================task fail, reason: {0}'.format(exc))
        return super(MyTask, self).on_failure(exc, task_id, args, kwargs, einfo)

@app.task(base=MyTask)
def taskfunc(x, y):
    return x + y
```

###### 启动celery工作者

(venv) [embsky@localhost Test]$ celery -A tasks worker --loglevel=NOTSET

###### 任务调度（test.py)

```python
from tasks import add
import time

from tasks import taskfunc

result = taskfunc.delay(1,2)

while not result.ready() :
	time.sleep(1)
print('task done: {}'.format(result.get()))
```

(venv) [embsky@localhost Test]$ python test.py 

### 任务状态回调

利用任务状态回调可以实现监测一个后台任务的进度

###### 构建任务（tasks.py）

```python
from celery import Celery
import time

app = Celery('tasks',  backend='redis://localhost:6379/0', broker='redis://localhost:6379/0')

#bind=True代表绑定任务为实例方法
@app.task(bind=True)
def test_mes(self):
    for i in range(10):
        time.sleep(1)
        #更新状态，状态为PROGRESS，状态的值为{'p':i}
        self.update_state(state="PROGRESS", meta={'p': i})
    return 'finish'
```

###### 启动celery工作者

(venv) [embsky@localhost Test]$ celery -A tasks worker --loglevel=NOTSET

###### 任务调度（test.py）

```python
from tasks import test_mes
import sys

def pm(body):
    #获取状态值
    res = body.get('result')
    #获得状态，如果任务结束的话或得的状态是SUCCESS
    if body.get('status') == 'PROGRESS':
        sys.stdout.write('\r任务进度: {0}%'.format(res.get('p')))
        sys.stdout.flush()
    else:
        print('\r')
        print(res)
r = test_mes.delay()
#任务每一次更新状态（任务结束也会更新状态）pm方法都会被调用，body参数为任务的信息
print(r.get(on_message=pm, propagate=True))
```

###### 任务的其他状态

| 参数    | 说明         |
| ------- | ------------ |
| PENDING | 任务等待中   |
| STARTED | 任务已开始   |
| SUCCESS | 任务执行成功 |
| FAILURE | 任务执行失败 |
| RETRY   | 任务将被重试 |
| REVOKED | 任务取消     |

### 定时任务

利用定时任务机制可以定时循环执行一个任务

###### celery配置文件（celery_config.py）

```python
from datetime import timedelta
from celery.schedules import crontab

CELERYBEAT_SCHEDULE = {
    'ptask': {
        'task': 'tasks.period_task',
        'schedule': timedelta(seconds=5),
    },
}

CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'
```

###### 任务调度（tasks.py）

```python
from celery import Celery
import time

app = Celery('tasks', backend='redis://localhost:6379/0', broker='redis://localhost:6379/0')
app.config_from_object('celery_config')

@app.task(bind=True)
def period_task(self):
    print('period task done: {0}:{1}'.format(self.request.id))
```

###### 启动工作者

(venv) [embsky@localhost Test]$ celery -A tasks worker --loglevel=NOTSET

###### 启动定时任务

(venv) [embsky@localhost Test]$ celery -A tasks beat

### 链式任务

链式任务可以把多个异步任务组成一个链式的结构，前一个任务的输出作为后一个任务的输入，任务之间同步

###### 构建任务（tasks.py)

```python
from celery import Celery
import time

app = Celery('tasks', backend='redis://localhost:6379/0', broker='redis://localhost:6379/0')

@app.task
def task1(a):
    print('task1')
    return a + 1

@app.task
def task2(a):
    print('task2')
    return a + 2

@app.task
def task3(a):
    print('task3')
    return a + 3
```

###### 调度任务

```python
from tasks import task1, task2, task3
from celery import chain
#构建并执行链式任务，前一个任务的输出是后一个任务的其中一个输入
result = chain(task3.s(1), task2.s(), task1.s())
#或得链式任务的结果
print(result().get())

#构建链式任务
ch = task3.s(1) | task2.s() | task1.s()
#执行链式任务并且获得结果
print(ch().get())
```



# Django+Celery+Redis

### 安装

(venv) [embsky@localhost ~]$ pip install celery

(venv) [embsky@localhost ~]$ pip install celery-with-redis

(venv) [embsky@localhost ~]$ pip install django-celery

### 构建一个Django测试工程

(venv) [embsky@localhost ~]$ django-admin startproject celery_project

(venv) [embsky@localhost ~]$ cd celery_project

(venv) [embsky@localhost celery_project]$ django-admin startapp one

#### 1.注册APP

(venv) [embsky@localhost celery_project]$ vim celery_project/settings.py

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'one',   		#自己创建的app
    'djcelery',		#celery app
]
```

#### 2.增加Celery配置

(venv) [embsky@localhost celery_project]$ vim celery_project/settings.py

```python
import djcelery
djcelery.setup_loader()

BROKER_URL = 'redis://127.0.0.1:6379/0'
CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/0'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERYBEAT_SCHEDULER = 'djcelery.schedulers.DatabaseScheduler'
CELERY_TIMEZONE = 'Asia/Shanghai'
```

#### 3.创建Celery实例

(venv) [embsky@localhost celery_project]$ vim celery_project/celery.py

```python
import os
from celery import Celery
from django.conf import settings

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'celery_project.settings')

app = Celery('one')

app.config_from_object('django.conf:settings')
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)

@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```

#### 4.引入Celery实例

(venv) [embsky@localhost celery_project]$ vim celery_project/\_\_init\_\_.py 

```python
from .celery import app as celery_app
```

#### 5.创建任务文件

(venv) [embsky@localhost celery_project]$ vim one/tasks.py

```python
from time import sleep

@task()
def Task_A(message):
    Task_A.update_state(state='PROGRESS', meta={'progress': 0})
    sleep(10)
    Task_A.update_state(state='PROGRESS', meta={'progress': 50})
    sleep(10)
    return message

def get_task_status(task_id):
    task = Task_A.AsyncResult(task_id)

    status = task.state
    progress = 0

    if status == u'SUCCESS':
        progress = 100
    elif status == u'FAILURE':
        progress = 0
    elif status == 'PROGRESS':
        progress = task.info['progress']

    return {'status': status, 'progress': progress} 
```

#### 6.启动Broker

python manage.py celeryd -l info

#### 7.测试

```python
(venv) [embsky@localhost celery_project]$ python manage.py shell
Python 3.6.6 (default, Aug 13 2018, 18:24:23) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-28)] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from one.tasks import *
#调度任务
>>> t = Task_A.delay('hahaha')
>>> t.ready()
False
>>> t.ready()
True
#获得任务结果
>>> t.get(t.id)
'hahaha'
```

### 发送异步邮件

#### 8.添加邮件配置

(venv) [embsky@localhost celery_project]$ vim celery_project/settings.py

```python
EMAIL_USE_SSL = True
EMAIL_HOST = 'smtp.163.com' # 如果是 163 改成 smtp.163.com 
EMAIL_PORT = 465
EMAIL_HOST_USER = 'lizhiyong_python@163.com' # 帐号 
EMAIL_HOST_PASSWORD = 'uplooking123' # 密码 
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER
```

#### 9.编写发送邮件任务

(venv) [embsky@localhost celery_project]$ vim one/tasks.py

```python
from django.core.mail import send_mail

@task()
def sendmail(id) :
    send_mail('test', 'hahahaha', 'lizhiyong_python@163.com', ['bjzhangwei@uplooking.com', 'lizhiyong_beyond@163.com', 'lzy@uplooking.com', 'lizhiyong_python@163.com'], fail_silently=False)
    return id
```

#### 10.编写测试视图函数

(venv) [embsky@localhost celery_project]$ vim one/views.py

```python
from django.shortcuts import HttpResponse
from .tasks import sendmail

# Create your views here.

def testfun(request) :
    sendmail.delay(1)
    return HttpResponse('send over')
```

#### 11.增加路由

(venv) [embsky@localhost celery_project]$ vim celery_project/urls.py

```python
from one import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('test/', views.testfun),
]
```

#### 12.重新启动Broker

python manage.py celeryd -l info

#### 13.启动服务器程序那你

python manage.py runserver

#### 14.在浏览器中访问测试

Localhost:8000/test

