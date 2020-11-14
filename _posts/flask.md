# 1 使用Flask-Migrate实现数据库迁移 pip install flask-migrate
```
from flask.ext.migrate import Migrate, MigrateCommand

migrate = Migrate(app, db)
manager.add_command('db', MigrateCommand)

# 创建迁移仓库
(venv) $ python hello.py db init
# 自动创建迁移脚本
(venv) $ python hello.py db migrate -m "initial migration"
# 检查并修正好迁移脚本后，更新数据库（尴尬：还要去检查）
(venv) $ python hello.py db upgrade  # upgrade() 将改动应用到数据库；downgrade() 则删除改动
```

# 2 使用Flask-Mail提供电子邮件支持 pip install flask-mail
```
from flask.ext.mail import Mail
mail = Mail(app)

import os
# ...
app.config['MAIL_SERVER'] = 'smtp.googlemail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = os.environ.get('MAIL_USERNAME')
app.config['MAIL_PASSWORD'] = os.environ.get('MAIL_PASSWORD')

(venv) $ python hello.py shell
>>> from flask.ext.mail import Message
>>> from hello import mail
>>> msg = Message('test subject', sender='you@example.com',
... recipients=['you@example.com'])
>>> msg.body = 'text body'
>>> msg.html = '<b>HTML</b> body'
>>> with app.app_context():
... mail.send(msg)
```
# 3 使用Flask-SQLAlchemy管理数据库 pip install flask-sqlalchemy
```
from flask.ext.sqlalchemy import SQLAlchemy
basedir = os.path.abspath(os.path.dirname(__file__))
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///' + os.path.join(basedir, 'data.sqlite')
app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN'] = True  # 每次请求结束后都会自动提交数据库中的变动
db = SQLAlchemy(app)  # db 对象是 SQLAlchemy 类的实例，表示程序使用的数据库，同时还获得了 Flask-SQLAlchemy
提供的所有功能
```
- 定义模型
```
class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True)
    users = db.relationship('User', backref='role')  # 定义关系
    
    def __repr__(self):  # 返回一个具有可读性的字符串表示模型，可在调试和测试时使用
        return '<Role %r>' % self.name
    
    
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))  # 定义关系
    
    def __repr__(self):
        return '<User %r>' % self.username

# 常用的SQLAlchemy关系选项
# backref 在关系的另一个模型中添加反向引用
# primaryjoin 明确指定两个模型之间使用的联结条件。只在模棱两可的关系中需要指定
# lazy 指定如何加载相关记录。可选值有 select（首次访问时按需加载）、immediate（源对象加载后就加载）、joined（加载记录，但使用联结）、 subquery（立即加载，但使用子查询），noload（永不加载）和  dynamic（不加载记录，但提供加载记录的查询
# uselist 如果设为 Fales，不使用列表，而使用标量值
# order_by 指定关系中记录的排序方式
# secondary 指定多对多关系中关系表的名字
# secondaryjoin SQLAlchemy 无法自行决定时，指定多对多关系中的二级联结条件

关系类型是多对多，需要用到第三张表， 这个表称为关系表
```
-  数据库操作
```
创建表
(venv) $ python hello.py shell
>>> from hello import db
>>> db.create_all()  # db.drop_all()

插入行
>>> from hello import Role, User
>>> admin_role = Role(name='Admin')
>>> user_david = User(username='david', role=user_role)

通过数据库会话管理对数据库的改动，所以在准备写入之前要先保存到会话中
>>> db.session.add(admin_role)
>>> db.session.add(user_david)
或 >>> db.session.add_all([admin_role, user_david])

提交会话（数据库会话也成为事务）
>>> db.session.commit()
```
- 数据库会话能保证数据库的一致性。提交操作使用原子方式把会话中的对象全部写入数据
- 库。如果在写入会话的过程中发生了错误， 整个会话都会失效。如果你始终把相关改动放
- 在会话中提交，就能避免因部分更新导致的数据库不一致性。
- 数据库会话也可回滚。调用 db.session.rollback() 后，添加到数据库会话
- 中的所有对象都会还原到它们在数据库时的状态。
```
修改行
>>> admin_role.name = 'Administrator'
>>> db.session.add(admin_role)
>>> db.session.commit()

删除行
>>> db.session.delete(mod_role)
>>> db.session.commit()

查询行: Flask-SQLAlchemy 为每个模型类都提供了 query 对象
>>> Role.query.all()
[<Role u'Administrator'>, <Role u'User'>]
>>> User.query.all()
[<User u'john'>, <User u'susan'>, <User u'david'>]
>>> User.query.filter_by(role=user_role).all()  # 使用过滤器
[<User u'susan'>, <User u'david'>]
>>> str(User.query.filter_by(role=user_role).first())  # 查看原生的SQL语句
```
-  query 对象常见的过滤器
```
filter() filter_by() limit()  offset()  order_by()  group_by()

```
- 常见的 SQLAlchemy 查询执行函数
```
all()  first()  first_or_404()  get()  get_or_404()  count()  paginate()#返回一个 Paginate 对象，包含指定范围内的结果
```
- 在视图函数中操作数据库
```
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.name.data).first()
        if user is None:
            user = User(username = form.name.data)
            db.session.add(user)
            session['known'] = False
        else:
            session['known'] = True
            session['name'] = form.name.data
            form.name.data = ''
            return redirect(url_for('index'))
    return render_template('index.html', form = form, name = session.get('name'), known = session.get('known', False))

{% if not known %}
    <p>Pleased to meet you!</p>
{% else %}
    <p>Happy to see you again!</p>
{% endif %}
```
# 4 集成Python shell，将不再需要每次启动 shell 会话都导入数据库实例和模型
```
from flask.ext.script import Shell
def make_shell_context():  # 为 shell 命令注册一个 make_context 回调函数
    return dict(app=app, db=db, User=User, Role=Role)
manager.add_command("shell", Shell(make_context=make_shell_context))
```
# 5 Flask 学习笔记
在 Flask 中有两种上下文： 程序上下文和请求上下文; Flask 在分发请求之前激活（或推送）程序和请求上下文，请求处理完成后再将其删除
```
current_app 程序上下文 当前激活程序的程序实例
g 程序上下文 处理请求时用作临时存储的对象。每次请求都会重设这个变量
request 请求上下文 请求对象，封装了客户端发出的 HTTP 请求中的内容
session 请求上下文 用户会话，用于存储请求之间需要“记住”的值的词典
```
在程序实例上调用 app.app_context() 可获得一个程序上下文
```
app_ctx = app.app_context()
app_ctx.push()
curent_app.name
app_ctx.pop()

app.url_map() app.route  app.add_url_urle()
```
Flask 支持以下 4 种钩子
```
before_first_request：注册一个函数，在处理第一个请求之前运行
before_request：注册一个函数，在每次请求之前运行
after_request：注册一个函数，如果没有未处理的异常抛出，在每次请求之后运行
teardown_request：注册一个函数，即使有未处理的异常抛出，也在每次请求之后运行

在请求钩子函数和视图函数之间共享数据一般使用上下文全局变量 g, g.user 获取已登录用户
```
使用Flask-Script支持命令行选项, pip install flask-script
```
from flask.ext.script import Manager  # 专为 Flask 开发的扩展都暴漏在 flask.ext 命名空间下
manager = Manager(app)
# ...
if __name__ == '__main__':  # python hello.py runserver --help
manager.run()
```
渲染模板
```
from flask import Flask, render_template  # Flask 提供的 render_template 函数把 Jinja2 模板引擎集成到了程序中
# ...
@app.route('/')
def index():
return render_template('index.html')  # 第一个参数是模板的文件名

@app.route('/user/<name>')
def user(name):
return render_template('user.html', name=name)  # 第二个参数是变量的键值对
```
模板中使用变量的常用方法
![image 01](https://github.com/Django-27/workspace/blob/master/pic/flask_01.png)

模板中常用的控制语句
![image 02](https://github.com/Django-27/workspace/blob/master/pic/flask_02.png)

使用Flask-Bootstrap集成Twitter Bootstrap， pip install flask-bootstrap

![image 03](https://github.com/Django-27/workspace/blob/master/pic/flask_03.png)
自定义错误页面
![image 04 ](https://github.com/Django-27/workspace/blob/master/pic/flask_04.png)
静态文件, 其引用被当做一个特殊的路由，即 /static/<filename>
默认会从根目录下名为 static 的子目录中寻找静态文件
![image 05](https://github.com/Django-27/workspace/blob/master/pic/flask_05.png)
使用Flask-Moment本地化日期和时间, pip install flask-moment，依赖(moment.js 和 jquery.js)
采用：在服务器中只使用 UTC 时间，Web 浏览器负责转换成当地时间（浏览器可以获取用户电脑的时区和区域设置）
![image 06](https://github.com/Django-27/workspace/blob/master/pic/flask_06.png)

# 6 Web 表单 pip install flask-wtf
app.config 字典可用来存储框架、扩展和程序本身的配置变量
```
app = Flask(__name__)
app.config['SECRET_KEY'] = 'hard to guess string`
```
一个简单的 Web 表单，包含一个文本字段和一个提交按钮
```
from flask.ext.wtf import Form
from wtforms import StringField, SubmitField
from wtforms.validators import Required

class NameForm(Form):
name = StringField('What is your name?', validators=[Required()])  # 使用了內建的验证函数，确保字段中有值；还有验证电子邮箱 Email，输入的两次密码一样 EqualTo 等
submit = SubmitField('Submit')
```
视图函数可以将一个 NameForm 实例通过参数 form 传入模板，在模板中生产一个简单的表单
![image 07](https://github.com/Django-27/workspace/blob/master/pic/flask_07.png)
更简便的方式，使用 bootstrap 中预定义表单渲染整个 Flask-WTF 表单，只需一次操作
![image 08](https://github.com/Django-27/workspace/blob/master/pic/flask_08.png)
视图函数中处理表单
```
from flask import Flask, render_template, session, redirect, url_for
@app.route('/', methods=['GET', 'POST'])
def index()：
    form = NameForm()
    if form.validate_on_submit():  # 简单例子，有输入值，则显示相应的欢迎信息，并清空 form 中的 name 信息； 一次 GET 一次 POST 交替进行
        session['name'] = form.name.data  # 刷新的时候也可以记住这个名字
        return redirect(url_for('index')  # 加入重定向
    return render_template('index.html', form=form, name=session.get['name'])
```
Flash消息：在发给客户端的下一个响应中显示一个消息
```
from flask import Flask, render_template, session, redirect, url_for, flash
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        old_name = session.get('name')
        if old_name is not None and old_name != form.name.data:
            flash('Looks like you have changed your name!')  # 显示消息
        session['name'] = form.name.data
        return redirect(url_for('index'))
    return render_template('index.html', form = form, name = session.get('name'))
```
在模板中国加入对 flash 的支持，才能显示消息; Flask 把 get_flashed_messages() 函数开放给模板，用来获取并渲染消息
![image 09](https://github.com/Django-27/workspace/blob/master/pic/flask_09.png)
