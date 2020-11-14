# 1 消息队列
- 消息队列是一种异步的服务间通信方式，用途就是：**解耦、异步、消峰/限流**（其他还有：消息驱动、日志处理（kafka）、消息通讯（点对点通信、聊天室））
- 缺点是系统可用性降低、系统复杂性增加；

ActiveMQ | RabbitMQ |  RocketMQ | kafka
:-:| :-: | :-: | :-: 
开发语言	| java |	erlang |	java |	scala
单机吞吐量	| 万级 |	万级 |	10万级	| 10万级
时效性	 | ms级	| us级 | 	ms级  |	ms级以内
可用性	| 高(主从架构)	| 高(主从架构)	| 非常高(分布式架构)	| 非常高(分布式架构)

## Celery 使用
任务队列是一种在线程或机器间分发任务的机制,消息队列的输入是工作的一个单元，称为任务
- Worker 持续监视队列中是否有需要处理的新任务
- Broker (中间人)在客户端和 Worker 间斡旋-
- Celery 系统可包含多个Worker和Broker（RabbitMQ，Redis），以获得高可用性和横向扩展能力
- Celery 可以单机运行，也可以在多台机器上运行，甚至可以跨越数据中心运行
```
from celery import Celery
app = Celery('hello', broker='amqp://guest@localhost//')
@app.task
def hello():
    return 'hello world'
```

## 消息队列协议
- AMQP-高级消息队列协议，具有可靠性和互操作性
- MQTT-消息队列遥测传输，具有简单、集中特点，专门针对资源受限的设备和低带宽、高延迟网络
- STOMP-简单/流式文本导向消息传递协议，基于文本

## 以rcoketMQ为例，保证系统可用性
他的集群就有多master 模式、多master多slave异步复制模式、多 master多slave同步双写模式。多master多slave模式部署架构图(网上找的,偷个懒，懒得画):
image
其实博主第一眼看到这个图，就觉得和kafka好像，只是NameServer集群，在kafka中是用zookeeper代替，都是用来保存和发现master和slave用的。通信过程如下:
Producer 与 NameServer集群中的其中一个节点（随机选择）建立长连接，定期从 NameServer 获取 Topic 路由信息，并向提供 Topic 服务的 Broker Master 建立长连接，且定时向 Broker 发送心跳。Producer 只能将消息发送到 Broker master，但是 Consumer 则不一样，它同时和提供 Topic 服务的 Master 和 Slave建立长连接，既可以从 Broker Master 订阅消息，也可以从 Broker Slave 订阅消息。

## 消息队列包括两种模式，点对点模式（point to point）和发布/订阅模式（publish/subscribe）
### 点对点模式
- 包含三个角色：消息队列、发送者 (生产者)、接收者（消费者）；
- 每个消息只有一个接收者，即一旦被消费，消息就不再在消息队列中；
- 发送者和接收者间没有依赖性，发送者发送消息之后，不管有没有接收者在运行，都不会影响到发送者下次发送消息；
- 接收者在成功接收消息之后需向队列应答成功，以便消息队列删除当前接收的消息；
### 发布/订阅模式
- 包含三个角色：角色主题（Topic）、发布者(Publisher)、订阅者(Subscriber)
- 发布者将消息发送到Topic,系统将这些消息传递给多个订阅者；
- 每个消息可以有多个订阅者；
- 发布者和订阅者之间有时间上的依赖性。针对某个主题（Topic）的订阅者，需要提前订阅该角色主题，并保持在线运行，才能消费发布者的消息。

### 消费者怎么从消息队列里边得到数据
- 生产者将数据放到消息队列中，主动叫消费者去拿(俗称push)
- 消费者不断去轮训消息队列，看看有没有新的数据，如果有就消费(俗称pull)

### 解决高可用 (连接丢失或失败时Worker和客户端会重试，中间人通过主从同步、主主同步)集群、分布式
### 解决数据丢失 同步持久化、异步持久化
### 数据一致性 分布式事务