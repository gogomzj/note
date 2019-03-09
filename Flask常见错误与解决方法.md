## Flask常见错误与解决方法

1.ValueError: urls must start with a leading slash

这个错误原因可能发生在所有路由相关地方，少加了一个'/'造成的。

2.ImportError: cannot import name 'db'

这个错误原因是产生了循环导入问题，修改import的位置即可

3.AssertionError: View function mapping is overwriting an existing endpoint function: manager.check_permission

这个错误原因是自定义的装饰器中没有使用functools模块下的wraps(func)修饰wrapper (func)

4.socket.gaierror: [Errno -2] Name or service not known

这个错误原因是当发送邮件等事务发生时，虚拟机没联网

5.The method is not allowed for the requested URL.

这个错误原因是路由书写时候，前一个路由没有写methods=['POST','GET']

6.AttributeError: 'SQLAlchemy' object has no attribute 'commit'

这个错误原因是db.session.commit(),可能忘记写session

7.RuntimeError: No application found. Either work inside a view function or push an application context.

这个错误原因是应该写进视图模块中的代码约束，写到了表单模块中

8.TypeError: redirect() got an unexpected keyword argument 'id'

这个错误原因可能不仅仅是重定向传错参数，还有可能只写了redirect,忘记写url_for

9.ERROR [root] Error: Can't locate revision identified by '8a92edbd0e8e'

这个问题是数据库中表alembic_version版本过低，直接删除

 