# celery 
- Celery的架构由三部分组成，消息中间件（message broker）、任务执行单元（worker）和 任务执行结果存储（task result store）组成。
![celery 架构图](https://github.com/Django-27/workspace/blob/master/pic/celery_1.png)
- 消息中间件(Broker) Celery本身不提供消息服务，但是可以方便的和第三方提供的消息中间件集成。包括，RabbitMQ, Redis等等
- 任务执行单元(Worker) Worker是Celery提供的任务执行的单元，worker并发的运行在分布式的系统节点中;
- 任务结果存储(Backend) Task result store用来存储Worker执行的任务的结果，Celery支持以不同方式存储任务的结果，包括AMQP, redis等
```
res = add. delay(3, 8) # 将装饰的任务函数条件到消息队列中，此时提交的任务函数并没有执行，只是提交到worker，它会返回一个标识任务的字符串
print(res) # 结果是标识任务的字符串(id号) 26918028-221c-229d5-8735-788e2612349d
celery worker -A celery_task -l info -P eventlet # 使用命令启动worker去刚才提交的执行任务
from celery.result import AsyncResult
from celery_task import app
async = AsyncResult(id='26918028-221c-229d5-8735-788e2612349d', app=app)
if async.successful(): # 根据提交任务返回的字符串去查询执行状态
    print(async.get())
```


# Django - Celery 队列的配置
有些异步任务如果已经知道，肯能会耗时很多，那么会影响队列中的其他任务
采取配置两个队列，一个正常的，一个慢的，互不影响。

![image](https://github.com/Django-27/workspace/blob/master/pic/Django-Celery-%E9%98%9F%E5%88%97%E7%9A%84%E9%85%8D%E7%BD%AE.png)

``` order/tasks.py
class ExportOrderList(AsyncTask):
    """
        导出订单列表
    """

    def _run(self, infos, save_path, url):
        cdn_file_url = order_ctl.organize_export_order_info(infos, save_path, url)
        redis = utils.redis.get()
        redis_key = utils.redis.get_key('big_file_saved', cdn_file_url=cdn_file_url)
        redis.set(redis_key, True, 60*60*3)


export_order_list = ExportOrderList()
```
# 异步发送电子邮件; 异步任务的另一个强大的帮手是 Celery
from threading import Thread

def send_async_email(app, msg):
    with app.app_context():
        mail.send(msg)
        
def send_email(to, subject, template, **kwargs):
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + subject,
    sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
    msg.body = render_template(template + '.txt', **kwargs)
    msg.html = render_template(template + '.html', **kwargs)
    
    thr = Thread(target=send_async_email, args=[app, msg])
    thr.start()
    return thr

# from werkzeug.security import generate_password_hash, check_password_hash
```

