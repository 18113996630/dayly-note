# 									Docker学习

## 什么是Docker

> Docker是一个开源的应用容器引擎。Docker的基础是Linux容器（LXC）等技术。 在LXC的基础上，Docker进行了进一步的封装，在操作系统的层面上实现的虚拟化，让用户不需要去关心容器的管理，使得操作更为简便。用户操作Docker的容器就像操作一个快速轻量级的虚拟机一样简单。 

## Docker的基本概念

- 镜像
	
	> 镜像是一个只读的容器模板，通过已有的镜像可以快速创建一个容器
	
- 容器

	> 容器是通过镜像创建出来的，容器是程序的运行时环境，可以看作一个简易版的Linux系统，容器之间互相隔离
	> 

- 仓库

	> 仓库是存储镜像的地方，可以通过github进行类比理解，仓库也分为公有仓库和私有仓库


## Centos上 Docker 的安装

- 安装前的准备

```
systemctl stop firewalld

systemctl disable firewalld

关闭selinux
	vi /etc/selinux/config

	SELINUX=disabled
	:wq
	getenforce
	
安装iptables
	yum install -y iptables-services
	
	启动iptalbes
	systemctl start iptables
	设置开机自启
	systemctl enable iptables
	清空规则
	iptables -F
	service iptables save

更新内核
	yum update
	
重启
	reboot
```



### 1. script安装(不常用)

```
yum update

curl -sSl https://get.docker.com/ | sh

systemctl start docker

systemctl enable docker
```



### 2. yum安装(常用)


> 更新yum
>
> ​	yum update
>
> 删除旧版本
>
> ​	yum remove docker  docker-common docker-selinux docker-engine
>
> 安装需要的软件包
>
> ​	yum install -y yum-utils device-mapper-persistent-data lvm2
>
> 设置yum源
>
> ​	yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
>
> 查看所有仓库中所有docker版本，并选择特定版本安装
>
> ​	yum list docker-ce --showduplicates | sort -r
>
> 安装：
>
> 1. yum install docker-ce  #安装最新稳定版
> 2. yum install docker-ce-17.12.0.ce  #安装指定版本

启动Docker并设为开机自启

> sudo systemctl start docker.service
>
> sudo systemctl enable docker.service



### 3. rpm安装

```
下载地址：https://download.docker.com/linux/centos/7/x86_64/stable/Packages/

下载：
docker-ce-17.03.0.ce-1.el7.centos.x86_64.rpm  
docker-ce-selinux-17.03.0.ce-1.el7.centos.noarch.rpm

上传至linux，也可以直接使用wget进行下载 

yum install -y *

systemctl start docker

systemctl enable docker
```

###  加速配置(可选)

```
cp /lib/systemd/system/docker.service /etc/systemd/system/docker.service

chmod 777 /etc/systemd/system/docker.service

vim /etc/systemd/system/docker.service

	ExecStart=/usr/bin/dockerd --registry-mirror=https://kfp63jaj.mirror.aliyuncs.com
	
systemctl daemon-reload
systemctl restart docker
ps -ef|grep docker
```

------

## Docker 常用命令

> 使用**docker --help**查看命令帮助


### 镜像命令
- 列出本机所有镜像
```
docker images
  -a, --all       显示所有镜像 (及其中间映象层)
      --digests     显示摘要
      --no-trunc      不截断输出（显示完整信息）
  -q, --quiet       静默模式（仅显示镜像 ID）
```

- 查找镜像
```
docker search <image>
```

- 拉取镜像
```
docker pull <image>
```

- 删除镜像
```
docker rmi <image>
```

- 存入镜像(导出镜像文件到本地)

```shell
docker save [OPTIONS] IMAGE [IMAGE...]
```

- 载入镜像(将本地镜像文件导入到镜像库)

```
docker load [OPTIONS]
```



### 容器命令

- 列出本机容器列表
```
docker ps
  -a, --all             显示所有容器（包括已停止的容器）
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display numeric IDs
  -s, --size            Display total file sizes
```

- 新建并运行容器
```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
  --rm					Automatically remove the container when it exits
  -d, --detach      	Run container in background and print container ID
  -i, --interactive 	Keep STDIN open even if not attached
  -t, --tty       		Allocate a pseudo-TTY
  -p, --publish list    Publish a container's port(s) to the host
  -P, --publish-all     Publish all exposed ports to random ports
  -v, --volume list     Bind mount a volume
  --name string   		Assign a name to the container

docker run 0f3e07c0138f /bin/echo 'helloworld'    
```

- 退出容器
```
exit            退出并停止容器
Ctrl + P + Q        退出但不停止容器
```

- 停止容器
```
docker stop <container> 停止容器
docker kill <container> 强制停止容器
```

- 启动、重启容器
```
docker start <container>
docker restart <container>
```

- 删除容器
```
docker rm <container>   删除容器（已停止的）
docker rm -f <container>  强制删除容器（包括正在运行的容器）
```

- 查看容器信息

```
docker inspect containerId
```



## Docker镜像的创建

1. ##### 修改已有镜像

```
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]     使用该命令向仓库提交一个新的镜像
```



2. ##### 利用Dockerfile来创建镜像

> Dockerfile基本的语法：
>
> 使用#来注释
> FROM指令告诉Docker使用哪个镜像作为基础
> 接着是维护者的信息
> RUN	开头的指令会在创建中运行，比如安装一个软件包，在这里使用apt-get来安装了一些软件

**示例：**

```shell
# Base image
FROM centos
# 维护人员信息
MAINTAINER xxx@163.com
# 执行安装命令
RUN yum install lrzsz
```

- 创建**Dockerfile**文件

  ```shell
  # Base image
  FROM centos
  # 维护人员信息
  MAINTAINER xxx@163.com
  # 执行安装命令
  RUN yum install -y lrzsz
  ```

- 编写完成Dockerfile后可以使用 **docker build**命令来生成镜像

  >  docker build [OPTIONS] PATH | URL | - 

  ```
  docker build -t 'centos_lrzsz:v1' .
  
  -t 指定repository和tag信息
  .表示Dockerfile文件的路径
  如果文件名为Dockerfile可不指定文件名，否则需指定文件名
  ```



## 数据卷

> 使用**docker run -v**进行数据卷的指定，实际上就是指定共享文件
>
> **数据卷的修改不会影响镜像，数据卷会一直存在除非没有容器使用它**

- 使用方法

  ```
  docker run -p 6379:6379 -v /usr/etc/redis/conf/redis.con:/etc/redis/redis.conf -d  --name redis redis
  ```



## 数据卷容器

> 如果有一些持续更新的数据需要在容器中进行共享，则最好创建数据卷容器，供其他的容器挂载

- 使用方法

  ```
  # 创建一个基于redis的数据卷容器
  docker run -d -v /dbdata --name test_db redis
  # 创建其他容器进行数据卷容器的挂载
  docker run -d --volumes-from test_db redis --name other_db
  ```

> 使用 --volumes-from 参数所挂载数据卷的容器自己并不需要保持在运行状态

## 网络功能

### 外部访问容器-端口映射

> 容器有自己的内部网络和ip地址（使用docker inspect可以获取所有的变量）
>
> 在容器中运行的应用可以通过**-p**来指定端口映射，**-P**来随机指定一个端口来代理容器中的端口
>
> 使用**-p**可以指定多个端口

- 使用方法

  ```
  # 随机指定端口进行绑定
  docker run -P --name portredis -d redis
  # 使用指定端口进行端口绑定
  docker run -p 6390:6379 --name portredis2 -d redis
  
  CONTAINER ID   PORTS                     NAMES
  0d480854a2a7   0.0.0.0:6390->6379/tcp    portredis2
  81df0eb1dd1b   0.0.0.0:32769->6379/tcp   portredis
  ```

### 容器互联

> 使用**--link**参数可以让容器之间安全的进行交互
>
> 使用方法：**--link name:alias**，其中name是要链接的容器的名称，alias是这个连接的别名

```
docker run -d --name redis redis

# 使用link连通redis_two与redis
docker run -P --name redis_two --link redis:redis redis
```

Docker通过2种方式为容器公开连接信息：

- 环境变量
- 更新/etc/hosts文件

使用**env**命令查看容器环境变量

```
> docker run --rm -P --name redis_three --link redis:reids redis env

REIDS_PORT=tcp://172.17.0.2:6379
REIDS_PORT_6379_TCP_ADDR=172.17.0.2
HOSTNAME=941831e5f779
REIDS_NAME=/redis_three/reids
REIDS_PORT_6379_TCP_PORT=6379
REDIS_DOWNLOAD_SHA=61db74eabf6801f057fd24b590232f2f337d422280fd19486eca03be87d3a82b
HOME=/root
REIDS_PORT_6379_TCP_PROTO=tcp
REIDS_ENV_REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-5.0.7.tar.gz
REIDS_ENV_REDIS_VERSION=5.0.7
REIDS_PORT_6379_TCP=tcp://172.17.0.2:6379
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
REIDS_ENV_REDIS_DOWNLOAD_SHA=61db74eabf6801f057fd24b590232f2f337d422280fd19486eca03be87d3a82b
REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-5.0.7.tar.gz
REIDS_ENV_GOSU_VERSION=1.11
REDIS_VERSION=5.0.7
GOSU_VERSION=1.11
PWD=/data

其中REDIS_开头的环境变量是redis容器提供给redis_three容器所使用的环境变量，为连接别名的大写
```



## Docker安装Redis主从集群

> 参考博客：<https://my.oschina.net/dslcode/blog/1936656
>
> <https://blog.csdn.net/yingziisme/article/details/100088226>
>
> <https://www.jianshu.com/p/062669be85da>

- 拉取最新稳定版redis镜像

  > docker pull redis

- 创建**redis-cluster.tmpl**，该文件是redis.conf模板文件

  > cd 
  >
  > mkdir -p docker/redis-cluster
  >
  > cd docker/redis-cluster
  >
  > vi redis-cluster.tmpl

  ```
  # bind 127.0.0.1
  protected-mode no
  port ${PORT}
  daemonize no
  dir /data/redis
  appendonly yes
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 15000
  ```

- 创建docker内部网络

  ```
  docker network create redis-cluster-net
  ```

- 编写脚本**conf-redis.sh**并进行授权

  > 1. 创建本地目录文件，并生成data目录及**redis.conf**文件
  > 2. 使用docker运行redis，并进行端口的绑定和文件挂载
  > 3. 设置redis集群

  ```shell
  #!/bin/bash
  echo 'start to generate master/slave directory...'
  # 创建 master 和 slave 文件夹
  for port in `seq 7000 7005`; do
      ms="master"
      if [ $port -ge 7003 ]; then
          ms="slave"
      fi
      mkdir -p ./$ms/$port/ && mkdir -p ./$ms/$port/data \
      && PORT=$port envsubst < ./redis-cluster.tmpl > ./$ms/$port/redis.conf;
  done
  echo 'direcotry has been generated...'
  echo 'start to run redis instance by docker...'
  # 运行docker redis 的 master 和 slave 实例
  for port in `seq 7000 7005`; do
      ms="master"
      if [ $port -ge 7003 ]; then
          ms="slave"
      fi
      docker run -d -p $port:$port -p 1$port:1$port \
      -v $PWD/$ms/$port/redis.conf:/data/redis.conf \
      -v $PWD/$ms/$port/data:/data/redis \
      --restart always --name redis-$ms-$port --net redis-cluster-net \
      redis redis-server /data/redis.conf;
  done
  echo 'redis has been started...start to set cluster...'
  # 组装masters : slaves 节点参数
  matches=""
  for port in `seq 7000 7005`; do
      ms="master"
      if [ $port -ge 7003 ]; then
          ms="slave"
      fi
      matches=$matches$(docker inspect --format '{{(index .NetworkSettings.Networks "redis-cluster-net").IPAddress}}' "redis-$ms-${port}"):${port}" ";
  done
  # 创建docker-cluster
  docker run -it --rm --net redis-cluster-net redis redis-cli --cluster create $matches --cluster-replicas 1
  echo 'successfully'
  ```
  > chmod 777 conf-redis.sh

- 执行脚本完成配置，【中途会输一次**yes】**

  > ./conf-redis.sh

  **出现successfully即表明redis主从集群搭建成功**

```
停止redis集群：
docker ps | grep redis |awk '{print $1}'|xargs docker stop
重启redis集群：
docker ps -a | grep redis |awk '{print $1}'|xargs docker restart
```

## Docker安装









