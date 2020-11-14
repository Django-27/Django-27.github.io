# 1 django on_delete 参数
- 当一个被外键关联的对象被删除时，执行的相应操作;
- CASCADE 模拟SQL语言中ON DELETE CASCADE约束，将定义有外键的模型对象同时删除，即A中被删除时，B也被删除 （关联的那些行）
- PROTECT 阻止上面的删除操作，但是弹出ProtectedError异常;
- SET_NULL 将外键字段设为null，只有当字段设置了null=True时，才可以使用该参数;
- SET_DEFAULT 将外键字段设为默认值，只有当字段设置了default参数是，才可以使用该参数;
- DO_NOTHING 什么也不做; SET(): 自定义一个值，该值当然只能是对应的实体;
``` javascript
class A(models.Model):
    date = models.DateField()
class B(models.Model):
    foreign = models.ForeignKey(A, on_delete=models.CASCADE)
```

# 2 Django 多 LTS(long-term support) release 认识
- 在任何时候，Django的开发团队都将支持不同级别的发行版,即多LTS并行存在
- 每个发行周期由三个部分组成：功能建议（新功能公布在wiki roadmap），持续开发（实现上一步的计划，未实现的移到下一个版本),bugfixes(修复错误，确定稳定版本)[Django’s release process](https://docs.djangoproject.com/en/dev/internals/release-process/)

Django version | Python versions
:-:| :-:
1.11   | 2.7, 3.4, 3.5, 3.6, 3.7 (added in 1.11.17)
2.0 |3.4, 3.5, 3.6, 3.7
2.1 |3.5, 3.6, 3.7
2.2 |3.5, 3.6, 3jjuuuu.7, 3.8 (added in 2.2.8)
3.0, 3.1 |    3.6, 3.7, 3.8

![django-release-roadmap](https://static.djangoproject.com/img/release-roadmap.688d8d65db0b.png)

- django2.* python兼容性，可以查看上面的表格
- django2.* 外键约束字段需要增加on_delete参数,由之前的可选变为必选参数
- django2.* 中间件配置MIDDLEWARE_CLASSES = (...) 变为 MIDDLEWARE = （...）
- django2.* 路由语法改变，django.conf.urls.url()和include（）改变为 from django.urls import path, re_path, include (url中的值之前是圆括号现在是尖括号,可以为捕获到的值指定类型如int默认是字符串，匹配模式的开头不加\ )
- django2.* QuerySet切片后禁止再次reverse()、last()
- django2.* forms.IntegerField(25, 10) 替换为 forms.IntegerField(max_value=25, min_value=10) 即不在接受可选参数作为位置参数
- django2.* models.Index(['headline', '-pub_date'], 'index_name') 替换为 models.Index(fields=['headline', '-pub_date'], name='index_name') 即不在接受位置参数
- django2.* SessionAuthenticationMiddleware 不在支持，需要注释掉
- django2.* AUTHENTICATION_BACKENDS的配置从tuple变成array，即AUTHENTICATION_BACKENDS = (...) 变为 AUTHENTICATION_BACKENDS  = [...]
- django2.* 如果在urls.py里使用了namespace, 需指定app_name,如url(r'^videos/', include('applications.videos.urls', namespace='videos')) 并在 pplications.videos.urls里添加app_name = 'videos'
- django3.* 支持ASGI,Django3.0开始具备全双工的异步通信能力(WSGI模式下只能同步通信,ASGI模式同时支持异步和同步通信)

# Django 性能监控
- 接触多的是Django Debug Toolbar，Silk和FlameGraphs是开源的第三方扩展 [django-debug-toolbar](https://django-debug-toolbar.readthedocs.io/en/latest/)
- Silk是Django框架的实时分析和检查工具，在将HTTP请求和数据库查询呈现在用户面前前进行进一步检查 ![django-silk](https://raw.githubusercontent.com/jazzband/django-silk/master/screenshots/1.png)
- Flame Graphs (火焰图) for Django Debug Toolbar ![djdt-flamegraph](http://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)
- 拓展：企业级的应用性能监控平台，例如 OneAPM，听云 和其他大厂的APM产品；(Application Performance Monitor APM 是应用性能监测软件）
- 拓展：电竞中APM是手速Actions Per Minute的缩写;后来又有了EAPM,每分钟有效操作数(Effective Actions Per Minute）

#  [django]按天分组统计数据
```
from django.db import connection 
from django.db.models import Count

# date_created = models.DateField(auto_now_add=True)
select = {'day': connection.ops.date_trunc_sql('day', 'date_created')}
Foobar.objects.extra(select=select).values('day').annotate(number=Count('id'))
# [{'number': 10, 'day': datetime.datetime(2013, 9, 29, 0, 0, 0}]
```
# Django 并发时保证操作数据的一致性
```
from django.db import transaction
添加事物一：
@transaction.atomic
添加事物二：
with transaction.atomic():
```
使用 select_for_update ，是数据库层面上专门用来解决并发取数据后再修改的场景
告诉数据库锁定对象，直到事务的完成(悲观锁)
```
# 必须需要事务
with transaction.atomic():
    # 需要使用 save 进行更新，
    order = Order.objects.select_for_update().get(id=order_id)
    if order.extra_status = Order.EXTRA_ST_NOT_PAY:
        order.extra_status = Order.EXTRA_ST_NORMAL
        order.save()
    
    # 不需要 save； 引出一个知识点，get 之后是一个 class 对象，而 filter 之后是一个 QuerySet 对象
    order = Order.objects.select_for_update().filter(id=order_id)
    if order.extra_status = Order.EXTRA_ST_NOT_PAY:
        order.extra_status = Order.EXTRA_ST_NORMAL
```
乐观锁：可以同时修改，先保存结果的为最后的保存结果，并且通知另一个无法保存的实时
悲观锁：不允许同时修改

# Django auth 修改用户密码
```
from django.contrib.auth.models import User
user = User.objects.create_user('jack', 'jack@hello.com', 'jackpassword')

u = User.objects.get(username__exact='jack')
u.set_password('new password')
u.save()

# 创建超级管理员
manage.py createsuperuser --username=jack --email=jack@hello.com
```
# datetime，timezone 的深入理解
```
# pytz 提供pytz.all_timezones_set提供了上海，没有北京，上海的STD时间就是会比UTC多出8小时6分钟
# pytz.country_timezones('cn') 查看这个国家的所有时区
localtz = pytz.timezone('Asia/Shanghai')  # <DstTzInfo 'Asia/Shanghai' LMT+8:06:00 STD>（Local Mean Time）
from datetime import datetime 

datetime.now() # 没有时区信息
>> datetime.datetime(2018, 3, 5, 11, 9, 21, 364845)

datetime.now(localtz) # 增加了时区信息,内部进行normalize转换
>> datetime.datetime(2018, 3, 5, 11, 9, 58, 592981, tzinfo=<DstTzInfo 'Asia/Shanghai' CST+8:00:00 STD>)

now = localtz.localize(datetime.strptime(now_str, '%Y-%m-%d %H:%M:%S')) # 接收字符串，输出时间带着时区
>> datetime.datetime(2018, 3, 5, 11, 9, 58, tzinfo=<DstTzInfo 'Asia/Shanghai' CST+8:00:00 STD>)

# 另一个方式，使用dateutil
from dateutil import tz
cn = tz.gettz('Asia/Shangshai')
ny = tz.gettz('America/New_York')
cn_dt = datetime.datetime(2020,4,17, 12, tzinfo=cn)
ny_dt = cn_dt.astimezone(tz=ny)
```
- 需要注意的概念：natvie time 和 active time
- natvie time 就是不带时区的时间，如datetime.datetime.now()，- datetime.datetime.utcnow()
- active time 是带时区的时间，如django.utils.timezone.now(),需提前指定USE_TZ
- 在django中USE_TZ=True则使用active time，输出为UTC时间，存到数据库中也是UTC时间；否则输出native time
- tip1：在django中USE_TZ=True和TIME_ZONE='Asia/Shanghai',用datetime.datetime.now()获取的时间由于不带时区，django会把这个时间转换成UTC-Asia/Shanghai时间，即东八区时间存到数据库
- tip2：只要设置了USE_TZ=True，django.utils.timezone.now()输出的都是UTC时间，不管TIME_ZONE是什么；如果USE_TZ=False，则django.utils.timezone.now()输出就和datetime.datetime.now()一样是一个native time，没有时区，不管TIME_ZONE设置的是什么；
- tip3：在django的模板显示时，如果设置里TIME_ZONE='Asian/Shanghai'，尽管数据库存储的是UTC时间，模板显示的时候，会转换到TIME_ZONE的本地直接进行显示；

# 15 Share session between Django and Flask
- Django 中是通过在 setting.py 文件中向 INSTALLED_APPS 列表中添加 ’django.contrib.session’
- 和 MIDDLEWARE_CLASSES 列表中添加 ’django.contrib.sesssion.middleware.SessionMiddleware’ 
- Flask 中 session 是在 cookie功能的基础上实现的在服务器端保存状态数据的一个功能
- 和cookie操作方式不同是在使用session时，要添加一个安全混淆密匙（app.sercet_key)  
#### (1) Django
```
Django中启用会话后; 每个HttpRequest对象将具有一个session属性
是一个类字典对象（request.session[“变量名”]=变量） 

set_expiry(value)：设置会话的超时时间，如果没有指定，则两个星期后过期 
get(key, default=None)：根据键获取会话的值（request.session.get(‘变量名’）） 
clear()：清除所有会话（request.session.clear()) 
flush()：删除当前的会话数据并删除会话的Cookie 
del request.session[‘member_id’]：删除会话 
```
#### (2) Flask
```
Flask中通过session[‘变量名‘]=变量 设置session值并保存在本地客户端cookie中，再通过session.get(’变量名‘)得到sessions值
```
#### (3) 引申理解
```
- HTTP协议本身是无状态的, 为了维护服务器和客户端之间的每一次通信的上下文信息
- cookie就是服务器存储在客户端浏览器的一组键值对

- 客户端首次进入一个网址时，服务器根据其请求信息判断该客户端本地没有cookies，服务器返回正常的response
- 然而在response的 header中加入类似于： Set Cookies: keyA:valueA,keyB:valueB;这样的信息
- 客户端浏览器收到该信息之后，将这一组键值对存储到本地中。以后再次登陆同一个网址时，在request header中发送cookies的键值对
- 浏览器在request header中发现键值对之后，就知道该浏览器之前也访问过本网站，然后就可以根据这些键值对的信息做出个性化的响应
- 当然这些响应中又可以包含新cookies设置的信息等。这样每一次 从浏览器都服务器通信都在 header中夹带cookies的方法，完成了客户端都浏览器的上下文通信。

 - session也是服务器将一部分信息存储在浏览器端，然后浏览器再次请求时携带这一部分信息供服务器查询
 - session的实现方式有好几种，只要能起到将所需信息在第二次之后的请求中发送给服务器的功能就可以了。 session的实现方式中有一种是采用 cookies的方式。
 - 在flask中， session可以认为是存储在cookies中的 键为 “session”的一个加密字符串， 然后这个字符串本身又是一个 键值对。
 - app.secret_key = 'A0Zr98j/3yX R~XHH!jmN]LWX/,?RT'  # SECRET_KEY， session 加密的密钥
 - flask中session是一个全局对象，就像request一样，每一个请求的上下文中都有一个 session 全局对象
 
 - 只要 session 在每一个请求中是隔离的，那只需要在 session 中存储 login=True 的键值对，那么只要 session 中有这样的键值对，即可以表明发起该次请求的用户是登录的
 - 每一次请求的 session 是隔离的，那么在 session 中存储 user_id=xxx，如果某两次请求的 user_id 是相同的，那么这两次请求的用户即是相同的。
 
 - 那手动实现 session_id 的功能也非常的简单，在存入 cookie 的时候，不直接存储存储加密之后的 session 
 - 而是使用 sha1 计算出 session 的哈希值，作为索引，然后将其存入 Redis 缓存中，然后将哈希值提供给前端
 - 在取出 cookie 的时候，通过哈希值来查找到 session 的内容，这样在传给前端页面的 cookie 也能够保证一个好看的，长度固定的 sid ，无法猜测，无法破解。
```
```
def qsort(a_list):
    if len(a_list) == 0:
        return a_list
    else:
        pivot = a_list[0]
        small = qsort([x for x in a_list[1:] if x < pivot])
        big = qsort([x for x in a_list[1:] if x >= pivot])
        return small + [pivot] + big
```
# django tips
### Base
```
from __future__ import absolute_import, print_function

import logging
import ujson as json

from django import http
from django.contrib.auth import authenticate
from django.contrib.auth import login as _login
from django.contrib.auth import logout as _logout

# from api.common import View
from easyshop import errors
from dash.views import PageInfoMixin

log = logging.getLogger('app')


class Base(object):

    NEED_LOGIN = True
    LOG_POST = True

    def __init__(self, **kwargs):
        for k, v in kwargs.iteritems():
            setattr(self, k, v)

    def _pre_routing(self, request):
        if self.NEED_LOGIN:
            user = request.user
            if not (user.is_authenticated() and user.is_active):
                resp = http.HttpResponse(u'用户未登陆')
                return resp

    def _authority_request(self, request):
    
        if not self.NEED_LOGIN:
            return True
        current_user = self.get_current_user(request)
        return bool(current_user)

    def _handle404(self, request):
        return http.HttpResponseNotFound('method not found')

    def _handle403(self, request):
        if request.is_ajax():
            return http.HttpResponse(u'无权访问', content_type='application/json')
        return http.HttpResponseForbidden(u'无权访问')

    def _pre_exec(self, request):
        if request.method == 'POST' and self.LOG_POST:
            log.info('%s#%s: %s', request.user.id, request.path, request.POST)

    def _exec_view(self, method, request, *args, **kwargs):
        return method(request, *args, **kwargs)

    def get_current_user(self, request):
        return request.user.id

    def __call__(self, request, *args, **kwargs):
        resp = self._pre_routing(request)
        if resp:
            return resp

        method = getattr(self, request.method, None)
        if not method:
            return self._handle404(request)

        auth = self._authority_request(request)
        if not auth:
            return self._handle403(request)

        self._pre_exec(request)
        try:
            resp = self._exec_view(method, request, *args, **kwargs)
        except:
            log.exception('unknown error: %s %s' % (request.get_full_path, request.body))
            resp = http.HttpResponse(u'服务器错误')
        return resp


class JSONView(Base, PageInfoMixin):

    def _pre_routing(self, request):
        if self.NEED_LOGIN:
            user = request.user
            if not (user.is_authenticated() and user.is_active):
                data = {
                    'errnum': errors.AuthorizationError.errno,
                    'errmsg': errors.AuthorizationError.errmsg,
                }
                return http.HttpResponse(json.dumps(data), content_type='application/json')

    def _exec_view(self, method, request, *args, **kwargs):
        try:
            data = super(JSONView, self)._exec_view(method, request, *args, **kwargs)
            data = {
                'errnum': 0,
                'errmsg': '',
                'data': data,
            }
        except errors.Base as e:
            data = {
                'errnum': e.errno,
                'errmsg': e.errmsg,
            }
        except Exception as e:
            log.exception(e)
            data = {
                'errnum': errors.Base.errno,
                'errmsg': errors.Base.errmsg,
            }
        return http.HttpResponse(json.dumps(data), content_type='application/json')

    def _handle404(self, request):
        data = {
            'errnum': 404,
            'errmsg': u'地址不存在',
        }
        return http.HttpResponse(json.dumps(data), content_type='application/json')

    def _handle403(self, request):
        data = {
            'errnum': 403,
            'errmsg': u'无权访问',
        }
        return http.HttpResponse(json.dumps(data), content_type='application/json')


class Login(JSONView):

    NEED_LOGIN = False
    TIME_OUT = 60*60*12

    def POST(self, request):
        username = request.POST.get('username', '')
        password = request.POST.get('password', '')
        if not (username and password):
            raise errors.InvalidArgument

        user = authenticate(username=username, password=password)
        if not user:
            raise errors.UsernamePasswordMismatch
        if not user.is_active:
            raise errors.AccountBanned

        _login(request, user)
        request.session.set_expiry(self.TIME_OUT)
        return {'uid': user.id}


class Logout(JSONView):

    NEED_LOGIN = True

    def POST(self, request):
        _logout(request)
        return {
            'user_name': None
        }

```
### PageInfoMixin
```
class PageInfoMixin(object):
    limit = 20

    def get_page_range(self, page_info, adjacent_pages=3):
        start = max(page_info.number - adjacent_pages, 1)
        end = min(page_info.number + adjacent_pages + 1,
                  page_info.paginator.num_pages + 1)
        page_range = [n for n in range(start, end)]
        return page_range

    def get_page_info(self, values, page):
        paginator = Paginator(values, self.limit)
        try:
            page_info = paginator.page(page)
        except PageNotAnInteger:
            page_info = paginator.page(1)
        except EmptyPage:
            page_info = paginator.page(paginator.num_pages)
        page_info.num_pages = paginator.num_pages
        page_info.page_range = self.get_page_range(page_info)
        return page_info
```
###  加入分页支持
```
class DealerList(JSONView):

    NEED_LOGIN = True

    def GET(self, request):
        data = request.GET
        try:
            page = int(data.get('page', 1))
        except (KeyError, ValueError, TypeError):
            raise errors.InvalidArgument
        dshl_sub_company = self.get_current_user(request)
        # FIXME 需要过滤掉已经不使用的店铺
        query = {
            'dshl_sub_company': dshl_sub_company,
        }
        dealer_ids = Dealer.objects.filter(**query).values_list('user_ptr_id', flat=True)
        total = dealer_ids.count()
        infos = []

        # 分页
        page_info = self.get_page_info(dealer_ids, page)
        for dealer_id in page_info.object_list:
            dealer_info = dealer_ctl.getDealerBasicInfo(dealer_id)
            rcdi = RequestCreateDealerInfo.objects.filter(dealer=dealer_id).first()
            infos.append({
                'dealer_id': dealer_info.get('id', ''),
                'dealer_name': dealer_info.get('shop_name', ''),
                'account': dealer_info.get('phone', ''),
                'dhb_account': dealer_info.get('phone', ''),
                'create_time': dealer_info.get('date_joined', ''),
                'dshl_sub_company': dshl_sub_company.name,
                'address': dealer_info.get('address', ''),
                'bd_phone': rcdi and rcdi.bd_phone or '',
            })
        return {
            'dealer_list': infos,
            'page_info': {
                'total': total,
                'current_page': page,
            }
        }
```

# 数据库读写分离，可能出现的问题; 与Django的多数据库路由有交叉
数据库进行主从分离后升级后，主库负责读操作、从库负责写操作；但是如果代码中的一个写操作之后就要读操作，那么简单点是问题会出现读不到数据
严重了之后，可能程序中的代码会崩掉，假设主从同步是几百毫秒，代码的执行肯定更快；可能是由于升级之前的代码会出现写完之后的立即读操作
1：在Django的数据库操作中间件中加入cookies信息，标记每一次的数据库操作，如果是阈值之内的读，那么还是走主库、否则走从库
2：每一次的数据库操作都在一个线程中执行
3: 有多个数据库时候，需要syncdb多次，每一次同步一个数据库; syncdb --database=dbase 默认同步的default
[来源与扩展](https://github.com/jbalogh/django-multidb-router)
### (1) settings.py 文件中添加多个数据库
```
DATABASES = {
    'default': 
    {
        'ENGINE': 'django.db.backends.mysql', 
        'NAME': 'default',                   
        'USER': 'xxx',                     
        'PASSWORD': 'xxx',                
        'HOST': 'localhost',               
        'PORT': '3306',     
    },
    'slave':
    {
        'ENGINE': 'django.db.backends.mysql', 
        'NAME': 'pinyin',                     
        'USER': 'xxx',                    
        'PASSWORD': 'xxx',               
        'HOST': 'localhost',                      
        'PORT': '3306',                     
    },    
             
```
### (2)  定义models指定属性
```
./manage.py syncdb --database=users # 会把所有的model都同步到users数据库
_database, 说明要连接的数据库，默认是default；与本次讨论的问题可能关系不大；只是做个扩展
```
### (3) 编写router
```
import random

class DataBaseRouter(object):

    def __init__(self):
        self.db_choices = {
            'default': 10,
            'slave': 90,
        }
        self._cnt = reduce(lambda x, y: x+y, self.db_choices.values())
        self.master_dbs = ['default']
        self.slave_dbs = ['slave']

    def db_for_write(self, model, **hints):
        return random.choice(self.master_dbs)

    def db_for_read(self, model, **hints):
        num = random.randint(1, self._cnt)
        for db, ratio in self.db_choices.iteritems():
            if num < ratio:
                return db
        return random.choice(self.slave_dbs)
```
### (4) 向settings.py中添加 DATABASE_ROUTERS
```
DATABASE_ROUTERS = ['path.to.DataBaseRouter']

# 补充知识：日志按天分组 settings.py
LOG_DIR = os.path.join(BASE_DIR, 'var')
today = datetime.datetime.now().strftime("%Y-%m-%d")
LOG_DIR = os.path.join(BASE_DIR, 'var/%s' % today)
if not os.path.exists(LOG_DIR):
    os.makedirs(LOG_DIR)
LOGGING{ # 配置不变
    'version': 1,
    'disable_existing_loggers': False,
    'filters': {},
    'formatters': {},
    'handlers': {},
    'loggers': {},
}
```
# 23 时区的问题 MySQL <=> MongoDB

from django.utils import timezone
tzinfo_1 = timezone.get_current_timezone()
输出：<DstTzInfo 'Asia/Shanghai' LMT+8:06:00 STD>

import pytz
tzinfo_2 = pytz.timezone('Asia/Shanghai')
输出：<DstTzInfo 'Asia/Shanghai' LMT+8:06:00 STD>

Django 中关于时区的配置：TIME_ZONE = 'Asia/Shanghai'  USE_TZ = True  
当为True后那么存入 MySQL 的时间都进行了时区操作

Order 的 deal_tiem 更新操作： timezone.datetime.now(timezone.get_current_timezone())

首先进来一个 time.time 的时间戳，time.tzname => ( 'CST', 'CST')
然后datetime.datetime.fromtimestamp(ts, tzinfo_2)  # 这里表示用了一个上海的时区
输出：datetime.datetime(2017, 11, 30, 21, 47, 10, 854104, tzinfo=<DstTzInfo 'Asia/Shanghai' CST+8:00:00 STD>)
Django 将这个时间存入了 MongoDB， 是一个 datetime.datetime 类型， 格式 'YYYY-MM-DD HH:MM:SS'
datetime.datetime 有一个属性 tzinfo， 但是这里没有了时区的属性

import calendar
calendar.timegm(Order.deal_time.timetuple())  => 恢复了存入Django数据库时候的8个小时的时间差

0 3 * * * python /var/www/easyshop/manage.py check_ucfpay_bill
0 */1 * * * 
格林威治标准时间 GMT 世界协调时间 UTC 北京时间 CST

flask mongo 中处理办法:没有根本解决
start_time += datetime.timedelta(hours=8, minutes=-6)
end_time += datetime.timedelta(hours=8, minutes=-6)


### 注意下面的这三步

```
import pytz
localtz = pytz.timezone('Asia/Shanghai') 
>> <DstTzInfo 'Asia/Shanghai' LMT+8:06:00 STD>  # 有时区，但是多加+6分钟

t = datetime.datetime.now()
>> datetime.datetime(2017, 12, 2, 0, 31, 24, 323199) # 没有时区

localtz.localize(t)
>> datetime.datetime(2017, 12, 2, 0, 31, 24, 323199, tzinfo=<DstTzInfo 'Asia/Shanghai' CST+8:00:00 STD>)  # 有时区，并且+8小时

datetime.now(localtz)
>> datetime.datetime(2017, 12, 2, 0, 47, 50, 603771, tzinfo=<DstTzInfo 'Asia/Shanghai' CST+8:00:00 STD>)  # 同上
```

### 注意二者之间的区别

```
datetime.strptime('2017/11/12', '%Y/%m/%d').replace(tzinfo=localtz, hour =8, minute=59, second=59)
>>> datetime.datetime(2017, 11, 12, 8, 59, 59, tzinfo=<DstTzInfo 'Asia/Shanghai' LMT+8:06:00 STD>)

localtz.localize(datetime.strptime('2017/11/12', '%Y/%m/%d').replace( ho ur=8, minute=59, second=59))
>> datetime.datetime(2017, 11, 12, 8, 59, 59, tzinfo=<DstTzInfo 'Asia/Shanghai' CST+8:00:00 STD>)

import datetime
import pymongo

client = pymongo.MongoClient('localhost', 27017)
db = client['test']
col = db['about_datetime']

col.insert_one({'test_time':datetime.datetime(2017,10,21,11,0,0)})
col.insert_one({'test_time':datetime.datetime(2017,10,21,11,10,0)})


{ "_id" : ObjectId("5a0da9a8ab78622e98a73cc4"), "test_time" : ISODate("2017-10-21T11:00:00Z") }
{ "_id" : ObjectId("5a0da9a8ab78622e98a73cc5"), "test_time" : ISODate("2017-10-21T11:10:00Z") }

import datetime

import pymongo

client = pymongo.MongoClient('localhost', 27017)
db = client['test']
col = db['about_datetime']

# col.insert_one({'test_time':datetime.datetime(2017,10,21,11,0,0)})
# col.insert_one({'test_time':datetime.datetime(2017,10,21,11,10,0)})


print(list(col.find({'test_time':{'$lt': datetime.datetime(2017,10,21,19,5,0)}})))

[{'_id': ObjectId('5a0da9a8ab78622e98a73cc4'), 'test_time': datetime.datetime(2017, 10, 21, 11, 0)}, {'_id': ObjectId('5a0da9a8ab78622e98a73cc5'), 'test_time': datetime.datetime(2017, 10, 21, 11, 10)}]
```
# 25 工作问题思考（星星分享）

```
### 问题：

我们有一个model是Order，其中有一个字段是create_time，也就是DateTimeField类型，但是现在想要按天来聚合一下，看看每天有多少订单。

### 解决办法：

```
from django.db import connection
select = {'day': connection.ops.date_trunc_sql('day', 'create_time')}
Order.objects.extra(select=select).values('day').annotate(count=Count('id'))
```

> 上面的查询语句转化成sql大概是这样的
>
> ```
> select count(id), DATE_FORMAT(deal_time, '%Y-%m-%d') days from order_order group by days;
> ```

### 坑：

> 在django的settings.py文件中，有一个变量
>
> ```
> USE_TZ=True
> TIME_ZONE = 'Asia/Shanghai'
> ```
>
> 如果USE_TZ设置成了True，那么django的ORM就会把时间转成UTC时间后存入mysql中。
>
> 在这种情况下使用上面的聚合方法就会出现问题。
>
> 举个例子：
>
> 我们的2017-10-02这一天的时间的UTC时间范围是：2017-10-01 16:00:00  — 2017-10-02 16:00:00
>
> 在使用上面的聚合时，2017-10-02这一天会被聚合成两天(2017-10-01,2017-10-02)
>
> 因此，只有在mysql中存的是你本地时间时，才可以使用此方法。
>
> 在django中可以通过创建项目后，把USE_TZ设置成False来完成此目的。
>
> 如果已经是咱们项目这种结果了，那么可以通过原始SQL方式进行聚合：
>
> ```
> select count(id), DATE_FORMAT(DATE_ADD(deal_time, interval 8 hour), '%Y-%m-%d') days from order_order group by days;
> ```
>
> 暂时只找到这么一种方式
```

补充：简书都打不开，，考虑更换dns，[成功解决](https://jingyan.baidu.com/article/f96699bb8e11ed894e3c1bc8.html)。
控制面板->无线网络连接->去掉ipv6的勾选->双击IPv4右键属性->指定dns：211.137.191.26；备用为218.201.96.130
# CORS（跨域资源共享，Cross-Origin Resource Sharing）

在服务器的response header中，加入“Access-Control-Allow-Origin: *”即可支持CORS

举个例子,以下过程就发生了跨域访问：
(1) API部署在DomainA上；
(2) Ajax文件部署在DomainB上，Ajax文件会向API发送请求，返回数据；
(3) 用户通过DomainC访问DomainB的Ajax文件，请求数据

```
# flask 可以中通过一个装饰器
import json
from werkzeug.wrappers import Response
from functools import wraps
from flask import json

def api_to_datav(func):
    @wraps(func)
    def _wrapper(*args, **kwargs):
        resp = func(*args, **kwargs)
        resp = Response(json.dumps(resp), status=200, mimetype='application/json')
        resp.headers["Access-Control-Allow-Origin"] = "*"
        return resp
    return _wrapper

# 进行制定
resp.headers["Access-Control-Allow-Origin"] = "http://datav.aliyun.com"
```