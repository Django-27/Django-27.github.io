# API网关概念
- API 网关，即API Gateway，是大型分布式系统中，为了保护内部服务而设计的一道屏障，可以提供高性能、高可用的 API托管服务;
- API 网关的作用就是把业务无关的功能剥离出来，让你的 API 只关心业务本身，与业务无关的功能全都丢给 API 网关;
- 从而帮助服务的开发者便捷地对外提供服务，而不用考虑安全认证控制、流量控制、审计日志、黑白名单等问题，统一在网关层实现;
- 网关可以提供API发布、管理、维护等主要功能;开发者只需要简单的配置操作即可把自己开发的服务发布出去，同时置于网关的保护之下;
- 闭源的各个大厂都有，开源的可以选中：apisix、kong、trk等


# tips
- py文件开头的(#!/usr/bin/python2.7),对python解释器进行了指定；（碰到一次yum用的python2，系统当时用的python是3.6，找不到yum包，进行指定后问题解决)
- where python 查看安装的Python有哪些；which python 查看当前使用的是哪个；更改系统默认的python，可以 cd /usr/bin  ->  rm python  -> ln -s python3.6m python  -> python -V
- linux下回有一个hash表，开机前是空，第一次执行SHELL命令会从PATH找，第二次就会从hash表找，即进行了缓存大大加快速率，hash  -r 清除 hash -l 查看(碰到在虚拟环境中安装ipython用的还是系统的ipython，hash -r 后解决)
- which pip 或 which ipython 打开对应的文件，发现都是对编译器进行了指定，这也就是修改第一行的作用；碰到系统有py3.6和py3.8但是pip指向了3.8,如果想往3.6安装ipython怎么都不可以，最后使用 /usr/bin/python3.6 -m pip install 安装进去了
- 拓展：you-get、httppie 都可以实现方便的download， httppie相对于curl返回的结果更加友好
- django2.2要求sqlite3是3.8以上的版本，升级sqlite3后外面检查没有问题，虚拟环境中没有同步过来，export LD_LIBRARY_PATH=/usr/local/lib 解决
- 替换django shell为ipython，pip install django-extensions、pip install ipython、django_extensions 添加到 INSTALLED_APPS ，settings 添加SHELL_PLUS = 'ipython'
- class Question(models.Model) 中 verbose_name='xx' 可以通过 from app_xx import models -> fields = models.Question._meta.fields 取出
- When I enter VIM I get The ycmd server SHUT DOWN (restart with ':YcmRestartServer') => python .vim/bundle/YouCompleteMe/install.py
- git diff 彩色显示 git config color.ui true
- python -m cProfile [-o output_file] [-s sort_order] myscript.py 或者 python -m memory_profile a.py 或 pip install guppy
- jupyter notebook –generate-config 生成 jupyter的配置文件 [Kernels for different environments](https://ipython.readthedocs.io/en/stable/install/kernel_install.html#kernels-for-different-environments)
- Python的内置函数：getattr()、hasattr()、delattr()和setattr()可实现基于字符串的反射机制;相当于把一个字符串变成一个函数名的过程;
- Python内置的@property装饰器可以把类的方法伪装成属性调用的方式。也就是本来是Foo.func()的调用方法，变成Foo.func的方式;
- super(子类名, self).方法名()，需要传入的是子类名和self，调用的是父类里的方法;
- 动态语言调用实例方法时不检查类型，只要方法存在，参数正确，就可以调用。这就是动态语言的“鸭子类型”，它并不要求严格的继承体系，一个对象只要“看起来像鸭子，走起路来像鸭子”，那它就可以被看做是鸭子。
- list(filter(lambda x: x%3==0, range(10))) -> [0, 3, 6, 9]
- list(map(lambda x: x%3==0, range(10)))    -> [True, False, False, True, False, False, True, False, False, True]
- 悲观锁:当查询某条记录时，即让数据库为该记录加锁，锁住记录后别人无法操作;（可通过设置过期时间，解决互斥锁问题）
-     select stock from tb_sku where id=1 for update;   ->   SKU.objects.select_for_update().get(id=1)
- 乐观锁:乐观锁并不是真实存在的锁，而是在更新的时候判断此时的库存是否是之前查询出的库存，如果相同，表示没人修改，可以更新库存，
- 否则表示别人抢过资源，不再执行库存更新 update tb_sku set stock=2 where id=1 and stock=7;   ->   SKU.objects.filter(id=1, stock=7).update(stock=2)
- django 批量创建  objs=[models.Book(title="图书{}".format(i+15)) for i in range(100)]  # 100条数据列表  models.Book.objects.bulk_create(objs)  # 一次批量插入
- 偏函数:当函数的参数个数太多，需要简化时;使用functools.partial可以创建一个新的函数，这个新函数可以固定住原函数的部分参数，从而在调用时更简单
- flask 通过g对象存储用户信息，再一次调用的任何地方都可以使用;g对象与request对象的session又不同；
- session对象是可以跨request的，只要session还未失效，不同的request的请求会获取到同一个session，但是g对象不是，g对象不需要管过期时间，请求一次就g对象就改变了一次，或者重新赋值了一次
- cpulimit -i -l 60 tar -czf 5555.tar.gz 5555 # 限制压缩解压对CPU100%的使用
- 对象方法自动传self，类方法自动传cls，from types import FunctionType,MethodType 判断方法类型 print(isinstance(func, FunctionType))



# 资产管理系统 CMDB (Configuration Management Database)
- CMDB用于存储与管理企业IT架构中设备的各种配置信息，它与所有服务支持和服务交付流程都紧密相联，支持这些流程的运转、发挥配置信息的价值，同时依赖于相关流程保证数据的准确性;
- MDB是ITIL(Information Technology Infrastructure Library，信息技术基础架构库)的基础，常常被认为是构建其它ITIL流程的先决条件而优先考虑，ITIL项目的成败与是否成功建立CMDB有非常大的关系;
- CMDB的核心是对整个公司的IT硬件/软件资源进行自动/手动收集、变更操作,即对IT资产进行自动化管理;(还可以有用户管理、权限管理、API安全认证、REST设计等等需求)

# 使用 oh-my-zsh  +  autojump
```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

```
查看系统有几种shell： cat /etc/shells
如果要切换回去bash：chsh -s /bin/bash

. /usr/share/autojump/autojump.sh

1 在数据库中添加一个目录: autojump -a [dir]

2 提升当前目录value数目的权重: autojump -i [value]

3 显示 autojump 数据库中的统计数据: autojump -s

# tip 岁月静好
```
# Linux查看文件夹大小
du -sh 查看当前文件夹大小
du -sh * | sort -n 统计当前文件夹(目录)大小，并按文件大小排序(sort 的更多参数，-r逆序) 
du -sh * | sort -n | wc -l # 统计个数(统计行数l， 统计字节数c，统计字符数m)
du -sh * | sort -n | tail -1 # 按照大小排序，返回最大的一个文件;同理也可以对sort -r逆序，使用head -1
```
合并多个自定的方法
``` pythonic
m3 = {k: v for d in [query, post, route] for k, v in d.items()}
```


# FTP 登录与查看目录下文件
```
from pay.unify_pay import UnifyPay
from ftplib import FTP

p = UnifyPay(mode=20)
s, u, p = p.unifypay.ucfpay.ftp_server, p.unifypay.ucfpay.ftp_user, p.unifypay.ucfpay.ftp_pass
ftp = FTP(s)
ftp.set_debuglevel(2)
ftp.login(u, p)
 
ftp.cwd('/')
ftp.nlst()
```
## from ftplib import FTP
```
def download_yesterday_bill(date, types, local_path='./'):
    # type data: str 20160114

    year, month = date[:4], date[4:6]

    ftp = FTP(FTP_SERVER)
    ftp.set_debuglevel(2)
    ftp.login(FTP_USER, FTP_PASS)
    buf_size = 1024

    remote_path = '/' + year + '/' + month  # /dt/bills/{}/{}/
    ftp.cwd(remote_path)

    filename_txt = '{}-{}-{}.txt'.format(date, types, FTP_USER)
    file_txt = open(local_path + filename_txt, 'wb')
    filename_ok = '{}-{}-{}.OK'.format(date, types, FTP_USER)
    file_ok = open(local_path + filename_ok, 'wb')

    try:
        ftp.retrlines('RETR {}'.format(filename_txt), file_txt.write, buf_size)
        ftp.retrlines('RETR {}'.format(filename_ok), file_ok.write, buf_size)
    except IOError:
        return 'No such file or directory {}'.format(filename_txt)

    ftp.quit()


if __name__ == '__main__':
    download_yesterday_bill('20160114', 'Pay')
# 一定要注意FTP服务器文件的格式，二进制还是TXT；
# 还有就是登陆时候的目录情况，是不是自己想到达的目录，确保目录正确（结合cwd和nlst和pwd）
```
### python 2.7 StringIo TypeError: unicode argument expected, got 'str'
```
from io import StringIO   --->      from io import BytesIO 
```

### 次用 StringIO 替换之前的文件下载
只需导入相应的包， 初始化file_txt = StringIO()
其他的都不变，最后得到的及时一个_io.ByteIo对象的 file_txt 实例, 试试 file_txt.getValue()
之前换行就是有问题，采用手动  ftp.retrlines('RETR {}'.format(filename_txt), lambda line: file_txt.write("%s\n" % line)) 增加一个 \n 的方式
StringIO是一个整体，可以使用 split（'\n')将行分开的办法，即还是一行一个数据.
### 常用函数
```
login(user='',passwd='', acct='')     登录到FTP 服务器，所有的参数都是可选的
# pwd()                       当前工作目录
# cwd(path)                   把当前工作目录设置为path
# dir([path[,...[,cb]])       显示path 目录里的内容，可选的参数cb 是一个回调函数，会被传给retrlines()方法
# nlst([path[,...])           与dir()类似，但返回一个文件名的列表，而不是显示这些文件名
retrlines(cmd [, cb])       给定FTP 命令（如“RETR filename”），用于下载文本文件。可选的回调函数cb 用于处理文件的每一行
retrbinary(cmd, cb[,bs=8192[, ra]])     与retrlines()类似，只是这个指令处理二进制文件。回调函数cb 用于处理每一块（块大小默认为8K）下载的数据。
storlines(cmd, f)   给定FTP 命令（如“STOR filename”），以上传文本文件。要给定一个文件对象f
storbinary(cmd, f[,bs=8192])    与storlines()类似，只是这个指令处理二进制文件。要给定一个文件对象f，上传块大小bs 默认为8Kbs=8192])
rename(old, new)    把远程文件old 改名为new
delete(path)     删除位于path 的远程文件
mkd(directory)  创建远程目录
```
### 更加 pythonic 的方式
```
FILENAME = 'StarWars.avi'

with ftplib.FTP(FTP_IP, FTP_LOGIN, FTP_PASSWD, FTP_TIMEOUT):
    ftp.cwd('/whyfix/movies/')
    with open(FILENAME, 'wb') as f:
        ftp.retrbinary('RETR %s' %  FILENAME, f.write, 1024)
```
### 涉及到目录操作的时候可以参考一下；还没有上机验证
```
def DownLoadFileTree( self, LocalDir, RemoteDir ):
    if os.path.isdir( LocalDir ) == False:
        os.makedirs( LocalDir )
    self.ftp.cwd( RemoteDir )
    RemoteNames = self.ftp.nlst()  
    for file in RemoteNames:
        Local = os.path.join( LocalDir, file )
        if self.isDir( file ):
            self.DownLoadFileTree( Local, file )                
        else:
            self.DownLoadFile( Local, file )
    self.ftp.cwd( ".." )
    return
```
#### 碰到的一个问题，没哟换行，已解决
```
ftp.retrlines('RETR {}'.format(filename_txt), lambda line: file_txt.write("%s\n" % line))

```
#### Git添加文件时出现了一个warning:LF will be replaced by CRLF

```
git config --global core.autocrlf  false
# curl -I www.baidu.com
```
# 将经纬度转化为弧度
```javascript
def rad(d):
    return d * math.pi / 180.0

def get_distance(dealer_id, order_id):
    """计算订单的配送距离"""
    distance = 0
    oc = OrderControl.objects.filter(order_id=order_id).first()
    if oc:
        dealer = Dealer.objects.filter(pk=dealer_id).first()
        d_lat, d_lon = rad(dealer.latitude), rad(dealer.longitude)
        o_lat, o_lon = rad(oc.lat), rad(oc.lon)
        a = d_lat - o_lat
        b = d_lon - o_lon
        s = 2 * math.asin(math.sqrt(math.pow(math.sin(a / 2), 2) + math.cos(
            d_lat) * math.cos(o_lat) * math.pow(math.sin(b / 2), 2)))
        earth_radius = 6378.137
        distance = s * earth_radius
    return abs(distance)
```
# 一清 二清； 进件 退件
一清：一次清算，消费者的付款直接通过银联或第三方支付平台打到商家绑定的银行卡
二清：二次清算，中间通过了‘其他人’

进件：将资料准备好后提交给贷款公司或者银行的系统，又他们完成审核
退件：提交资料后不想办理了，把资料拿回来的过程叫退件。
# 排查支付接口慢的原因
traceroute 116.213.95.221
proxies = {'https': 'http://123.206.51.174:6668'}
先锋接口慢，可能是双方机房的问题；使用阿里云进行接口模拟请求，先锋和阿里的请求很快；采用给请求增加代理的方式，有效的降低了和先锋的通讯时间。
```
r = requests.post(ucfpay_wx.GATEWAY_URL, data=query, proxies={'https': 'http://123.206.51.174:6668'})
r.elapsed
```
![image](https://github.com/Django-27/workspace/blob/master/pic/%E6%8E%92%E6%9F%A5%E6%94%AF%E4%BB%98%E6%8E%A5%E5%8F%A3%E6%85%A2%E7%9A%84%E5%8E%9F%E5%9B%A0.png?raw=true)


更新：
- 先锋方面的问题是由于‘机房’导致延迟；我们这边加入代理，代码进行优化，发短信的异步任务没有delay，给mongo_bi入数据耗时长
- 将生成订单和请求二维码图片分为两个部分，前端拿到创建的订单信息即进行展示（100ms），之后在通过订单编号去请求二维码地址（2s），
- 这样体验上会感觉速度很快（是否可以去掉正扫，都进行反扫，避免跑单）

# 2 Pycharm 远程调试，Flask, Django, sftp, remote debug

```
# pycharm 中的 Requested setting CACHES, but settings are not configured. You must either define the environment variable DJANGO_SETTINGS_MODULE 问题解决

- Run  -->  EditConfigures 
- 修改里面的Environment variables 添加一项, 名称是DJANGO_SETTINGS_MODULE 值是settings,比如 easyshop.settings

- 还有就是碰到了一个坑：无论怎么是就是sftp失败，但是在shell中可以；原来时候版本 2017.3.4 的原因，重新按了一个版本直接就好了
- pycharm 插件： [ideavim](https://plugins.jetbrains.com/pluginManager/?action=download&id=IdeaVIM&build=PY-173.4674.37&uuid=e41ecbc6-6362-4ff3-aa44-41574217b412),  [markdown](https://github.com/vsch/idea-multimarkdown), [bashsupport](https://www.plugin-dev.com/project/bashsupport/) 
```
![image 2018-03-13](https://github.com/Django-27/workspace/blob/master/pic/pycharm-deployment.png)

# 6 supervisor ctl 使用tips
- 首先有一个配置文件 /etc/supervisord.conf  # 新建日志文件夹和文件，安装确实的包pip install gunicorn_thrift
- 然后添加一个代码看， 时候 supervisorctl update
- 最后，进入 supervisorctl 执行 restart all
- 备注：easyshop 中 thrift ，调试时通过 ./manage.py thrift_server (easyshop/rpc/server.py)启动； 正式环境通过 supervisorctl 进行启动
![image 2018-03-19](https://github.com/Django-27/workspace/blob/master/pic/supervisor-ctl.png)

- Thrift支持多线程的server用于提高并发处理能力
- 对于高并发后台服务，还需要考虑：集群下如何部署启、- 服务挂了如何自动恢复以保持可用
- client端如何找到server地址及端口，也就是服务发现的问题

### (1) Linux (Ubuntu) 添加新用户
```
adduser   qiyuan  # 只会添加一个用户，没有创建它的主目录，除了添加一个新用户之外什么都没有
userdel qiyuan  # -r 删除目录
# useradd 没有自动化的配置，更多需要参数指定

- 创建用户时调用的配置，/etc/adduser.conf
- 创建用户主目录时默认在/home下，而且创建为 /home/用户名  
```
### (1-2) [SwitchHosts](https://github.com/oldj/SwitchHosts)
SwitchHosts是一个管理、快速切换Hosts小工具，开源软件，一键切换Hosts配置，非常实用，高效![image](https://raw.githubusercontent.com/oldj/SwitchHosts/master/screenshots/sh_light.png)
### (2)git log 无法正常显示中文
```
git config --global core.pager more
# 捣鼓了一上午，有时候可能先去做别的事，再回来弄，一下就弄好了

还有就是git log 正常，git commit 的时候乱码；通过~/.gitconfig 添加
[core]
    editor = "vim"
```

### (3)nginx location 中的一些使用
- ~ 波浪线表示执行一个正则匹配，区分大小写
- ~* 表示执行一个正则匹配，不区分大小写
- @  #"@" 定义一个命名的 location
```
try_files: 找指定路径下文件，如果不存在，则转给哪个文件执行
rewrite:指uri重写

location  = / { # 前面加了=， 表示普通字符的精确匹配，只匹配"/".
  [ configuration A ] 
}
location  / { # 匹配任何请求，因为所有请求都是以"/"开始，但是更长字符匹配或者正则表达式匹配会优先匹配
  [ configuration B ] 
}
location ^~ /images/ { # ^~表示普通字符匹配， 匹配任何以 /images/ 开始的请求，并停止匹配 其它location
  [ configuration C ] 
}
location ~* \.(gif|jpg|jpeg)$ { # 匹配以 gif, jpg, or jpeg结尾的请求，但是所有 /images/ 目录的请求将由 [Configuration C]处理
  [ configuration D ] 
}
```
- root 配置段：http、server、location、if
- root的处理结果是：root路径＋location路径
- alias 配置段：location
- alias的处理结果是：使用alias路径替换location路径
- alias后面必须要用“/”结束,root 则可有可无

### (4)备注：
- git blame 查看每一行的commit
- PaaS (Platforms as a Service) :衡量标准，标准、价格、稳定性、服务、扩展性、文档、地点、公司、保持环境一致、自动化部署、测试、备份
#### (5)负载均衡
- 负载：就是后端系统的承载能力；（比如同等条件下，一个1核cpu-1G内存的机器的承载能力一般会比8核cpu-8G内存的机器要差；相同配置下，一个cpu利用率为80%的机器比30%的承载能力一般要差等等）
- 均衡：保证后端请求的平衡；（比如在同等情况下，分配到多台机器的请求要相当；有些情况下，同一用户尽可能分配到同一台机器等等）
- 负载均衡的算法实际上就是，在考虑后端机器承载情况的前提下，保证请求分配的平衡和合理
- 算法一：手工配置，对于中小系统来讲是最有效最稳定的。因为后端机器的性能配置、上面部署了哪些服务、还能有多大的承载能力等等，我们是最清楚的。那我们在配置的时候，就可以明确的告诉调用者，你只能分配多大的压力到某台服务器上，多了不行！这种方式配置简单，而且很稳定，基本不会产生分配的抖动;缺点是，不能动态调整
```
upstream simplemain.com {
     server  192.168.1.100:8080 weight=30;
     server  192.168.1.101:8080 weight=70;
     
     # location 对URL进行匹配.可以进行重定向或者进行新的代理 负载均衡
}
```
- 算法二：动态调整，这种方案会根据机器当前运行的状态和历史平均值进行对比，发现如果当前状态比历史的要糟糕，那么就动态减少请求的数量。如果比历史的要好，那么就可以继续增加请求的压力，直到达到一个平衡；极端情况可能会造成系统雪崩
- 更好的方案，将两者结合。一方面静态配置好承载负荷的一个范围，超过最大的就扔掉；另一方面动态的监控后端机器的响应情况，做小范围的请求调整
- 算法三：均衡算法，经常会用到以下四种算法：随机（random）、轮训（round-robin）、一致哈希（consistent-hash）和主备（master-slave）
```
upstream simplemain.com {
     ip_hash;  # 按ip做hash算法，然后分配给对应的机器
     fair;  # 按后端服务器的响应时间来分配请求，响应时间短的优先分配
     server  192.168.1.100:8080;
     server  192.168.1.101:8080;
     server 127.0.0.1:9090 down;  # down 表示单前的server暂时不参与负载
     server 127.0.0.1:8080 weight=2; # 默认为1.weight越大，负载的权重就越大
     server 127.0.0.1:7070 backup;  # backup： 其它所有的非backup机器down或者忙的时候，请求backup机器,压力最轻
     
}
```


### (6)Django 备注
-  CBVs (Class-based views)，是Django为解决建站过程中的常见的呈现模式而建立的
-  FBVs (Function-based views)，在Django发展的前期阶段, function-based views被开发人员广为使用
-  FBVs 编写原则：应尽量避免内套的"if"逻辑
-  CBVs 编写原则：保持mixin简单明了
-  共同原则：代码越少越好；永远不要重复代码；View应当只包含呈现逻辑, 不应包括业务逻辑；保持view逻辑清晰简单；可以用作403, 404, 500的错误处理程序
-  在编程中mixin是指为继承它的class提供额外的功能, 但它自身却不能单独使用的类. 在具有多继承能力的编程语言中, mixin可以为类增加额外功能或方法. 在Django中, 我们可以使用mixin为CBVs提供更多的扩展性
-  mixin永远继承自Python的object类型
# 有关支付的通知和回调

正扫走的是通知，反扫时轮询，正扫成功后，想极光发一个推送， 极光负责向收银机发送通知
在表 Notification.TYP_DEALER_CASHER_PAYED 中记录一条相关的记录， jpush.JPush 

# 先锋支付，提现和订单相关的兼容问题（案例）
```
# 开始代发请求
2018-03-23 09:16:19,269  单笔代发查询 加密前  ONLINE-ORDER-WITHD-5846P5858

# 通知到达的时间
2018-03-23 00:08:04,543 先锋回调解密后的数据:  u'status': u'S', u'resCode': u'00000', u'merchantNo': u'ONLINE-ORDER-WITHD-5846P5858'

# 代发请求，同步接口返回时间；status 两次不一致
2018-03-23 00:08:04,925   请求返回数据：u'status': u'I', u'resCode': u'00002', u'merchantNo': u'ONLINE-ORDER-WITHD-5846P5858'
```
- 处理提现的时候，同步请求接口有三种可能：成功S、处理中I和失败F，对应进行处理
- 此次问题是支付的通知先于同步接口进行了返回，但是代码没有兼容这种情况
- 并且, 同步接口的返回结果是 I，支付通知是 S
- 造成在同步接口的后续代码中对状态进行了重写
- 解决是首先确定订单状态，相关表的受影响情况，最后发现其他相关表都正常处理
- 只有WithdrawalsRecord 和 WithdrawalsProcessRecord 的状态被影响了
- 采用更改状态，不能进行接口的重新请求，否则相关表、短信的都会在来一遍
![image order-pay.png](https://github.com/Django-27/workspace/blob/master/pic/order-pay.png)

- 还有就是：Order_Order, Account_Bill, Account_DealerBalanceLog
- 订单-账单-资金变动记录， 前者通过 dealer_id进行关联，后者通过 related 进行关联
- 通过这个过程状态的确认，就可以说明，订单成功的，账单一定有，钱一定加到了账户里
- 有可能商户点击了‘取消’，那么当时在收银机的订单列表是不展示的，但是后续订单支付成功了
- 我们还是会将订单的状态进行变更，此时在收银机订单中应该是可以看到的，没有问题。
![image order-pay-mysql](https://github.com/Django-27/workspace/blob/master/pic/order-pay-mysql.png)

- py2 vs py3 main diff:
- （1）print 由语句变为了函数 （2）整数相除（3）Unicode（4）异常处理，py3只有as e 这种写法（5）xrange（6）map函数，py2返回list，py3是一个iterator（7）py3字典不在支持has_key，只有in



# 短信预警：生成二维码失败
![image 2018-03-29](https://github.com/Django-27/workspace/raw/master/pic/qrcode_err_message_notify.png)
# 临时查看一个调用需要的时间
```
import datetime, time  # py2.7

t1 = datetime.datetime.fromtimestamp(time.time())
···
t2 = datetime.datetime.fromtimestamp(time.time())
print('\ntime used: %s ms\n' % ((t2 - t1).microseconds / 1000))
```

# 分析日志：api 发出到回来的时间间隔，并可视化展示
![181](https://github.com/Django-27/workspace/blob/master/pic/api_time.png)
```
#! /usr/bin/env python
# -*- coding: utf-8 -*-
import re
from matplotlib import pyplot as plt


from pylab import *  # 解决中文是方块的问题
mpl.rcParams['font.sans-serif'] = ['SimHei']
mpl.rcParams['axes.unicode_minus'] = False

f = open('work_old_pay.log')
x, y, index = [], [], 0
while 1:
    line_1 = f.readline()
    p1 = re.compile('pay INFO(.*?)wxpay_api.py#107')
    p4 = re.compile('pay INFO(.*?)wxpay_api.py#51')
    # p1 = re.compile('pay INFO(.*?)alipay_api.py#110:')
    # p4 = re.compile('pay INFO(.*?)alipay_api.py#36:')

    p_time = re.compile(r'\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2},\d{3}')
    if p1.findall(line_1):
        line_2 = f.readline()  # 统一下单(正扫)前
        line_3 = f.readline()
        line_4 = f.readline()  # 微信支付回调

        if p4.findall(line_3):
            start_time = p_time.search(line_1).group(0)
            end_time = p_time.search(line_3).group(0)

            s1 = datetime.datetime.strptime(start_time, '%Y-%m-%d %H:%M:%S,%f')
            e1 = datetime.datetime.strptime(end_time, '%Y-%m-%d %H:%M:%S,%f')

            diff_microsecond = ((e1 - s1).microseconds / 1000)
            print(start_time, end_time, diff_microsecond)
            # print(('%s %s %s %s %s %s %s \t %s') % (start_time, s_second, s_milsecond, end_time, e_second, e_milsecond, total_milsecond, index))
            x.append(index)
            y.append(diff_microsecond)
            index += 1
    if not line_1:
        break

begin, end = 0, -1
# plt.plot(x[begin:end], y[begin:end])  # 折线图
# plt.bar(x, y)  # 柱状图
plt.scatter(x, y)  # 散点图
# plt.subplot(121)
# plt.subplot(122)
plt.title(u'统计原生支付-微信正扫，api发出请求到收到回调的时间( 单位/毫秒) ')
plt.xlabel(u'总共记录数 %s 选取统计的范围 [%s:%s]' % (len(x), begin, end))
plt.ylabel(u'毫秒')
plt.grid()
plt.show()
print(index)
"""
原生-微信-正扫
1 pay INFO 2017-10-27 11:58:03,801 /var/www/easyshop/pay/sdk/ucfpay_sdk/ucfpay_api.py#59:
2 正扫 加密前：
3 {'transCur': 156, 'scanType': 'PSCAN', 'returnUrl': 'http://yd.huikaixin.cn/pay/notify/order/ucfpay/', 'source': 2, 'bizType': 200, 'merchantType': 'OUT', 'subMerchantId': 'ds01qiyuan', 'payType': 'ALIPAY', 'noticeUrl': 'http://yd.huikaixin.cn/pay/notify/order/ucfpay/', 'productName': 'pay', 'subMerchantName': '\xe6\x83\xa0\xe5\xbc\x80\xe5\xbf\x83\xe4\xbe\xbf\xe5\x88\xa9\xe8\xb4\xad', 'outOrderId': 'ONLINE-ORDER-NATIVE-41358', 'amount': 800, 'bankId': 'ALIPAY', 'failUrl': '', 'methodVersion': '1.0', 'payModeType': 'SCANPAY', 'isLimitCreditPay': 0, 'bizProduct': 103}
4 pay INFO 2017-10-27 11:58:04,665 /var/www/easyshop/pay/sdk/ucfpay_sdk/ucfpay_api.py#384:
5 请求返回数据：
6 {u'status': u'00', u'respMsg': u'\u6210\u529f', u'payInfo': u'http://pay.ucfpay.com/finExchange/qrCode?imageUrl=https%3A%2F%2Fqr.alipay.com%2Fbax09419l1lolujwoen26002'}

原生-支付宝-正扫
pay INFO 2017-09-21 10:00:58,408 /var/www/easyshop/pay/sdk/alipay_sdk/alipay_api.py#110:
扫码支付:{'timestamp': '2017-09-21 10:00:58', 'charset': 'utf-8', 'app_id': '2017051207217101', 'biz_content': '{"out_trade_no": "ONLINE-ORDER-NATIVE-2198", "extend_params": {"sys_service_provider_id": "2088721013729525"}, "total_amount": 0.01, "timeout_express": "30m", "subject": "pay"}', 'sign': 'bA+umNLa0kLEzXqPceq+4mzkTI3tQbHDlvdUDPX+sLsd/JRdyLVcK8PjlnVZ2DiGcDdB2/EpZgC2eroVRYaoRge8WIC0QvGtyPlpXQn7vETDXKUa6eHw1pLuRzcGhvLY+iIS+3WmOS/g1Ghuv97bYw4hoGrWUmc1YYROgdKfgII=', 'app_auth_token': '201705BB1ab1e7a0f6b14cffab6a7cef6c472X84', 'version': '1.0', 'notify_url': 'http://yd.huikaixin.cn/pay/notify/order/alipay/', 'sign_type': 'RSA', 'method': 'alipay.trade.precreate'}
pay INFO 2017-09-21 10:00:58,581 /var/www/easyshop/pay/sdk/alipay_sdk/alipay_api.py#36:
支付宝请求返回数据:{u'alipay_trade_precreate_response': {u'msg': u'Insufficient Token Permissions', u'sub_code': u'aop.invalid-app-auth-token', u'code': u'20001', u'sub_msg': u'\u65e0\u6548\u7684\u5e94\u7528\u6388\u6743\u4ee4\u724c'}, u'sign': u'oH6hfZAMaRu14Qz3Hj9NxAo+nPNpf1HhD0m4xCt+NbndnEUnbzFSzfv/76PkzmeEFhs158TIuHW86JIEUQzpzLikt9M8B8j79oZw5PWL6npi3CJo3/jLUd0046dCTZ+t3goR3V3wNKGx52P0Os57P3wiq8NJ7xrUD9OjWeU6MQ0='}
"""
```
# 用Navicat SSH安全连接MySQL
![132](https://github.com/Django-27/workspace/blob/master/pic/navcat_1.png)
1.    新建一个连接，选择SSH标签，设置SSH登录的信息
2.    回到General设置数据库登录信息(SSH+常规的方式，常规中设置的是数据库的信息，localhost，3306，要登录的数据库，要登录的数据库密码)
3.    重要：连接测试环境的数据库，唯一的不同地方是讲本地的公钥上传，验证时使用本地的私钥
4.  线上的服务器托管到了其他的地方，ssh中用域名而不是ip这样即使ip变了但是域名是不经常变得；常规中主机也是填写域名而不是localhost
表示只允许这个域名进行访问连接
5.  window中本直接采用默认就可以连接并看到（localhost 3306 root 密码为空）

![image](https://github.com/Django-27/workspace/blob/master/pic/navcat_2.png)
MySQL的自动补全和语法高亮工具MyCli
```
    pip install mycli
    mycli -uroot -p -D test  -h 192.168.16.150 // 直接mycli后输入秘密进行，也可以指定后面的参数进去
```
# ssh 无密码连接
1 ssh-keygen
```
ssh-keygen -t rsa -C "nfs_fly@outlook.com"
```
2 将公钥写入你希望无密码进入机器的用户目录下: 
```
cat  id_rsa.pub  >> ~/.ssh/authorized_keys
# 有时需要设置目录权限:chmod 600 authorized_keys 或 chmod 700 -R .ssh
```
1 填写配置文件.ssh/config
```
Host dev
    HostName 192.168.4.141
    Port 22
    User weiqiyuan 
Host test
    HostName 192.168.2.10
    port 42022
    User weiqiyuan
```
# 22 升级 vim 到 8.0  Vim -> Youcompleteme 
```
centos 编译过一次yum，拷贝到mac，需要重新编译,最后用brew在python3的虚拟环境中安装顺利解决
brew info vim ; brew install vim 
brew link --overwrite vim

# “Python quit unexpectedly” when launching vim with YouCompleteMe
# brew uninstall --ignore-dependencies --force python python@2
# unset PYTHONPATH
# brew install python python@2
# cd ~/.vim/bundles/YouCompleteMe
./install.sh --all
```
通过软连接指定python版本
```
whereis python
which python
rm /usr/bin/python                      
ln -s /usr/local/bin/python3.8 /usr/bin/python  # -s 创建软连接，类似windows中的快捷方式；不指定-s则创建硬链接是一个指针或者说是文件的引用或别名，单独删除一个硬连接原始文件还在
```
[从第七步开始](https://blog.csdn.net/Bobdragery/article/details/100665639)
```
# 第十步异常，
➜  YouCompleteMe git:(master) ✗ ./install.py  --clang-completer  --gocode-completer --tern-completer
Searching Python 3.8 libraries...
ERROR: found static Python library (/usr/local/lib/python3.8/config-3.8-x86_64-linux-gnu/libpython3.8.a) but a dynamic one is required. You must use a Python compiled with the --enable-shared flag. If using pyenv, you need to run the command:
  export PYTHON_CONFIGURE_OPTS="--enable-shared"
before installing a Python version.

# 修改后的第十步，也就是制定一些当前的python版本
➜  YouCompleteMe git:(master) ✗ /usr/bin/python3 ./install.py  --clang-completer  --gocode-completer --tern-completer

```
七步之前的就是下面的自己重新编译vim，为了使vim加入python3支持
```
sudo yum install epel-release
sudo yum install gcc-c++ ncurses-devel ruby ruby-devel lua lua-devel luajit luajit-devel ctags python python-devel python3 python3-devel tcl-devel perl perl-devel perl-ExtUtils-ParseXS perl-ExtUtils-XSpp perl-ExtUtils-CBuilder perl-ExtUtils-Embed cscope gtk3-devel libSM-devel libXt-devel libXpm-devel libappstream-glib libacl-devel gpm-devel

yum list installed | grep -i vim
# sudo yum erase vim-common.x86_64 vim-enhanced.x86_64 vim-filesystem.x86_64 vim-X11

# sudo depends on vim-minimal
sudo rpm -e --nodeps vim-minimal

sudo ln -s /usr/bin/python3.8 python3

git clone https://github.com/vim/vim.git
cd vim
./configure --with-features=huge \
            --enable-multibyte \
            --enable-rubyinterp=yes \
            --enable-python3interp=yes \
            --with-python3-config-dir=$(python3-config --configdir) \
            --enable-perlinterp=yes \
            --enable-luainterp=yes \
            --enable-gui=gtk2 \
            --enable-cscope \
            --prefix=/usr/local
make
make install
```
```
sudo add-apt-repository ppa:jonathonf/vim
sudo apt update
sudo apt install vim

sudo apt remove vim
sudo add-apt-repository --remove ppa:jonathonf/vim

sudo apt-get remove vim-common  # 卸载系统的vim
sudo apt-get install vim-nox-py2  # 支持python和python3的切换
sudo update-alternatives --config vim  # 切换命令

git clone --recursive https://github.com/Valloric/YouCompleteMe.git
# 报错 Unable to find current revision in submodule path 'third_party/requests'
# 报错 Failed to recurse into submodule path 'third_party/ycmd'
rm -fr third_party/
git submodule update --init --recursive

[http://www.jianshu.com/p/d908ce81017a?nomobile=yes](http://www.jianshu.com/p/d908ce81017a?nomobile=yes)
[https://vimawesome.com/](https://vimawesome.com/)

# 构建 ycm_core
cd ～/.vim/bundle/YouCompleteMe
mkdir ~/.ycm_buil && cd ~/.ycm_build
cmake --build . --target ycm_core --config Release    

# 复制 .ycm_extra_conf.py 文件

# cd 的进化版吧
 sudo apt-get install autojump  # 安装
 source /usr/share/autojump/autojump.sh on startup
 echo '. /usr/share/autojump/autojump.sh' >> ~/.bashrc  # 自启动
 j --stat  # 查看权重文件
```
# 30 datav 大屏显示的升级
之前的设计是每一项的统计都分为两个部分，第一部分是本小时的数据，通过唯一的Key由redis中取得，第二部分通过聚合mongodb查找；问题是还是对mongodb的压力太大

改进版，希望尽可能的将数据由redis中取得，比如之前 redis 是一小时已清除，现在改为一天已清除；如果是月环比，也是一天一个计算结果，并且将结果以一种唯一的 Key 存入 redis
```
[{'dealer_name': 1, 'price': 21}, {'dealer_name': 1, 'price': 3}, {'dealer_name': 2, 'price': 5}]
# 两个复杂字典相加，通过这种方式实现了复杂字典的内容求和
dealer = defaultdict(int)
for item in today_price:
    dealer[int(item['dealer_id'])] += int(item['price'])
ret_price = [{'dealer_id': d, 'price': p} for d, p in dealer.items()]
```

# 31 Axure RP Extension for Chrome下载与安装方法
```
1、首先把需要安装的第三方插件，后缀.crx 改成 .rar，然后解压，得到一个文件夹

2、再打开chrome://extensions/谷歌扩展应用管理，点击右上角的开发者模式，就可以看到“加载正在开发的扩展程序”这一选项。

3、选择刚才步骤1中解压好的文件夹，确定

4、确认新增扩展程序，点击添加，成功添加应用程序。
```
# 20 Linux 踢掉用户：
```
who 或 w
who am i
pkill -kill -t pts/22   
```
# 18 vagran Django 使用
```
端口映射模式：将 vagrant 中的端口，映射为电脑的8001端口
config.vm.network :forwarded_port, guest: 8000, host: 8001

启动 django 的方式要注意()
./manage.py runserver 0:8000  =>   浏览器前访问地址：http://127.0.0.1:8001/admin/

第一次的登录后台 admin 的账户
./manage.py createsuperuser  # 先创建


```

# 9 HTTP 中 GET 与 POST 的区别
get用于获取数据，post用于提交数据
GET把参数包含在URL中，POST通过request body传递参数

GET的语义是请求获取指定的资源。GET方法是安全、幂等、可缓存的（除非有 Cache-ControlHeader的约束）,GET方法的报文主体没有任何语义。

POST的语义是根据请求负荷（报文主体）对指定的资源做出处理，具体的处理方式视资源类型而不同。POST不安全，不幂等，（大部分实现）不可缓存。为了针对其不可缓存性，有一系列的方法来进行优化，以后有机会再研究（FLAG已经立起）。

还是举一个通俗栗子吧，在微博这个场景里，GET的语义会被用在「看看我的Timeline上最新的20条微博」这样的场景，而POST的语义会被用在「发微博、评论、点赞」这样的场景中。


方法 | Get | Post
---|---|---
后退/刷新 | 无害 | 数据会被重新提交（浏览器告知用户）
书签 | 可收藏 | 不可收藏为书签
缓存 |能被缓存 |  不能缓存 
编码类型 | application/x-www-form-urlencoded | application/x-www-form-urlencoded 或 multipart/form-data。为二进制数据使用多重编码
历史 | 参数保留在浏览器历史中 | 参数不会保存在浏览器历史中
安全性 | 与 POST 相比，GET 的安全性较差，因为所发送的数据是 URL 的一部分。在发送密码或其他敏感信息时绝不要使用 GET ！ | POST 比 GET 更安全，因为参数不会被保存在浏览器历史或 web 服务器日志中。 

对数据类型的限制HTTP协议并没有限制URI的长度，具体的长度是由浏览器和系统来约束的

## HTTP 和 HTTPS 的区别
相同点:
大多数情况下，HTTP 和HTTPS 是相同的，因为都是采用同一个基础的协议
不同点:
HTTP 的URL 以http:// 开头，而HTTPS 的URL 以https:// 开头
HTTP 是不安全的，而 HTTPS 是安全的
HTTP 标准端口是80 ，而 HTTPS 的标准端口是443
在OSI 网络模型中，HTTP工作于应用层，而HTTPS 工作在传输层
HTTP 无法加密，而HTTPS 对传输的数据进行加密
HTTP无需证书，而HTTPS 需要认证证书(SSL数字证书)
HTTP主要有这些不足：
通信使用明文,内容可能被窃听
不验证通信方身份,因此有可能遭遇伪装
无法验证报文的完整性,所有有可能已篡改
####　HTTP + 加密 + 认证 + 完整性保护 = HTTPS
HTTPS是身披SSL外壳的HTTP,通常情况下HTTP是直接和TCP层进行通信的。当使用SSL(安全套阶字)时,则演变成HTTP先和SSL通信,SSL再和TCP通信的了
加密技术：讲解SSL前,科普一下加密方法,SSL采用的是一种叫做公开密钥加密的加密处理方式（只要拿到秘钥就能破解密码）；对称加密:加密和解密用的一个密钥的方式称为对称加密,也叫做共享密钥加密；对称加密在发送加密信息时也需要将密钥发送给对方,但这样可以被攻击者截取,就不安全啦；非对称加密：非对称加密又称作公开密钥加密,它很好的解决了对称加密密钥被截取的问题。非对称加密采用一对非对称的密钥,一把叫做私有密钥,一把叫做共有密钥。使用非对称加密,发送密文一方使用对方的共有密钥进行加密处理,对方收到加密信息后,再使用自己的私有密钥进行解密；HTTPS采用对称加密和非对称加密所混合的加密机制。若密钥能安全交换,那么有可能仅考虑非对称加密。但是非对称加密与对称加密相比,处理速度相对较慢。公开密钥的认证：使用数字证书认证机构和其颁布的公开密钥证书进行认证。即让第三方独立机构进行验证。有密钥是保存在服务器端的～注意??：认证是要钱的！！！HTTPS安全通信机制有密钥是保存在服务器端的～注意??：认证是要钱的！！！HTTPS安全通信机制
#### 为什么HTTPS不是那么普及
1.加密通信与纯文本通信相比,消耗更多的CPU和内存资源
2.购买ca证书是要钱的,免费证书很少，需要交费.
3.http和https使用的是完全不同的连接方式用的端口也不一样,前者是80,后者是443
4.少许对客户端有要求的情况下,会要求客户端也必须有一个证书.目前少数个人银行的专业版是这种做法,具体证书可能是拿U盘作为一个备份的载体
5.HTTPS 一定是繁琐的

补充：env.bat  # 更快速的切换python环境
      call env\py3_test\Scripts\activate
      denv.bat
      call env\py3_test\Scripts\deactivate

# 10 tmp tips
[As listed on PyPI - packages in red don't support python 3, packages in green do. Hopefully one day everything will be greener. ](https://python3wos.appspot.com/)
>>> help()   keywords  print ``` quit 
###  CSV, Comma Swparated Values
```
import csv 

# exp 1
with open('tmp.csv', encoding='Big5', quotechar=',') as rf:
    with open('tmpout.csv', 'w', encoding='utf-8', newline='') as wf:
        # for row in csv.reader(rf)
        #     print(row, end='\t\t')
        rows = csv.reader(rf)
        csv.writer(wf).writerows(rows)

# exp2        
cuts = [
    'first, last',
    'justin, Lin',
    'Monica, Huanng'
]
for row in csv.DictReader(cuts):  # 除了以list方式处理，还可以以dict处理，DictReader(),DictWriter()
    print(row)
# {'last': 'Lin', 'first': 'Justin'}
# {'last': 'Huang', 'first': 'Monica'}

# exp3
cutss = [
    'Justin, Lin',
    'Monica, Huang'
]
for row in csv.DictReader(cuts, fieldnames=['first', 'lastname']):
    print(rwo)
# {'firstname': 'Justin', 'lastname':'Lin'}

# exp4
cuts = [
    {'firstname': 'Justin', 'lastname':'Lin'},
    {'firstname': 'Monica', 'lastname':'Huang'}
]
with open('sample.csv', 'w', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=['firstname', 'lastname'])
    writer.writeheader()
    writer.writerows(cuts)
# firstname,lastname
# Justin,Lin
# Monica,Huang

```
### JSON, JavaScript Object Notation
- 名称必须使用双引号包括
- 只可以是双引号包括的字串，或者 数字、true、false、null、Python的list，或子JSON格式
- 编码是将Python內建类型转换为JSON格式，反之为解码

Python | JSON
---|---
dict | 物体
list、tuple | 阵列（Python的list）
str | 字串
int、float | 数字
True、False、None | true、false、null

Python內建类型编码为JSON格式
```
import json

# exp1
obj = {
    'name': 'Justin',
    'age': 40,
    'childs': [{'name': 'Irens', 'age':8 }]
}
json.dumps(obj, sort_keys=True, indent=4)  # 指定separators={',',':'} 不需要空白，节省流量开销
                                           # dict的键只能是字串，否则ValueError，指定skipkeys=True则表示忽略
# output: '{"name": "Justin", "childs": [{"name": "Irene", "age":8 }], "age": 40}'

# exp2
class Customer:
    def __init__(self, name, age):
        self.name, self.age = name, age
    def json_serializable(self):
        return {'name': self.name, 'age': self.age}
json.dumps(cust, default=Customer.json_serializable) # default参数必须传回python內建类型
# output: '{"age":40, "name": "Jsutin"}'

# exp3 写入文件
with open('data.txt', 'w') as f:
    json.dump(obj, f)  # 参数2必须是具有write方法
    
# exp4 JSON解码为Python内键类型
jsonText = '{"name": "Justin", "childs": [{"name": "Irene", "age":8 }], "age": 40}'
json.loads(jsonText)
# output: {'name': 'Justin', 'age': 40, 'childs': [{'name': 'Irens', 'age':8 }] }

# exp5 JSON解码为自定义
def to_cuts(obj):
    return Customer(obj['name'], obj['age'])
cust = json.loads(jsonTest, object_hook=to_cust)
>>> cust.name, cust.age  # output: 'Justin', 40

# exp6 读入是解码
with open('test.txt') as f:
    print(json.loads(f))
# output: '{"name": "Justin", "childs": [{"name": "Irene", "age":8 }], "age": 40}'
```
### XML, python 提供 xml.dom (Document Object Model) xml.sax (Simple API for XML) xml.etree.ElementTree
对于常见的XML处理，Python建议使用xml.etree.ElementTree,它比DOM更为简单快速；比较SAX也有iterparse()可以使用，在
读取XML文件的过程中及时进行处理
parse()载入XML，将返回ElementTree实例，代表整个XML树
getroot()取得根节点，返回一个Element实例，一个Element就代表XML中的一个标签元素，是iterable的，可遍历其子元素
```
try:
    import xml.etree.cElementTree as ET
except ImportError:
    import xml.etree.ElementTree as ET
    
# exp1
def show_tags(elem, ident=' '):
    print(ident + elem.tag)
    for child in elem:
        show_tags(child, ident + ' ')
tree = ET.parse('country_data.xml')
show_tags(tree.getroot())

# exp2
root = ET.fromstring(xml)  # 参数是一个XML字符串,返回Element实例，代表XML的根节点
root.tag  # text 取得标签间包含的文字； attrib 取得属性，返回字典

# exp3
tree = ET.parse('data.xml')
tree.find('country')  # 返回找到的第一个
tree.findall('country/neighbor') # 以list返回全部子元素
tree.iterfind('country/neighbor')  # 建立成·生成器，迭代

ET.tostring(country)  # f返回取得的XML字符串的bytes信息
tree.write('sample.xml')  # 写入XML文件

country = ET.Element('country')
country.set('name', 'Taiwan')  # append 附加、insert 插入、remove 移除、set 设置元素

# exp4 一边读取一边剖析，iterparse()，针对标签的'start','end','start-ns','end-ns'事件发生时，进行对应的处理
       预设的iterparse(）只响应'end'事件，若要指定其他的事件，可一个传一个tuple给event参数
doc = ET.iterparse('country_data.xml', ('start', 'end'))
for event, elem in doc:
    if event == 'start' and elem.tag == 'country':
        print('<country>{}'.format(elem.attrib['name']), end='')
    elif event == 'end' and elem.tag == 'country':
        print('</country>')

```
# 14 git diff “old mode 100755 new mode 100644”
![image](https://github.com/Django-27/workspace/blob/master/pic/git-diff-1.png)

# 16 安装yeoman时出现，并且在fetchmetadata时卡住
使用npm 这是常有的事情，可以先用Ctrl+C终止安装，再进行重新安装。有更好的替代方案：命令行执行
```
npm install cnpm -g --registry=https://registry.npm.taobao.org

cnpm install -g yo  # cnpm跟npm用法完全一致，只是在执行命令时将npm改为cnpm;cnpm -v
```



# 11 win-sshfs的使用
最近因为需要在linux虚拟机里进行开发程序，虽然在linux里有超强的编辑器vim，但vim开发html前台代码并没有某些编辑器如sublime高效，在linux下使用sublime的话又要安装桌面环境，而且需要修复sublime在linux下不支持中文等一些问题。所以想要直接在windows下直接修改linux里的文件。

ssfhs可以通过ssh方式将远程的服务器上硬盘挂载到本地硬盘，也就是说只有你的虚拟机支持ssh链接，你就可以将虚拟机的硬盘挂载到本地，然后采用本地的方式来操作硬盘里的文件。

安装win-sshfs依赖Dokan，所以先安装Dokan再正常安装win-sshfs [link](http://pan.baidu.com/s/1hrGcHkK)

![image](https://github.com/Django-27/workspace/blob/master/pic/win-sshfs.png)


# 12 Raven：Sentry的Python客户端 
Sentry 是一个错误记录和聚合的平台，集中化的日志管理
```
SENTRY_DSN = 'xxx:xxx'                                            -> 将配置写到settings.py 中
SENTRY_HOST = 'ip:port/x'                             ->
RAVEN_CONFIG = {                                                  -> that whats in easyshop 
    'dsn': 'http://{}@{}'.format(SENTRY_DSN, SENTRY_HOST),        ->
    'release': raven.fetch_git_sha(os.path.dirname(os.pardir)),   ->
}                                                                 ->
client = raven.Client(settings.RAVEN_CONFIG['dsn'])               ->

client = Client('http://xxx:xxx@ip:port/x')           -> 方法二
>>> try:
...     1/0
... except:
...     client.captureException()
...
'22ebd14ec3a04db29f3ea7fbe886c418'
```

# 25
```
# pip install pytesseract
# sudo apt-get install tesseract-ocr
# python
>>> import pytesseract
>>> from PIL import Image
>>> img = Image.open("/home/wq/Downloads/number.png")
>>> img.show()
>>> pytesseract.image_to_string(img)
u'1234567890'

curl img_url > out.png
tesseract out.png stdout
```
```
import os
import subprocess

def image_to_string(img, cleanup=True, plus=''):
    # cleanup为True则识别完成后删除生成的文本文件
    # plus参数为给tesseract的附加高级参数
    subprocess.check_output('tesseract ' + img + ' ' +
                            img + ' ' + plus, shell=True)  # 生成同名txt文件
    text = ''
    with open(img + '.txt', 'r') as f:
        text = f.read().strip()
    if cleanup:
        os.remove(img + '.txt')
    return text

Error opening data file /usr/share/tesseract-ocr/tessdata/sim.traineddata
正确的设置data的位置：
```
sudo mv /usr/share/tesseract-ocr/tessdata/chi_sim.traineddata.traineddata /usr/share/tesseract-ocr/tes sdata/chi_sim.traineddata
export TESSDATA_PREFIX=/usr/share/tesseract-ocr/
echo $TESSDATA_PREFIX
```
由于版本不对应的关系，从新安装了 libpango1.0-dev tesseract=3.04
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```
cd tesseract-ocr
./autogen.sh
./configure
make
sudo make install
sugo ldconfig
$ make training
$ sudo make training-install
```
tesseract --list-langs

weiqiyuan@181che-dev:~$ tesseract 04.png -l chi_sim stdout
actual_tessdata_num_entries_ <= TESSDATA_NUM_ENTRIES:Error:Assert failed:in file tessdatamanager.cpp, line 53
错误原因：tesserat与tesserdata版本不统一
源码安装后还是没有解决数据格式不统一的问题


# 1 requests中text和content的区别
```
r.text is the content of the response in unicode.
r.content is the content of the response in bytes.
```
# 2 cookies中的_ga, _gat是什么意思
```
cookies = {
    '_ga': 'GA1.2.2003702965.1486066203',
    '_gat': '1',
}
```
#### 全称：Google Analytic's javascript to be used on a web page for web-tracking.
#### 同理：Hm_lpvt_, Hm_lvt_, 为百度统计

```
<script type="text/javascript">  
        var _gaq = _gaq || [];//定义GA变量数组。  
        _gaq.push(['_setAccount', 'UA-24479793-2']);//设置本跟踪代码所对应的Google帐户。  
        _gaq.push(['_trackPageview']);//定义按页面跟踪  
        (function () {//定义匿名的执行方法  
            var ga = document.createElement('script');//定义GA的脚本Dom对象。到时候会appendChild到Document中  
            ga.type = 'text/javascript';//不解释  
            ga.async = true;//定义GA数据传输方式为异步传输。  
            ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';//定义GA的JS源路径，自动取的，主要是做了一个协议判断，意味着GA可以跟踪htts网页和ssl网页，当你 的页面是http时就去http://www.google-analytics.com/ga.js取代码。当你是https页面时就去https://www.google-analytics.com/ga.js取代码。也可以指定为本地服务器目录/ga.js即本地加载ga.js代码。  
            var s = document.getElementsByTagName('script')[0];  
            s.parentNode.insertBefore(ga, s);//添加GA代码  
        })();  
    </script> 
```
# 3 opencv不支持打开gif，采用SimpleCV和matplotlib、PIL替代
gif分为动态的和静态的，返回的验证码保持为gif文件，需要去图片查看器打开；能不能使用opencv直接显示呢，不行，到3.6版还是不支持此格式；SimpleCV到时可以，2015.4.7停更，用起来也麻烦；使用matplotlib实现了gif的显示。
```
import matplotlib.pyplot as plt
img=plt.imread('captcha.gif')       
plt.imshow(img) 
plt.show()
```
# 4 收集chrome插件
Octotree,在浏览github的时候，添加侧边栏，避免了从选目录、文件的麻烦。
![104](https://github.com/Django-27/workspace/blob/master/pic/chrom-plugin.png)
# 5  .gitignore 使用小记
给项目添加一个 .gitignore 文件，从而使那些不该纳入版本控制的文件能够被 Git 排除在外的，如Python中的.pyc文件，可以自定义也可以参考[gitignore.io](https://github.com/joeblau/gitignore.io)提供的对于文件。
![image](https://github.com/Django-27/workspace/blob/master/pic/gitignore.png)


# 5 生成自己的词云 [link](http://amueller.github.io/word_cloud/generated/wordcloud.WordCloud.html#wordcloud.WordCloud)
```
import matplotlib.pyplot as plt
from wordcloud import WordCloud
import jieba

text_from_file_with_apath = open('星辰变.txt').read()
wordlist_after_jieba = jieba.cut(text_from_file_with_apath, cut_all = True)
wl_space_split = " ".join(wordlist_after_jieba)
my_wordcloud = WordCloud().generate(wl_space_split)

plt.imshow(my_wordcloud)
plt.axis("off")
plt.show()
```
![image](https://github.com/Django-27/workspace/blob/master/pic/word-cloud1.png)
```
my_wordcloud = WordCloud(font_path="C:\Windows\fonts\SIMYOU.TTF").generate(wl_space_split)  
```
![image](https://github.com/Django-27/workspace/blob/master/pic/word-cloud2.png)

# 查看公网ip
```
curl cip.cc
url ipinfo.io | more
```
# ab的命令参数比较多，我们经常使用的是-c和-n参数。
ab -c 10 -n 100 http://www.xxx.com/index.php [http://www.baidu.com/， 同时处理100个请求，并运行10次]
-c10表示并发用户数为10
-n100表示请求总数为100 
# JPS, java 进程状态工具
列出PID和Java主类名 jps
列出pid和java完整主类名 jps -l
列出pid、主类全称和应用程序参数 jps -lm
列出pid和JVM参数 jps -v(jps -lvm 类似于 ps -f | grep java)
# linux 查看进程运行的完整路径
通过ps及top命令查看进程信息时，只能查到相对路径，查不到的进程的详细信息，如绝对路径等
/proc Linux在启动一个进程时，系统会在/proc下创建一个以PID命名的文件夹，在该文件夹下会有我们的进程的信息， 其中包括一个名为exe的文件即记录了绝对路径，通过ll或ls –l命令即可查看。
ll /proc/PID
比如，我们查看mongo使用以下命令：
[root@rmpapp local]# ps -ef|grep mongo
root      9466  6380  0 13:41 pts/1    00:00:00 grep --color=auto mongo
root     16053     1  0 8月14 ?       06:45:52 ./mongod --config mongodb.conf
然后：ll /proc/16053
# 找出占用内存资源最多的前 10 个进程 ps -auxf | sort -nr -k 4 | head -10
# 找出占用 CPU 资源最多的前 10 个进程 ps -auxf | sort -nr -k 3 | head -10

# Linux服务器之间拷贝文件，比如定时备份。
scp -r /local/d24.tar.gz  root@111.11.143.135:/home/backups/
如果想增量拷贝，我们可以使用rsync命令。
rsync -avz  /local  root@111.11.143.135:/home/backups/

# linux SSH默认端口是22，不修改的话存在一定的风险，要么是被人恶意扫描，要么会被人破解或者攻击，所以我们需要修改默认的SSH端口。
```
vi /etc/ssh/sshd_config 
```
默认端口是22，并且已经被注释掉了，打开注释修改为其他未占用端口即可。

开启防火墙端口并重复服务即可。
```
systemctl restart sshd.service
```