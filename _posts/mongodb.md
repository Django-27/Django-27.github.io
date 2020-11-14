# 1 一次小的记录 mongo
```javascript
from utils.mongodb import get_mongodb_collection
from account.controllers import deal_statis_controllers as d_ctl
from account.controllers import dealer as dealer_ctl

d_dash = d_ctl.get_dealers(status_nums=[20, 32, 1])

d = get_mongodb_collection('bi_dealer_base_info')
rs = d.find({'status': {'$gte': 0}}, {'_id': 0, 'dealer_id': 1}).sort('dealer_id', 1)
d_bi = [int(item['dealer_id']) for item in list(rs)]

[i for i in d_dash if i not in d_bi]
[i for i in d_bi if i not in d_dash]

dealer_ctl.get_test_dealer_ids()
d.remove({'dealer_id': 2})

(order.pay_by in (Order.PAY_BY_WX_TO_SELF, Order.PAY_BY_ALP_TO_SELF, Order.PAY_BY_CASH) and Order.price_total >= 10000 * 100)

_Order.objects.filter(**query).exclude(typ=_Order.TYP_REFUND).exclude(Q(
pay_by__in=(_Order.PAY_BY_ALP_TO_SELF, _Order.PAY_BY_WX_TO_SELF, _Order.PAY_BY_CASH)) & Q(price_total__gte=FILTER_MONEY)).values_list('price_total', flat=True)
```
# 2 mongo tips
根据业界规则，偶数为“稳定版”（如：1.6.X，1.8.X），奇数为“开发版”（如：1.7.X，1.9.X)

db.bi_order_base_info.find({'order.id': 415782}).explain()  # hint() 限制使用某个索引
#### (1)3.*版本需要再加参数
```javascript
db.bi_order_base_info.find({'order.id': 415782}).explain('executionStats')  # allPlanExecution

db.bi_order_base_info.getIndexes()
db.bi_order_base_info.createIndex({'dealer.region_city_id': 1})
```
#### (2)建立唯一索引
```javascript
db.person.ensureIndex({"name":1},{"unique":true})
```
#### (3)组合索引
```javascript
db.person.ensureIndex({'name': 1, 'age': 1})
```
#### (4)删除索引,查索引时的name值
```javascript
db.person.dropIndex("dealer.region_city_id_1")
```
# 3 tips 重要：
- mongo原生语句用false和true； pymongo语句用True和False
- 千万不要写成‘false’或‘true’
- null =>  None
```javascript
 # 使用 DATE_FORMAT 进行分组和格式化
 select count(*), DATE_FORMAT(add_time,'%Y%m') from pay_record where original_msg like '%ONLINE-ORDER-MICRO%' group by  date_format(add_time, '%Y-%m') ;
 
 # 表2某字段更新表1某字段
 # 只更新表1字段在表2中的；如果没有最后一句，表1中id在表2中没有，则更新null值
 update tabe_name_1 T1
    set column_name_1 = 
        (select column_name_2 from table_name_2 T2 where T1.id = T2.id)
 where exists (select 1 from table_name_2 T2) where T1.id = T2.id  
 
 # with as 用法 ，多表关联，对结果汇总
 with table_name_1 as 
        (select .. .. ), 
    table_name_2 as 
        (select .. .. ),  
    table_name_3 as 
        (select .. .. )
    select .. from table_name_1, table_name_2, table_name_3

# left join: 返回左表的所有行
select *
    from table_name_1
        left join table_name_2
            on table_name_1.column_name = table_name_2.column_name
```

# 4 mongo 模糊查询 与 索引的重要性
```
# in pymongo
import re
1 db.col.find({'region_province': {'$in': [re.compile(province_name_from_map)]}})  # 支持多个,或的关系
2 {'xxx': re.compile('xxx')}
3 {'xxx': {'$regex': 'xxx'}}

# in shell or ide robo3T
1 db.getCollection('bi_dealer_base_info').find({"region_province" : /北京|上海/}, {'_id': 0, 'region_province': 1})
2 db.getCollection('bi_dealer_base_info').find({'filed':/北京/})

# django orm fuzzy query
1 TestData.objects(content__icontains='abc123')  # 不区分大小写
2 TestData.objects(content__contains='abc123')
3 TestData.objects(Q(name__icontains=u'天地') | Q(name__icontains=u'宇宙'))  # 当使用Q()来进行组合查询时，必须使用位运算符（|和&），而不能使用or，and来进行逻辑运算;比较合适确定关键词数目的情况
```
- tips(坑): 需要给mongo中更新一个字段（barcode，cover），但是没有考虑更新索引，导致系统占用极高；加索引解决
```
db.getCollection('bi_order_item_base_info').getIndexes()
db.getCollection('bi_order_item_base_info').ensureIndex({'goods.id': 1})
b.getCollection('bi_order_item_base_info').dropIndex({'datetime': 1, 'dealer.region_province_id': 1})
```
- tips: 优化的思路
```
创建索引,限定返回结果数,只查询使用到的字段,采用capped collection,采用Server Side Code Execution
使用Hint(强制使用某个索引)，采用Profiling

自己的角度：索引非常重要；提高代码的质量，更多的使用函数，避免for eg：date_format
```
![image](https://github.com/Django-27/workspace/blob/master/pic/mongo1.png)

- Nginx 作为一个 HTTP 服务器， 它提供了许多的功能，但无法直接为 Flask 应用提供服务
- Gunicorn 是一个 WSGI 服务器，可以处理 HTTP 请求，并将它们通过路由交给任何支持 WSGI 的 python 应用处理（比如 Flask、Django、Pyramid 等）
- Nginx 收到 HTTP 请求，并将其传递给 Gunicorn 交由你的 Flask 应用进行处理（比如你在 view.py 中定义的路由）
- netstat -tnpl
```
Mongodb 3t
# 建立基础索引
db.users.ensureIndex({age:1}) 
# 后台建立基础索引
# MongoDB的ensureIndex()是阻塞型操作，会暂停数据库上所有正在进行的其他操作，直到创建索引完成
# 当启动后台模式时，其他的操作，包含写，在创建索引期间不会被阻塞；该索引在创建完成前不会被应用到查询中去
# 选择不选择后台方式，需要经验和积累; 后台模式虽好，会不会造成‘雪崩’
db.t3.ensureIndex({age:1} , {backgroud:true})
# 建立文档索引
db.factories.ensureIndex( { addr : 1 } );
# 建立组合索引
db.factories.ensureIndex( { "addr.city" : 1, "addr.state" : 1 } );
# 建立唯一索引
# 否则，在使用索引的时候有一个'最左原则'
db.t4.ensureIndex({firstname: 1, lastname: 1}, {unique: true});
# 删除索引
db.t4.dropIndex({firstname: 1}
# 强制使用索引，并查看索引使用情况
db.t4.find({name": 'a'}).hint({name:1}).explain()

pymongo
# 建立非唯一索引
db.collection.create_index([('key',pymongo.ASCENDING)])
# 建立唯一索引
db.collection.create_index([('key',pymongo.ASCENDING)],unique=True)
# 建立文档索引(中文检索)
db.collection.create_index([('field',pymongo.TEXT)]) #对名为field的项建立文档索引
# 删除索引
db.collection.drop_index([('key',pymongo.ASCENDING)])
```
# 5 django_debug_tool_bar 

- 实例 [link](https://www.jianshu.com/p/38b733cc0779)
- 添加其他的面板 [link](http://django-debug-toolbar.readthedocs.io/en/latest/panels.html)
- pip install django-debug-toolbar-request-history (0.0.7)
- pip install django-debug-toolbar
![image 2018-03-14](https://github.com/Django-27/workspace/blob/master/pic/django_debug_tool_bar.png)

- 尝试写到 local_settings 中， 但是注释了url的部分，也可以使用
```
if DEBUG:
    from easyshop.settings import INSTALLED_APPS, MIDDLEWARE_CLASSES
    INSTALLED_APPS += ('debug_toolbar',)
    MIDDLEWARE_CLASSES += ('debug_toolbar.middleware.DebugToolbarMiddleware',)

    INTERNAL_IPS = '127.0.0.1'

    DEBUG_TOOLBAR_PANELS = [
        'ddt_request_history.panels.request_history.RequestHistoryPanel',
        'debug_toolbar.panels.versions.VersionsPanel',
        'debug_toolbar.panels.timer.TimerPanel',
        'debug_toolbar.panels.settings.SettingsPanel',
        'debug_toolbar.panels.headers.HeadersPanel',
        'debug_toolbar.panels.request.RequestPanel',
        'debug_toolbar.panels.sql.SQLPanel',
        'debug_toolbar.panels.staticfiles.StaticFilesPanel',
        'debug_toolbar.panels.templates.TemplatesPanel',
        'debug_toolbar.panels.cache.CachePanel',
        'debug_toolbar.panels.signals.SignalsPanel',
        'debug_toolbar.panels.logging.LoggingPanel',
        'debug_toolbar.panels.redirects.RedirectsPanel',
    ]

    DEBUG_TOOLBAR_CONFIG = {
        'SHOW_TOOLBAR_CALLBACK': 'ddt_request_history.panels.request_history.allow_ajax',
    }

    # from easyshop.urls import urlpatterns
    # import debug_toolbar
    # urlpatterns += patterns(url(r'^__debug__/', include(debug_toolbar.urls)),)

```
# 6 MongoDB 备份(mongodump)与恢复(mongorestore) [参考](http://www.runoob.com/mongodb/mongodb-mongodump-mongorestore.html)
#### tips 
- mongo的安装问题，导致 which mongo 看不到，但进入sudo 后可以看到
- 还涉及到一个问题，则配置文件需要添加改动到.zshrc  ,改动.bashrc 将不再管用；因为终端用的是 zsh
- 如果终端用zsh，则在.zshrc中加入 . /usr/share/autojump/autojump.sh ，之后sutojump的快捷键j就能使用了
```
export PATH=$PATH:~/bin/arcanist/bin:/etc/mongodb/bin
```
- scp: /home/weiqiyuan/qiyuan_tmp1: not a regular file
- 办法一：sudo chown weiqiyuan:weiqiyuan qiyuan_tmp2
- 方法二：scp -r weiqiyuan@192.168.4.141:/home/weiqiyuan/qiyuan_tmp1 . # 加入 -r 参数

#### (1)导出：
```
mongodump --collection bi_order_base_info --db easyshop --out qiyuan_tmp1
```
#### (2)导入：
```
第一次： mongorestore --host localhost:27017 -d easyshop -c  qiyuan_tmp1
碰到问题：
2018-03-28T11:11:29.510+0800    the --db and --collection args should only be used when restoring from a BSON file. Other uses are deprecated and will not exist in the future; use --nsInclude instead
2018-03-28T11:11:29.510+0800    using default 'dump' directory
2018-03-28T11:11:29.510+0800    see mongorestore --help for usage information
2018-03-28T11:11:29.510+0800    Failed: mongorestore target 'dump' invalid: stat dump: no such file or directory

第二次：直接采用 mongorestore  qiyuan_tmp2   成功，即导入时不需要参数指定
```
#### (3)线上-开发环境的导数据
```
>> mongo # 进入mongo
>> use easyshop; # 进入数据库
>> db.auth({'user': 'dshlmongo', 'pwd':'MA0GC***' }) # 进行验证
>> show dbs; show collections;  # 有可能第一个命令用不了，那是因为直接指定了默认库，或权限还是不够

注：由于多台机器间没有都与线上ssh，所以还是借助本地scp，再scp到另一个地方；如果都可以相互ssh，那方便多了

mongodump --host localhost --port 27017 -u dshlmongo -p MA0**** -d easyshop --collection bi_order_item_base_info --out qiyuan_tmp4
```
#### (4)只进行部分导入、导出的方式（beautiful）
```
mongodump --query "{\"ts\":{\"\$gt\":{\"\$date\":`date -d 2011-08-10 +%s`000},\"\$lte\":{\"\$date\":`date -d 2011-08-11 +%s`000}}}"

# For MongoDB version 3.0.x and above, using mongodump --query is the recommended method if you need partial dump of your data
# 本语句不在支持：mongorestore --filter '{"field": 1}'

# 导出是进行条件筛选
mongodump --host localhost --port 27017 -u dshlmongo -p MA0**** -d easyshop --collection bi_order_item_base_info --query '{"datetime": {"$gt": Date(1517414400)}}' --out qiyuan_tmp5
```
- 其中的参数时一个json格式，可以用json.dumps产生
- 日期的处理是一个麻烦，需要用时间戳；可以 date -d 2018-02-01 +%s => 1517414400 用终端转换产生
- 时间戳*1000的，也就是毫秒
- 加入时区的考虑： date -d 2018-02-01 --date='TZ="Asia/Shanghai"' +%s
- 索引也自动进行了导入 db.getCollection('bi_order_base_info').getIndexes()
- error parsing command line options: unknown option "colllection"  那么就用 -c 替代 

# 7 MongoDB 使用
```
{
    "_id" : ObjectId("5a10dc57c8a0cd33286515d8"),
    "order" : {
        "pay_platform" : NumberLong(0),
        "pay_by" : 0,
        "price" : 2350,
        "typ" : 2,
        "id" : NumberLong(10534)
    },
    "datetime" : ISODate("2016-11-20T07:15:08.000Z"),
    "dealer" : {
        "is_test" : false,
        "region_province" : "北京",
        "name" : "友谊超市天润社区店122",
        "region_id" : NumberLong(651),
        "region_city" : "北京市",
        "region" : "海淀区",
        "region_city_id" : NumberLong(643),
        "id" : NumberLong(122),
        "region_province_id" : NumberLong(642)
    }
}
```
 - 更新其中的一个子文档（没有则创建）：collection.update({'order.id': 10534}, {'$set': {'order.pay_platform': 99}}
 - 若要删除真个 order 信息， {"$unset" : {"order" :1 }} 
 - {"$inc":{"pageviews": n}})  # 其中n可以是整数或负数
 - $rename 修改字段的名字
 - 如果order是个列表，可能要用到：{ $set: { "order.$.pay_platform" : 6 } }
 
 ```
db.getCollection('bi_order_base_info').aggregate([
              {
                '$match': {
                  'order.typ': {
                    '$in': [9]  # 如果是数据，不能写成字符串，否则找不到
                  },
                  'datetime': {
                    '$gte': ISODate('2017-11-01 00:00:00'),
                    '$lte': ISODate('2017-11-20 23:59:59')
                  }
                }
              },
              {
                '$group': {
                  '_id': 'null',
                  'price': {
                    '$sum': '$order.price',      # 统计钱数
                  },
                  'order_count': {
                    '$sum': 1    # 统计记录条数
                  }
                }
              }
            ])
 ```

