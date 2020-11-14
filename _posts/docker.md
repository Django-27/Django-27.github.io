# Docker，服务容器化过程
容器是一种轻量级、可移植的为应用程序提供了隔离的运行空间；
每个容器包含一个独享的完整用户环境，一个容器内的环境变动不会影响另一个容器；是应用程序在几乎任何地方以相同的方式运行；
技术方面，容器通过一系列系统级别的机制实现；如namespace实现空间隔离性，UnionFS实现文件隔离性，cgroups实现应用隔离性；
容器之间通过共享同一个系统内核提升内存使用率；
## 命名空间（namespace)，cgroups（control groups）、UnionFs（Union File Systems）、容器格式（container format）
- （空间隔离性）运行容器时，docker会为容器创建一组命名空间，每个容器都是一个独立的命名空间，仅有自己命名空间内的访问权限，pid、net、ipc、mnt、uts命名空间
- （应用隔离性）通过cgoups技术实现不同应用之间的隔离性，每个应用只能访问属于自己的资源；确保将硬件资源共享给所有容器，并进行限制，如每个容器可访问的内存大小
- （文件隔离性）UnionFS是docker创建层时采用的文件系统，使得docker轻量级、启动速度快；也可以使用其他类型的UnionFS，如AUFS、btrfs、vfs和DeviceMapper
- 容器格式：将namespace、cgroups和UnionFS封装称container format，称为容器；默认容器类型是libcontainer，也可以是BSD jails或者Solaris Zones
## docker引擎三大组件
docker是C/S结构的架构，客户端通过与后台服务交互，来编译、运行和发布容器；docker的客户端可以连接到本地的docker服务，也可以连接到远程的docker服务
docker客户端通过REST接口来与后台服务通信，它使用UNIX Socket连接或者网络接口实现
- Docker后台服务（Docker Daemon）长时间运行在后台的守护进程，是Docker的核心服务，通过dockerd命令与它交互通信
- REST接口（REST API)，程序可以通过REST接口来访问后台服务，向他发送操作指令
- 交互式命令行界面（Docker CLI)，通过命令行实现与docker的交互
```
（容器）-----------|      dockerr cli         |----------（镜像）
                   |      REST API        |
（网络）--------------|  docker daemon   |--------------（磁盘卷）
```
### 镜像（image）
只读的指令模板，用于创建docker容器（container)；通常一个镜像继承另一个镜像，然后扩展自定义的指令；
通过dockerfile文件，定义如何创建和运行镜像，其中每个指令在镜像中都会创建为一个层（layer）；
当修改dockerfile后重新编译时，仅有被修改的层会重新编译，保障了docker镜像的轻量级、体积小、速度快；
### 容器（container)
是进行运行的一个实例，可使用api或cli创建、运行、停止、移动或删除；
可以为容器绑定一个或多个网络（network)，挂载磁盘卷（volume)，也可以继承它创建一个新的镜像；
通常容器与另一个容器，或与宿主机都是相对独立和隔离，容器停止运行，其中所有改变的状态如果没有保存，都将消失；
### Docker服务（service)
可在多个docker后台服务（daemon)中伸缩扩展容器，共同组成一个多主多从模式的集群；
集群中的每个成员都是一个docker服务后台服务，之间通过docker接口通信；
可通过docker服务（service)定义集群参数，如集群中容器复本的个数；
默认情况下，集群负载是面向所有容器节点，对用使用者来说，docker集群就是一个大实例；

## 常用命令
```
docker --version
docker search 镜像名称        # docker pull 镜像名称:标签， docker rmi 镜像ID， 
docker-compose --version
docker-machine --version
docker version
docker image                 # 查看本地镜像
docker ps -a                 # 查看所有的容器状态， 等同于 docker container ls -a
docker run hello-world       # 运行docker自带的hello world程序
docker run -it ubuntu bash   # 运行ubuntu容器，并进入bash（it指定伪终端）
docker cp 宿主机目录 容器名称:容器目录 # 文件拷贝，docker cp 容器名称:容器目录 宿主机目录
docker inspect web
···
docker run -d -p 80:80 --name webserver nginx  # 以后台模式在80端口启动nginx容器
···
docker logs 容器名 
docker-compose up -d         # 1*gcf7tH14oh0fvrKp
docker pull wordpress:latest # 拉去镜像
docker stop 容器名            # 停止运行
docker rm 容器名              # 删除容器
```
## Docker后台服务的管理
dockerd是管理容器的常驻进程，dockerd命令启动Docker后台服务，dockerd -D命令以调试模式启动Docker后台服务（默认配置文件：/etc/docker/daemon.json)
## Docker的客户端命令
### image 镜像的常用命令
```
docker images [options] [repository[:tag]]  # 查看本地镜像，docker image java:8
docker image pull [options] name[:tag | @digest] # 拉取镜像，默认是docker hub，也可指定仓库, docker image pull registry.hub.qiyuan.com/qiyuan/java8:latest
docker run [options] image [command] [arg...] # 运行镜像，docker run --name test -it debian
 .                                             # -it bash 创建伪终端
 .                                             # -w /my/workspace/ 为容器设置工作空间， 
 .                                             # --storage-opt 120G 容器存储容量120G
 .                                             # -v /doesnt/exist:/foo 将/doesnt/exist目录挂载到容器/foo目录下
 .                                             # -w /foo 将当前的工作空间切换到/foo目录下
 .                                             # -p 127.0.0.1:80:8080 将容器的8080端口绑定到主机的12.0.0.1：80地址上
 .                                             # --expose 80 仅绑定容器的80端口，而主机未映射端口
 .                                             # --env-file -e --env 设置环境变量，优先级，--env会覆盖前面的，-e会覆盖--env-file
.                                              # -e MYVAR1 --env MYVAR2=foo --env-file ./env.list 没有等号的相当于export赋值$MYVAR1
.                                              # -l my-lable --lable com.qiyuan.cool=abc 设置容器元数据,没有等号是空数据
.                                              # 查看容器的lables，docker inspect -f '{{ index .Config.Labels }}' container_id
.                                              # --add-host=docker:1.123.0.1 向容器hosts文件（/etc/hosts）添加配置
docker commit [options] container [repository[:tag]]  # 通过修改容器来创建镜像，不包含挂载磁盘卷中的数据
.                                              # docker commit c12345678912 book/test3:v3 通过指定容器id创建仓库名为book/test3标志v3的镜像
.                                              # --change "ENV DEBUG true" 
docker image build [options] path | url | -   # commit创建镜像简单，但是不利于分享；可以用 Dockerfile 文件，后build创建
.    From docker/qiyuan:latest                              # From告诉docker此镜像基于docker/qiyuan
.    RUN apt-get -y update && apt-get install -y fortunes   # 添加fortunes程序，用来在命令行输出
.    CMD /usr/games/fortune -a | cowsay                     # 镜像加载完就开始运行，docker build -t docker-qiyuan Dockerfile_path
.                                                           # -t 指定新的镜像的标记信息， cowsay 打印“鲸说”信息
docker login [options] [server]  # 登录 Docker Hub 账户， docker login --username=xxx --email=xxx
.                                 # docker push book/test
```
### container 容器的常用命令
```
docker run debian /bin/echo 'hello qiyuan'        # 新建并启动容器
docker run -t -i debian /bin/bash                 # 新建并启动容器，并启动bash
.                                                 # -d 后台运行
docker logs [optinos] container                   # 获取容器日志信息
docker attach [options] container                 # 进入容器
docker exec [options] container command [arg...]  # 在运行的容器中执行命令，如果容器已经暂停命令会出错
docker stop [options] ocntainer [container ...]   # 停止容器
docker rm   [options] ocntainer [container ...]   # 删除容器；docker rm --link /webapp/redis 删除webapp容器和redis容器间的所有网络桥接
.                                                 # --forse 强制删除正在运行的容器
.                                                 # docker rm $(docker ps -a -q) 删除所有已停止的容器
```
### volume 磁盘卷的常用命令
```
docker volume create hello # 创建磁盘卷（容器可以从中读取和存储数据，未指定名称会随机生成）
docker run -d -v hello:/world busybos ls /world  # 然后配置给容器使用，磁盘卷被加载到容器的/world目录下（注意：docker不支持在容器下挂载相对路径）
.                          # 磁盘卷名称已经存在，就是同一个磁盘卷，实现了共享数据（多个容器，可以有的读，可以有的写）
.                          # --opt type=nfs 创建一个驱动为nfs的磁盘卷
.                          # --opt o=size=100m,uid=1000 大小为100MB字节，uid为1000
.                          # --opt device=/dev/sda2 指定device参数为/dev/sds2
.                          # --opt o=addr=192.168.1.1,rw --opt device=:/path/to/dir 从192.168.1.1上以读写模式加载/path/to/dir
docker volume inspect hello  # 显示磁盘卷的详细信息，默认JSON输出
.                            # --format '{{ .Mountpoint }}' 使用Go语音模板进行格式胡输出
docker volume ls     # 以列表显示磁盘卷, -f 过滤， (-f name=rose -f driver=local --label istimelord=no)            .                
docker volume prune  # 删除所有未使用的磁盘卷
docker volume rm hello 
```
### network 网络的常用命令
```
docker network create -d bridge my-bridge-network      # 创建一个bridge网络，只是Docker引擎上上独立的网络
docker network create -d overlay my-multehost-network  # 创建一个跨多个Docker主机通信网络，必须使用overlay网络
docker network connect multi-host-network contnainer1  # 为一个运行的容器添加网络
.                                                      # --ip 1.2.3.111 同时制定IP地址
docker network disconnet multi-host-network container1 # 将容器从网络上断开
docker network ls 
docker network prune
docker network rm my-network
```
### service 服务的常用命令
```
docker service create --name redis1 redis:3.0.6  # 创建一个名为redis1的服务
.                                                # --mode global 指定全局服务
docker service ls # service 两种部署方式：副本模式、全局模式
.                 # 副本模式：指定多个相同的任务队列，同时提供相同的服务
.                 # 全局模式：每个节点运行一个任务，当集群中有新节点增加，会自动在新节点上穿件任务
docker service create --name redis --replicas=5 redis:3.0.6  # 创建5个副本的Redis服务
docker service create --replicase 10 --name redis \
.                     --update-delay 10s --update-parallelism 2 \
.                     redis:3.0.6   # 创建灰度更新的服务，每次更新服务，最多每批次更新两次，间隔10s
.                                   # --env MYVAR=foo 为所有服务设置环境变量
.                                   # --hostname myredis 
.                                   # --label com.qiyuan.cool="bar" --label bar=baz 设置label元数据
docker service ls 
docker service ps service_name  # 列出某个服务的所有任务
docker service rm service_name
docker service scale service_name=50 # 对副本模式下的服务进行扩展，全局模式服务不支持
docker service update --limit-cup 2 redis  # 对服务进行更新，限制CPU数为2
.                                          # --rollback 可以回滚服务到最近一次的更新
```
### swarm 集群的常用命令
```
docker swarm init --advertise-addr 192.168.1.10 # 集群初始化，并指定用于服务发现的地址
.                         # 初始化集群，当前Docker引擎作为集群的管理节点，当前集群为单节点集群，只拥有一个管理节点
.                         # docker swarm init 会生成2个token，worker token 和 manager token 
.                         # docker swarm join --token xxx 当新节点加入节点是，节点是worker还是manager由传入参数的token决定
docker swarm join-token [optins] worker | manager  # token 的查看
docker swarm join [options] --token xxx host:port  # 加入集群，一个集群一般只需要3到7个管理节点，同时管理节点必须有稳定的host和静态的ip地址
docker swarm leave [options] # 退出集群,worker节点上执行直接退出
.                            # 在manager节点上执行，需--force；如果集群为单节点，则该集群解散；
.                            # 一般不适用--forse操作manager节点，而是docker node demote 命令先降级为worker节点再移除
docker node ls
docker swarm update 
```
## docker compose 编排工具的使用
Docker Compose（简称Compose）是一个用于定义和运行多个Docker应用程序的工具；
可以在compose文件中定义多个应用服务（service），并通过简单命令穿件和启动所有服务；
Compose 是很好的CI持续集成工具，能很方便的部署开发development、测试testing、线上生产环境staging；
（CI全名Continuous Integration，持续集成，CD全名是Continuous Deployment，持续部署）
### docker-compose 命令
```
docker-compose ps 
docker-compose logs
docker-compose config                 # 检查语法是否错误
docker-compose build                  # 构建或者重新构建服务（重新编译）
docker-compose start service_name     # stop，down, rm, 等等命令，docker-compose -h
docker-compose run service_name bash  # 重要：在一个服务上执行一个命令，这里是进入服务的bash
docker-compose -f docker-compose.yml -f production.yml up -d # 增加生产环境需要的配置项在production.yml文件
.                                                            # 多个.yml文件有顺序，后面的会应用到前面的配置的
.                                                            # --no-deps只编译、更新应用程序，不编译、不更新应用程序的依赖
```
###  docker-compose 应用容器例子
Docker Compose 可以轻松、高效的管理容器，它是一个用于定义和运行多容器 Docker 的应用程序工具
Docker Compose 将所管理的容器分为三层，分别是工程（project）、服务（service）、容器（container）
Docker Compose 运行目录下的所有文件（docker-compose.yml）组成一个工程,一个工程包含多个服务，服务中定义容器运行的镜像、参数、依赖，一个服务可包括多个容器实例
- 步骤一：创建目录       cd my_docker_wordpress   定义应用程序的环境
- 步骤二：创建yml文件    docker-compose.yml       定义构成应用程序的服务,在隔离环境中一起运行
- 第三部：拉取镜像并启动  docker-compose up -d     启动并运行整个应用程序
```
version: '3.1'                                       # 指定丢docker版本的[支持](https://docs.docker.com/compose/compose-file/#deploy)                                             
services:                                            # 多个服务的集合                 
    db:                                                                                             
        image: mysql:5.7                             # 指定服务所使用的镜像
        restart: always                                                                             
        environment:                                 # 环境变量配置，可以用数组或字典两种方式 
            MYSQL_USER: wordpress                                                                   
            MYSQL_PASSWORD: wordpress                                                               
            MYSQL_DATABASE: wordpress                                                               
            MYSQL_ROOT_PASSWORD: somewordpress                                                      
        volumes:                                     # 卷挂载路径                                            
            - db:/var/lib/mysql                                                                     
                                                                                                    
    wordpress:                                                                                      
        image: wordpress:latest                                                                     
        restart: always                                                                             
        environment:                                                                                
            WORDPRESS_DB_HOST: db:3306                                                              
            WORDPRESS_DB_USER: wordpress                                                            
            WORDPRESS_DB_PASSWORD: wordpress                                                        
        ports:                                       # 对外暴露的端口定义
            - 8000:80                                # 端口映射，8000表示宿主，80表示容器中的端口                         
        volumes:                                      
            - wordpress:/var/www/html                                                               
volumes:                                                                                            
    db:                                                                                             
    wordpress:
```
###  docker-compose python 例子
#### 1 准备一个简单的python程序
```
mkdir mytest
cd mytest
touch app.py
from flask import Flask
from redis import Redis
app = Flask(__name__)
redis = Redis(host='redis', port=6379)
@app.route('/')
def hello():
    count = redis.incr('visit')
    return "hello you're {} visiter!".format(count)
if __name__ == "__main__":
    app.run(host='0.0.0.0', debug=True)
# 并创建依赖库文件 requirements.txt
flask
redis
```
#### 2 在工作空间创建Dockerfile文件
```
FROM python:3.4-alpine               # 指定一个基础镜像
ADD . /code                          # 加载当前目录下文件到容器的 /code 目录下
WORKDIR /code                        # 改变当前工作空间为 /code 
RUN pip install -r requirements.txt  # 安装依赖
CMD ["python", "app.py"]             # 容器启动执行的命令，即运行 python 程序
```
#### 3 使用 compose 工具定义服务，创建docker-compose.yml文件
定义2个服务，web服务和redis服务； 容器内的5000端口与宿主机5000端口映射；挂载当前目录到容器的/code目录，方便修改代码而不用重新编译镜像；
```
version: '2'
service:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
  redis:
    image: "redis:alpine"
```
#### 4 使用compose工具编译并允许程序
```
docker-compose up
```
#### 5 更新应用程序
使用volumes挂载了应用程序代码，只要修改应用程序代码，就可以看到小阿哥，不需要从新编译镜像文件；
```
return 'hello qiyuan updated {}times!'.format(count)
```
## Dockerfile与docker-compose.yml文件
Dockerfile，是一个按一定规则编写的包含多行命令的文件，使用Dockerfile可以快速的构建一个定制的镜像，亦可避免单纯命令行的重复劳动；
使用Dockerfile， 每一条指令都会创建一个镜像层，它能将每一步改变内容的命令都做commit操作，便于查看history层；
更改容器，只需在Dockerfile中更改就可以重新生成镜像，不需要重头来一次；分享更加方便，可以只分享Dockerfile文件，服务器便可以生成一模一样的镜像。

Docker Compose File，按照docker官方的建议，每一个容器只启动一个进程，这样便于管理和解耦；
而在生产部署的时候，我们的一个应用不太可能只有一个进程，除了代码应用的主进程外，你可能还需要开启reids、mysql、nginx等；
即Dockerfile记录单个镜像的构建过程，docker-compose.yml记录一个项目的构建过程；
