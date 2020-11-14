# 1 Ays 调度平台全量执行当前所有任务

# 1.1 设计到的任务有
```javascript
- 现在就是3个媒体，gdt、jrrt、wx，如果手动全量执行，在ays调动平台
- gdt：需要执行 dmp_task_gdt_api_3  ， 13， 28 三个任务
- jrtt: 需要执行 dmp_task_jrtt_api_4 到 31 ， 这些是一代代理商
             - dmp_task_jrtt_one_agent 这个也是一个一代代理商
             - dmp_task_jrtt_child_agent 二代代理商
- wx：需要执行dmp_task_wx_api 任务
```
## 1.2 余额任务涉及到的账户信息，在 Redis 中的表
``` javasctipt
广点通-获取代理商下的广告主列表: dmp_gdt_api_advertiser_list
    Ark_gdt_api/gdt_api/interface/list/AdvertiserGet.py
    agent_key   = 'gdt_agents_v2_{}'.format(tenant_alias)
    account_key = 'gdt_advertiser_v1.1_{}_{}'.format(tenant_alias, date)
今日头条API广告主列表: dmp_jrtt_api_advertiser_list (（一代代理商、二代代理商、一代子账户、二代子账户）)
    Ark_jrtt_api/jrtt_api/interface/list/AdvertiserSelect.py
    加一步：获取账户信息 dmp_jrtt_api_next_jobs     account_list
今日头条二代-代理商报表-头任务: dmp_jrtt_api_agent_report
    Ark_jrtt_api/jrtt_api/interface/report/daily/DailyReportAgent.py
    one_agent_key: 'jrtt_one_agents_v2_T100117'
    one_account_key: 'jrtt_accounts_v2_T100117_20200519'
    child_agent_key: 'jrtt_child_agents_v2_T100117'
    child_account_key: 'jrtt_child_advertiser_v2_T100117_20200519'
微信-API-广告主列表: dmp_wx_api_advertiser_list
    Ark_wx_api/wx_api/interface/list/AdvertiserGet.py
    agent_key = 'weixin_agents_{}'.format(tenant_alias)
    wx_advertiser_{}_{}_{}
#
# 1 gdt  ->  3 jrtt ->  8 weixin  (代理商数量、账户数量查询 )
# redis-db2-> redis_account_key: Score->agent_id、Member->account_id
#
# zcard gdt_new_access_token_T100117->agent_id->6; zcard gdt_advertiser_v1.1_T100117_20200520->account->6987
#
# zcard wx_new_access_token_T100117 -> 3;          zcard wx_advertiser_T100117_9096610_20200520 -> 146
#
# zcard jrtt_agents_v2_T100117->103 zcard jrtt_child_advertiser_v2_T100117_20200520->24706
#
#d=c.zrevrange('gdt_new_access_token_T100117', 0, -1,  withscores=True)
#
```
------------------------
- 由于历史原因，agent_list = [agent_id], agint_list 只能是一个值，不能放一个真实的list
- account_list = [ account_id ], account_list 可以是多个值
- -a 5214977664
- wx_fund 广告主redisKey与指定账户列表不能同时为空  -g 4011704 -a 9998960
- jrtt_fund 广告主redisKey与指定账户列表不能同时为空 -g 3367268016
- 租户 - 媒体 - 代理商（今日头条特殊又分了：一代、二代）- 广告主 - 状态 - 创建时间 - 更新时间
------------------------
- 广告推广账户（Advertiser）-> 推广计划（Campaign）-> 广告组(Adgroups) -> 广告(Ad) -> 广告创意(Adcreative)
- 一条广告只能有一个创意，一个创意可以和多个退款计划关联
- REPORT_LEVEL_ADVERTISER广告主级别报表、REPORT_LEVEL_CAMPAIGN推广计划级别报表、REPORT_LEVEL_ADGROUP广告组级别报表、REPORT_LEVEL_AD广告级别报表
- REPORT_LEVEL_PROMOTED_OBJECT推广目标级别报表
------------------------
- gdt/wx 广告主列表就够了
- jrtt 先获取账户列表，一代还需要获取账户信息，二代获取DailyReportAgent.py
- gdt 只存有消耗的数据
------------------------
- ays 执行器地址 : 10.0.0.39:18181
- 到对应机器， 日志排查 /data/logs/api/gdt_v1.1/gdt_hourly_report_ad_get_T100117_20200619.log

## 1.3 模拟一次 jrtt 接口的请求过程
- 今日头条：多合一数据报表接口（https://ad.oceanengine.com/openapi/doc/index.html?id=605)
- jrtt 拿到一个账户 1664757665615880 ，还需要一个 token 字段
- token 是代理商的信息，jrtt的代理商分一代和二代，这里已知是一代，
- 遍历redis的表 key 是 account_id(即advertiserment id)， value 是 agent_id
- 拿到 agent_id 再去 redis： jrtt_new_access_token_T100117 找一下，里面有agent_id 的token
- 此时建立了 账户与token的关联关系
### 1.3.1 需要通过redis账户表找到对应的代理商
```JavaScript
    from api_data.api_common.RedisConn import RedisConnection
    r = RedisConnection.get_connection()
    d = r.zrevrange('jrtt_accounts_v2_T100117_20200526', 0, -1,  withscores=True)
    for i, j in d:
       if i.decode() == '1664757665615880':  # 二进制的字符串直接decode转换为字符串
          print(i, j)
```
### 1.3.2 通过 token 直接请求 jrtt 接口检查返回数据
```JavaScript
advertiser_id = '1664757665615880'
token = '9b7e0d9cbd9f7b8de39299632c2f962001e4421a'
def get_integrated_stat():
    import requests
    open_api_domain = "https://ad.oceanengine.com"
    path = "/open_api/2/report/integrated/get/"
    url = open_api_domain + path
    params = {
        "advertiser_id": advertiser_id,
        "start_date": "2020-05-21",
        "end_date": "2020-05-22",
        "group_by": ["STAT_GROUP_BY_BIDWORD_ID", "STAT_GROUP_BY_AD_ID"]
    }
    headers = {
        'Content-Type': "application/json",
        'Access-Token': token
    }
    resp = requests.get(url, json=params, headers=headers)
    resp_data = resp.json()
    print(resp_data)
    return resp_data
if __name__ == '__main__':
    get_integrated_stat()
```

# 2 涉及到的数据库 即 happybase 操作 hbase
- mysql: request-log, response-log
- redis: k-v token, agent, account
- hbase: row-key, response-data, request-info
- mysql-eton-agents 重点字段有，agent_id_from_platform 即代理商id, a.auth_status in (1,2) and a.auth_type = 1
- mysql-eton-tenant:`status` = 1 表示租户状态正常
- data_date 传给下发事件 evt 的 是带横线的
- 离线数据，也就是日报表，redis中只有昨天的，redis_accounts_key 中最新的日期是昨天
- 如果获取连接对象时 autoconnect=False 需要手动打开 c.open(), 默认 True
- 还有对应的前面的 platform_id: 1 gdt  ->  3 jrtt ->  8 weixin
```javascript
row_key:    9|20200508|99872976879|84421955984|T100117
resp_data:  {'tenant_alias': 'T100117', 'data': {'balance': 0.0, 'name': '杭州时趣信息技术有限公司lr3', 'grant': 0.0, 'valid_balance': 0.0, 'cash': 0.0, 'valid_cash': 0.0, 'email': 'm******3@y*******i.com', 'advertiser_id': 99872976879, 'valid_grant': 0.0, 'valid_return_goods_abs': 0.0, 'return_goods_abs': 0.0, 'return_goods_cost': 0.0}, 'timestamp': 1588984341}
_row_key_design = '{}|{}|{}|{}|{}|{}'.format(account_id[-1], self._data_date.replace('-', ''), account_id, agent_id, fund_type, self._tenant_alias)
接口请求参数： {'agent_id': 1050262, 'tenant_alias': 'T100117', 'data_date': '2020-05-30', 'media_name': 'gdt', 'cur_job_name': 'HourlyReportAdvertiser11', 'redis_accounts_key': 'gdt_advertiser_v1.1_T100117_20200530', 'redis_keys': [], 'account_list': [], 'agents_list': ['1050262'], 'task_type_list': 'SAVE', 'redis_channel': 'None', 'incr_min':60, 'output': 'HBASE', 'minutely_control': 1000, 'agent_level': 'parent', 'agent_total': 6, 'batch_key': '57ed39cc25bd5f43fa106779e405a189', 'data_type': 'day'}
编码后的参数： %7B'agent_id'%3A%201050262%2C%20'tenant_alias'%3A%20'T100117'%2C%20'data_date'%3A%20'2020-05-30'%2C%20'media_name'%3A%20'gdt'%2C%20'cur_job_name'%3A%20'HourlyReportAdvertiser11'%2C%20'redis_accounts_key'%3A%20'gdt_advertiser_v1.1_T100117_20200530'%2C%20'redis_keys'%3A%20%5B%5D%2C%20'account_list'%3A%20%5B%5D%2C%20'agents_list'%3A%20%5B'1050262'%5D%2C%20'task_type_list'%3A%20'SAVE'%2C%20'redis_channel'%3A%20'None'%2C%20'incr_min'%3A60%2C%20'output'%3A%20'HBASE'%2C%20'minutely_control'%3A%201000%2C%20'agent_level'%3A%20'parent'%2C%20'agent_total'%3A%206%2C%20'batch_key'%3A%20'57ed39cc25bd5f43fa106779e405a189'%2C%20'data_type'%3A%20'day'%7D
gdt指定接口返回信息字段：fields = ['hour', 'campaign_id', 'adgroup_id', 'ad_id', 'promoted_object_type', 'promoted_object_id',....]
gdt指定接口遍历信息字段：params = {'group_by': '["hour","ad_id"]', 'values': ('REQUEST_TIME', 'REPORTING_TIME', 'ACTIVE_TIME')}
```

## 2.1 happybase 遍历 HBase 内部数据
```JavaScript
from api_data.api_task.DataComparison import DataComparison
dc = DataComparison()
t = dc.conn.table('tenant_jrtt_advertiser_fund_get_v2')  # 获取表对象,不会检查表格是否存在
r = t.row('9|20200508|99872976879|84421955984|T100117')  # 获取一行信息
r[b'info:data']  # 获取指定 family:colum 的值
rows = t.rows(['9|20200509|99872976879|84421955984|T100117', '9|20200509|99872976879|84421955984|T100117'], include_timestamp=True)  # 返回多个row_key的结果，还可以加上时间戳
for key, value in rows:
    print(key, value)  # key即row_key， value是data+time的元祖
t.cells('9|20200509|99872976879|84421955984|T100117', b'info:data', versions=3)  # 获取多版本的信息
```
## 2.2 happybase 操作 HBase 获得返回数据
```javascript
from api_data.api_common.HbaseConn import HbaseConnection
c = HbaseConnection.get_connection()
c.open()                               # c.create_table('tenant_gdt_funds_qiyuan', {'info:data': dict()})
t = c.table('tenant_gdt_funds_qiyuan') # c.delete_table('tenant_gdt_funds_qiyuan', disable=True)
b = t.batch()
b.put(row='9|20200509|99872976879|84421955984|T100117',data={'info:data': json.dumps({'qiyuan': 'first'})} )
b.send()
t.row('9|20200509|99872976879|84421955984|T100117')  # {b'info:data': b'{"qiyuan": "first"}'}
b.delete(row='9|20200509|99872976879|84421955982|T100117'
```
## 2.3 通过 happybase scan 统计行数
```JavaScript
from api_data.api_common.HbaseConn import HbaseConnection
c = HbaseConnection.get_connection()  # 这里autoconnect=True
t = c.table('tenant_gdt_hourly_report_ad_test')
    table = self.conn.table(hbase_table)
    # row_filter = "RowFilter(=, 'regexstring:{}')".format('20190318\|5\|\d*\|25')
    # date_today = '20200606'  # filter = "RowFilter(=,'substring:20171025')"
    row_filter = "RowFilter(=, 'regexstring:{}')".format(self.date_today)
    scan = table.scan(filter=row_filter, columns=['info:data'])
    count = 0
    try:  # for i, j in t.scan(reverse=True):
        for k, v in scan:
            for _, _ in v.items():
                count += 1
    except:  # Hbase_thrift.IOError v没有找到表
        pass
    return count
```
## 2.3 HBase 刷数据  in -> HBase
``` javascript
from api_data.api_common.HbaseConn import HbaseConnection
c = HbaseConnection.get_connection()
t = c.table('tenant_gdt_daily_report_ad_v3')
for item in ['REQUEST_TIME', 'REPORTING_TIME', 'ACTIVE_TIME']:
    c.open()
    b = t.batch()
    b.put(row='8|20200509|9871225279|8442129383|T100117|{}'.format(item), data={'info:data': json.dumps(d)})
    b.send()
```
## 2.4 HBase shell 创建一张表
```JavaScript
disable table_name
drop_table_name
create 'tenant_gdt_daily_report_adgroup_v3',{NAME=>'info',COMPRESSION =>'SNAPPY'},{NAME=>'col',COMPRESSION=>'SNAPPY'},{SPLITS=>['1|','2|','3|','4|','5|','6|','7|','8|','9|']}
```
## 2.5 happybase==1.2.0
- Python操作hbase存、放、查等比较方便，但是遍历不是他的强项
- 需求是对比正式表和测试表指定日期数据量总数比对，充分利用filter、reverse、row_start、 row_stop等
- 筛选字段都是对 rowkey 提前过滤筛选，需要对rowkey具体情况具体分析
- from Hbase_thrift import IOError 表不存在需要捕获一下这个异常 except IOError as e:
```
scan = table.scan(filter=row_filter, row_start=b'0|20200611', row_stop=b'0|20200612')
row_start = bytes('{}|{}'.format(i, row_start_time), encoding='utf8')
scan = table.scan(row_start=row_start, row_stop=row_stop)
```
# 3 日志记录 kafka ，异常 sentry 报警

## 3.1 通过 Sentry 报警例子
```JavaScript
from api_data.api_common.ExceptionHandle import ExceptionHandle
try:
    ...
except Exception as e:
    print("eton get_connection:", repr(e))
    ExceptionHandle.capture_exceptio取Mysql连接异常', repr(e))
```
## 3.2 通过 Kafka 日志记录例子
```javascript
from api_data.api_common.LoggerRecord import LoggerRecord
from api_data.api_common.KafkaConn import KafkaConnection
self.__log_topic = 'gdt_api_log'
self.__kafka_client = KafkaConnection.get_client()
start_log = {"tenant": self._tenant_alias, "cur_job": self._cur_job_name,
             "interface": self.__api_request_path, "agent_id": self._agent_id,
             "data_date": self._data_date, "argv": 'test', "log_content":
             "ApiHandler start", "log_type": LogType.API_JOB_ATSRT,
             "log_time": Utils.get_cur_time()}
LoggerRecord.record_log(self.__log_topic, start_log, self.__kafka_client)
```

# 4 当前代码执行流程整体分析

# 4.1 准备脚本参数，ArgvUtilsj.py 解析参数：参数字典 -> encode编码后参数

 ``` JavaScript
 参数字典： {
 	'agent_id': 1050262,                代理商
 	'tenant_alias': 'T100117',
 	'data_date': '2020-05-10',
 	'media_name': 'gdt',                媒体
 	'cur_job_name': 'info',
 	'redis_accounts_key': 'gdt_advertiser_v1.1_T100117_20200510',  账号key
 	'redis_keys': [],
 	'account_list': [],
 	'agents_list': ['1050262'],
 	'task_type_list': 'INCR',
 	'redis_channel': 'None',
 	'incr_min': 60,
 	'output': 'HBASE',
 	'minutely_control': 1000,
 	'agent_level': 'parent',
 	'agent_total': 3,
 	'batch_key': '57ed39cc25bd5f43fa106779e405a189',
 	'data_type': 'day'}

    {'agent_id': 1050262,'tenant_alias': 'T100117','data_date': '2020-05-10','media_name': 'gdt','cur_job_name': 'info','redis_accounts_key': 'gdt_advertiser_v1.1_T100117_20200510','redis_keys': [],'account_list': [],'agents_list': ['1050262'],'tas'agent_level': 'parent','agent_total': 3,'batch_key': '57ed39cc25bd5f43fa106779e405a189','data_type': 'day'}

encode编码后参数： %7b%27agent_id%27%3a++1050262%2c++%27tenant_alias%27%3a++%27T100117%27%2c++%27data_date%27%3a++%272020-05-10%27%2c++%27media_name%27%3a++%27gdt%27%2c++%27cur_job_name%27%3a++%27info%27%2c++%27redis_accounts_key%27%3a++%27gdt_advertiser_v1.1_T100117_20200510%27%2c++%27redis_keys%27%3a++%5b%5d%2c++%27account_list%27%3a++%5b%5d%2c++%27agents_list%27%3a++%5b%271050262%27%5d%2c++%27task_type_list%27%3a++%27INCR%27%2c++%27redis_channel%27%3a++%27None%27%2c++%27incr_min%27%3a60%2c++%27output%27%3a++%27HBASE%27%2c++%27minutely_control%27%3a++1000%2c++%27agent_level%27%3a++%27parent%27%2c++%27agent_total%27%3a3%2c++%27batch_key%27%3a++%2757ed39cc25bd5f43fa106779e405a189%27%2c++%27data_type%27%3a++%27day%27%7d
 ```
## 4.2 准备脚本参数，Utils.py 解析参数
```JavaScript
-g 24641 -a 10058169
/home/fund_api/wx_fund/wx_api/list/FundGet.py  -g 4011704 -k wx_advertiser_T100117_4011704_20200518
execute_cmd:python /home/fund_api/jrtt_fund/jrtt_api/interface/list/OneAgentFund.py  -g 3367268016 -k jrtt_one_agents_v2_T100117
```
## 4.3 mysql-19-nyfz 日志记录
```
recording start info in mysql table [`nyfz`.`api_result_log`]
```
## 4.4 模块内部执行过程
```JavaScript
-   1   解析参数： def args_format 解析encode后的字典，复制给变量
- 必填： agent_id、tenant_alias、data_date  代理商ID、租户ID、数据日期
- 选填： media_name         媒体名称
        cur_job_name       当前任务名
        redis_accounts_key redis中存放广告主id的键
        redis_keys         redis其他键
        account_list       账户列表
        agents_list        代理商列表
        task_type_list     任务类型
        shown_count        展示条数
        redis_channel      redis_channel键
        output
        agent_total
        agent_level, batch_key, data_type
-   2   加载类包，初始化 ModelHandler 类成员变量
        tenant_alias、data_date、date
        r = RedisConnection.get_connection()
-   3   进入 if __name__ == '__main__'
        developer、fields 字段、params 参数
        **ApiHandler().start()**
-   4   进入 ApiHandler 的 __init__ 函数
        有api限定    ：频次限制、反页方式、请求参数params、额外参数fields等
        有handle限定 ：线程数、任务名称、报表路径等
        初始化   MysqlUtils     MySQL工具类
                ProcessExit    任务结束前执行
                DataProcess    数据处理类
                RequestProcess http请求类
                ResponseProcessHandler 应答响应数据处理类
-   5   进入 ApiHandler 的 start 函数
        1. 记录启动日志  self.__mysql_utils.record_process_start() 写入一条sql，信息是handler的一些参数等
            - 创建文件夹    self.__data_process.file_dir_create()
            - 删除已有文件    self.__data_process.remove_exists_file()
            - 创建空文件 self.__data_process.touch_file()
              {}/{}/{}/{}_{}_{}.json
              redis: zrange jrtt_new_access_token_T100117 0 -1 WITHSCORES
              jrtt_new_access_token_T100117
              ConfConstant.FILE_DIR, self._date, self._tenant_alias, self.__hbase_table_name, self._date, self.__file_index))
        2. 开始执行任务   self.__threads_req() 多线程
              拿到token字典（全量） self.__data_process.token_set_up
              获取数据源:
                        self.__data_process.get_source -> 拿到 source_list = [(None, {'params': {'advertiser_id': '3606821801', 'agent_id': '3367268016'}}), ... ]      注： advertiser_id == account_id
                        return: [(None, {"params": {"xx": "xx", ...}}), (None, {"params": {"xx": "xx", ...}}), ...]
                        注：脚本参数中 "account_list": ['3377683771'],"agents_list": ["3367268016"] 是需要的
                           如果 account_list 和 agent_list 都有值，用的是 account_list
                           如果 account_list 没有，那必须有 agent_list ; 否则：查到的 source_list 是一个全量，现在是一万多记录
                        source_list 不空 启动多线程
                                    为空 self.__data_type == 'day' -> 非 7Days -> 发sms记logger
              启动多线程请求接口:用上面的 source_list  ->  紧跟着后面的  self.__process  进行request请求
                    jobs = makeRequests(self.__process, source_list)
                    [self.__thread_pool.putRequest(job) for job in jobs]
                    self.__thread_pool.wait()
                  __threads_req -> __process -> __api_request -> __response_process  ->  response_dict 数据
                  ResponseProcessHandler -> api_code_0     正常数据处理
                                         -> api_code_4010

              将缓存中剩余数据存入Hbase self.__data_process.output_modules()
                  __output： HBASE/FILE/NONE  基本是一个必须的参数，如果什么都没有就记logger了
                        save_in_hbase   ->  batch.put
                        load_in_file    ->  f.writelines
            - 将缓存中剩余数据存入Hbase
            self.__data_process.output_modules()
            self.__data_process.upload_on_hdfs()
            - 使用过的redis键 ApiStatistics.used_redis_keys = self.__data_process.used_redis_key()
        3. 退出程序 self.__process_exit.finished(exit_code=1)
        --
        /Users/sunny/ybs/Ark_jrtt_api/jrtt_api/handler/api_utils/ProcessExit.py
        line: next_params   刚开始的时候解析了参数，这里需要的参数是接口需要的参数，接口间有依赖关系，即 next_job
        # FundTask 可以跑一个全量的数据出来，FundGet从中提取参数
        # /Users/sunny/ybs/Ark_jrtt_api/jrtt_api/interface/list/AdvertiserFundGet.py
        # Ark_tenant_auth 里面是一些头任务， 首先执行的 vim handler/ReceiveMsg.py handler/WxTask.py
        # datax 离线数据同步工具/平台(Job完成单个数据同步的作业，不同的切分策略将Job切分成多个小的Task子任务，Scheduler模块配置并发数、任务拆分、组装TaskGroup任务组)
        # 每一个Task都由TaskGroup负责启动，Task启动后，会固定启动Reader—>Channel—>Writer的线程来完成任务同步工作（job通过json文件进行配置）
```

# 5 异步asyncio+aiohttp VS 多线程TreadPool

## 5.1 多线程，需要注意参数有params 到 param 的变化
```javascript
import time
from threadpool import ThreadPool, makeRequests
def get_source_list():
    tenant_alias = 'T100117'
    data_date_short = '20200611'
    redis_account_key = 'gdt_advertiser_v1.1_{}_{}'.format(tenant_alias, data_date_short)
    r = RedisConnection.get_connection()
    ret = []
    """
     (None, {'params': {'account_token': '46f45241ab103e9a769e24366b870e19',
      'agent_id': 1050262, 'account_id': '10707125'}}), ..., (), ()]
    """
    for token, agent_id in r.zrange('gdt_new_access_token_{}'.format(tenant_alias), 0, -1, withscores=True):
        for account in r.zrangebyscore(redis_account_key, int(agent_id), int(agent_id)):
            ret.append({
                'account_token': token.decode(),
                'agent_id': int(agent_id),
                'account_id': account.decode()})
    return ret
def get_param(param):
    parameters = {
        'access_token': param['account_token'],
        'timestamp': int(time.time()),
        'nonce': str(time.time()) + str(random.randint(0, 999999)),
    }
    parameters.update({
        "account_id": param['account_id']
    })
    for k in parameters:  # 这个循环还暂时少不了，除非保证传入的都是字符串
        if type(parameters[k]) is not str:
            parameters[k] = json.dumps(parameters[k])
    url = 'https://api.e.qq.com/v1.1/funds/get'
    return url, parameters
def print_header_analysis(header_info):
    rate_limit = header_info.get('X-RateLimit-Remaining')  # 当前请求接口的频次余量百分比(99%,74%)(天,分钟)
    daily_quota, minutely_quota = rate_limit.replace('%', '').split(',')
    trace_id = header_info.get('X-TSA-Trace-Id')  # 媒体全局唯一id
    if int(daily_quota) < 20 or int(minutely_quota) < 20:
        time.sleep(random.randint(0, 3))  # 如果是aiohttp调用，这里需要变成 asyncio.sleep() '重要'
    print(rate_limit, get_mem_info(), trace_id)
def get_mem_info():  # {:.1f}MB
    return '{:.1f}MB'.format(resource.getrusage(resource.RUSAGE_SELF).ru_maxrss/1000000)
def requests_get_resp(param):
    url, parameters = get_param(param)
    resp = requests.get(url, params=parameters)
    # print(resp.headers)
    content = resp.json()
    print_header_analysis(resp.headers)
    print(content)
if __name__ == '__main__':
    params = get_source_list()  #  准备一个全量的参数 
    start_time2 = time.perf_counter()
    jobs = makeRequests(requests_get_resp, params)
    thread_pool = ThreadPool(50)
    [thread_pool.putRequest(job) for job in jobs]
    thread_pool.wait()
    end_time2 = time.perf_counter()
    print('Resquests Download {} sites in {} seconds'.format(len(params), end_time2 - start_time2))
```
## 5.2 异步asyncio + aiohttp
```JavaScript
async def download_one(sem, param, session):
    url, parameters = get_param(param)  # 多这一步是因为nonce字段是一个变值
    async with sem:
        async with session.get(url, params=parameters, ssl=False) as resp:
            content = await resp.text()
            rate_limit = resp.headers.get('X-RateLimit-Remaining')  # 当前请求接口的频次余量百分比(99%,74%)(天,分钟)
            daily_quota, minutely_quota = rate_limit.replace('%', '').split(',')
            trace_id = resp.headers.get('X-TSA-Trace-Id')  # 媒体全局唯一id
            if int(daily_quota) < 20 or int(minutely_quota) < 20:
                await asyncio.sleep(random.randint(0, 3))
            print(rate_limit, get_mem_info(), trace_id)
async def download_all(params):
    sem = asyncio.Semaphore(limit)  # 通过信号量限制协程的数目
    async with aiohttp.ClientSession() as session:
        tasks = [asyncio.create_task(download_one(sem, param, session)) for param in params]
        await asyncio.gather(*tasks)
if __name__ == "__main__":
    limit = 50
    params = get_source_list()
    start_time1 = time.perf_counter()
    asyncio.run(download_all(params))
    end_time1 = time.perf_counter()
    print('Async Download {} sites in {} seconds'.format(len(params), end_time1 - start_time1))
```
## 5.3 速度对比
- 有对比的是线程50和信号量50，线程慢了，消耗内存高了
- 异步调用快是快，但是如果服务器进行了限制，需要对这个地方进行重点处理
```JavaScript
Resquests (pool 10) Download 7481 sites in 468.949637 seconds       16/s 99%,90-80% 66.5MB
Resquests (pool 50) Download 7481 sites in 104.550943 seconds       71/s 99%,84% 105.1MB
Async (limit 10  )Download 7481 sites in 268.007115 seconds                          27/s
Async (limit 100 )Download 7481 sites in 20.36480 seconds(会出现code=11017) 374/s
Async (limit 50  )Download 7481 sites in 50.80374 seconds(马上到限制99%,1% ) 147/s
Async (limit 50  )Download 7481 sites in 53.29779 seconds(在中后位置出现了99%,1%)   140/s  69.8MB
Async (limit 100 + sleep) Download 7481 sites in 34.770966313 seconds                215/s
```