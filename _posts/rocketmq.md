# 1 RocketMQ 下载
## 1.1 创建mq文件夹，解压bin和src文件
```javascript
/Users/sunny/mq/
/Users/sunny/mq/bin  下载的bin文件
/Users/sunny/mq/src  下载的src文件
```
## 1.2 创建其他的文件夹
```javascript
/Users/sunny/mq/bin/store 新建的文件夹
/Users/sunny/mq/bin/store/logs 新建日志文件
/Users/sunny/mq/bin/store/commitlog 新建存储数据文件
/Users/sunny/mq/bin/store/consumerqueue 消息信息
/Users/sunny/mq/bin/store/index 存储消息的索引数据
```
## 1.3 修改配置文件,
```javascript
/Users/sunny/mq/bin/conf/2m-2s-async/broker-a.properties 进行配置目录修改
linux: sed -i 's#${user.home}#/Users/sunny/mq/bin#g' *.xml # 批量替换文件内容
mac  : sed -i "" 's#${user.home}#/Users/sunny/mq/bin#g' *.xml
/Users/sunny/mq/bin/bin/runserver.sh 配置中都改成1g
/Users/sunny/mq/bin/bin/runbroker.sh
```
## 1.3 启动，jps查看状态
```javascript
nohup sh mqnamesrv &     <--启动 nameserver
nohup sh mqbroker -c /Users/sunny/mq/bin/conf/2m-2s-async/broker-a.properties > /dev/null 2>&1 &    <--启动 broker
➜  bin ./mqbroker -m  # 查看 broker 信息查看
-->其他方式启动： bash mqnamesrv -n localhost:9876 启动 name server  <- mqshutdown namesrv
-->其他方式启动： bash mqbroker -n localhost:9876  启动 broker       <- mqshutdown broker(先关broker、在关namesrv)
```
## 1.5 控制台的安装
```javascript
/Users/sunny/mq/rocketmq-externals/rocketmq-console 进入控制台源码
/Users/sunny/mq/rocketmq-externals/rocketmq-console/src/main/java/org/apache/rocketmq/console/App.java 修改文件
/Users/sunny/mq/rocketmq-externals/rocketmq-console/src/main/resources/application.properties  可以修改端口8082
rocketmq.config.namesrvAddr=localhost:9876 关联name server addr
rocketmq.config.dataPath=/Users/sunny/mq/rocketmq-externals/rocketmq-console/data  指定一个data path
rocketmq.config.isVIPChannel=false
```
## 1.6 打包控制台
```javascript
cd /Users/sunny/mq/rocketmq-externals/rocketmq-console
mvn clean package -Dmaven.test.skip=true  # 注意打包时的jdk和运行时的jdk要一致，否则会报错
--> 结果 Building jar: /Users/sunny/mq/rocketmq-externals/rocketmq-console/target/rocketmq-console-ng-1.0.1-sources.jar
--> 运行 java -jar rocketmq-console-ng-1.0.1.jar  --> [mq-console](http://localhost:8082/#/)
# 注意打包使用的java版本，必须与运行时的java版本一致
# mvn dependency:analyze[tree/list] 查看状态
```
## 1.7 安装python客户端
```javascript
pip install rocketmq-client-python==2.0.0
```
## 1.8 安装依赖总结
```javascript
1. 64bit OS, Linux/Unix/Mac is recommended;
2. 64bit JDK 1.8+;
3. Maven 3.2.x;
4. Git;
5. 4g+ free disk for Broker server
6. pip install rocketmq-client-python==2.0.0
7. install librocketmq [https://github.com/apache/rocketmq-client-python]
8. rocketmq-4.7.0
```

# 2 RocketMQ 概念

亿级消息的堆积能力，依然保持写入低延迟
rocketmq支持万级别的topic（broker和nameserver心跳时，万级别topic可能需要发送几十m的数据）
rocketmq使用netty做的长连接，netty单机长连接数支持万级别----rocketmq长连接理论满足业务要求
1.要知道RocketMQ原生就是支持分布式的，而ActiveMQ原生存在单点性。
2.RocketMQ可以保证严格的消息顺序，而ActiveMQ无法保证！
3.RocketMQ提供亿级消息的堆积能力，这不是重点，重点是堆积了亿级的消息后，依然保持写入低延迟！
4.丰富的消息拉取模式（Push or Pull）
5.在Metaq1.x/2.x的版本中，分布式协调采用的是Zookeeper，而RocketMQ自己实现了一个NameServer，更加轻量级，性能更好！
6.消息失败重试机制、高效的订阅者水平扩展能力、强大的API、事务机制等等（后续详细介绍）

- 单Master模式：无需多言，一旦单个broker重启或宕机，一切都结束了！很显然，线上不可以使用。
- 多Master模式：全是Master，没有Slave。当然，一个broker宕机了，应用是无影响的，缺点在于宕机的Master上未被消费的消息在Master没有恢复之前不可以订阅。
- 多Master多Slave模式（异步复制）：高可用！采用异步复制的方式，主备之间短暂延迟，MS级别。Master宕机，消费者可以从Slave上进行消费，不受影响，但是Master的宕机，会导致丢失掉极少量的消息。
- 多Master多Slave模式（同步双写）：区别点在于采用的是同步方式，也就是在Master/Slave都写成功的前提下，向应用返回成功，可见不论是数据，还是服务都没有单点，都非常可靠！缺点在于同步的性能比异步稍低。

## 2.1 Group 机制
通过Group机制实现负载均衡，其中一个Consumer Group有3个实例（3个进程或3台机器），那么每个实例均摊3条消息

## 2.2 双Master模式架构
### 2.2.1 第一步：修改 /etc/hosts 文件，确保互相可以ping通
``` JavaScript
192.168.99.121 rocketmq-nameserver-1
192.168.99.121 rocketmq-master-1
192.168.99.122 rocketmq-nameserver-2
192.168.99.122 rocketmq-master-2
```
### 2.2.2 第二步，解压并创建存储路径
```JavaScript
tar -xvf rocketmq-all-4.7.0-bin-release.zip
mkdir -p alibaba-rocketmq/store/{commitlog,consumequeue,index}
```

# 3 rocketmq-client-python==2.0.0
- 订阅关系一致指的是同一个消费者 Group ID 下所有 Consumer 实例的处理逻辑必须完全一致。一旦订阅关系不一致，消息消费的逻辑就会混乱，甚至导致消息丢失。
- 同一个消费者 Group ID 下所有的实例需在以下两方面均保持一致：订阅的 Topic 必须一致/订阅的 Topic 中的 Tag 必须一致