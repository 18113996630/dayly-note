

# 1. 下载spark源码，导入idea并进行编译

## 1.下载与服务器版本相同的spark源码

**地址：**` https://github.com/apache/spark/releases `

## 2.使用maven编译

1. `export MAVEN_OPTS="-Xmx4g -XX:ReservedCodeCacheSize=1024m"`
2. `build/mvn -T 4 -DskipTests clean package`

参考` http://spark.apache.org/docs/latest/building-spark.html `

错误1：

```
Failed to execute goal org.apache.maven.plugins:maven-antrun-plugin:1.8:run (default) on project spark-core_2.11: An Ant BuildException has occured: Execute fa
iled: java.io.IOException: Cannot run program "bash" (in directory "E:\workspace\source_code\spark-2.3.4\core"): CreateProcess error=2, 系统找不到指定的文件。
[ERROR] around Ant part ...<exec executable="bash">... @ 4:27 in E:\workspace\source_code\spark-2.3.4\core\target\antrun\build-main.xml
```

解决办法：

```
在源码根目录打开git-bash，执行
./build/mvn -Pyarn -Phadoop-2.7 -Dhadoop.version=2.7.3 -DskipTests clean package
```



# 2. shell脚本的调试

## 1. 调试start-master.sh

#### 1. 执行 sh -x start-master.sh

```
+ '[' -z /usr/local/spark ']'
+ CLASS=org.apache.spark.deploy.master.Master
+ [[ '' = *--help ]]
+ [[ '' = *-h ]]
+ ORIGINAL_ARGS=
+ . /usr/local/spark/sbin/spark-config.sh
++ '[' -z /usr/local/spark ']'
++ export SPARK_CONF_DIR=/usr/local/spark/conf
++ SPARK_CONF_DIR=/usr/local/spark/conf
++ '[' -z '' ']'
++ export PYTHONPATH=/usr/local/spark/python:
++ PYTHONPATH=/usr/local/spark/python:
++ export PYTHONPATH=/usr/local/spark/python/lib/py4j-0.10.7-src.zip:/usr/local/spark/python:
++ PYTHONPATH=/usr/local/spark/python/lib/py4j-0.10.7-src.zip:/usr/local/spark/python:
++ export PYSPARK_PYTHONPATH_SET=1
++ PYSPARK_PYTHONPATH_SET=1
+ . /usr/local/spark/bin/load-spark-env.sh
++ '[' -z /usr/local/spark ']'
++ '[' -z '' ']'
++ export SPARK_ENV_LOADED=1
++ SPARK_ENV_LOADED=1
++ export SPARK_CONF_DIR=/usr/local/spark/conf
++ SPARK_CONF_DIR=/usr/local/spark/conf
++ '[' -f /usr/local/spark/conf/spark-env.sh ']'
++ set -a
++ . /usr/local/spark/conf/spark-env.sh
+++ JAVA_HOME=/usr/local/jdk
+++ HADOOP_CONF_DIR=/usr/local/spark/hadoop/etc/hadoop
+++ SPARK_LOCAL_IP=s101
++ set +a
++ '[' -z '' ']'
++ ASSEMBLY_DIR2=/usr/local/spark/assembly/target/scala-2.11
++ ASSEMBLY_DIR1=/usr/local/spark/assembly/target/scala-2.12
++ [[ -d /usr/local/spark/assembly/target/scala-2.11 ]]
++ '[' -d /usr/local/spark/assembly/target/scala-2.11 ']'
++ export SPARK_SCALA_VERSION=2.12
++ SPARK_SCALA_VERSION=2.12


+ '[' '' = '' ']'
+ SPARK_MASTER_PORT=7077


+ '[' '' = '' ']'
+ case `uname` in
++ uname
++ hostname -f
+ SPARK_MASTER_HOST=s101


+ '[' '' = '' ']'
+ SPARK_MASTER_WEBUI_PORT=8080
+ /usr/local/spark/sbin/spark-daemon.sh start org.apache.spark.deploy.master.Master 1 --host s101 --port 7077 --webui-port 8080
starting org.apache.spark.deploy.master.Master, logging to /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out
```

#### **2. start-master.sh再调用spark-daemon.sh**

> sh -x /usr/local/spark/sbin/spark-daemon.sh start org.apache.spark.deploy.master.Master 1 --host s101 --port 7077 --webui-port 8080

------

#### 3. 执行spark-daemon.sh

> sh -x /usr/local/spark/sbin/spark-daemon.sh start org.apache.spark.deploy.master.Master 1 --host s101 --port 7077 --webui-port 8080

```
+ usage='Usage: spark-daemon.sh [--config <conf-dir>] (start|stop|submit|status) <spark-command> <spark-instance-number> <args...>'
+ '[' 9 -le 1 ']'
+ '[' -z /usr/local/spark ']'
+ . /usr/local/spark/sbin/spark-config.sh
++ '[' -z /usr/local/spark ']'
++ export SPARK_CONF_DIR=/usr/local/spark/conf
++ SPARK_CONF_DIR=/usr/local/spark/conf
++ '[' -z '' ']'
++ export PYTHONPATH=/usr/local/spark/python:
++ PYTHONPATH=/usr/local/spark/python:
++ export PYTHONPATH=/usr/local/spark/python/lib/py4j-0.10.7-src.zip:/usr/local/spark/python:
++ PYTHONPATH=/usr/local/spark/python/lib/py4j-0.10.7-src.zip:/usr/local/spark/python:
++ export PYSPARK_PYTHONPATH_SET=1
++ PYSPARK_PYTHONPATH_SET=1
+ '[' start == --config ']'
# 第一个参数是start
+ option=start
+ shift
+ command=org.apache.spark.deploy.master.Master
+ shift
+ instance=1
+ shift
+ . /usr/local/spark/bin/load-spark-env.sh
++ '[' -z /usr/local/spark ']'
++ '[' -z '' ']'
++ export SPARK_ENV_LOADED=1
++ SPARK_ENV_LOADED=1
++ export SPARK_CONF_DIR=/usr/local/spark/conf
++ SPARK_CONF_DIR=/usr/local/spark/conf
++ '[' -f /usr/local/spark/conf/spark-env.sh ']'
++ set -a
++ . /usr/local/spark/conf/spark-env.sh
+++ JAVA_HOME=/usr/local/jdk
+++ HADOOP_CONF_DIR=/usr/local/spark/hadoop/etc/hadoop
+++ SPARK_LOCAL_IP=s101
++ set +a
++ '[' -z '' ']'
++ ASSEMBLY_DIR2=/usr/local/spark/assembly/target/scala-2.11
++ ASSEMBLY_DIR1=/usr/local/spark/assembly/target/scala-2.12
++ [[ -d /usr/local/spark/assembly/target/scala-2.11 ]]
++ '[' -d /usr/local/spark/assembly/target/scala-2.11 ']'
++ export SPARK_SCALA_VERSION=2.12
++ SPARK_SCALA_VERSION=2.12
+ '[' '' = '' ']'
+ export SPARK_IDENT_STRING=hadoop
+ SPARK_IDENT_STRING=hadoop
+ export SPARK_PRINT_LAUNCH_COMMAND=1
+ SPARK_PRINT_LAUNCH_COMMAND=1
+ '[' '' = '' ']'
+ export SPARK_LOG_DIR=/usr/local/spark/logs
+ SPARK_LOG_DIR=/usr/local/spark/logs
+ mkdir -p /usr/local/spark/logs
+ touch /usr/local/spark/logs/.spark_test
+ TEST_LOG_DIR=0
+ '[' 0 = 0 ']'
+ rm -f /usr/local/spark/logs/.spark_test
+ '[' '' = '' ']'
+ SPARK_PID_DIR=/tmp
+ log=/usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out
+ pid=/tmp/spark-hadoop-org.apache.spark.deploy.master.Master-1.pid
+ '[' '' = '' ']'
+ export SPARK_NICENESS=0
+ SPARK_NICENESS=0


# 使用option参数进行匹配，调用方法，传入不同的参数
+ case $option in
# 调用run_command方法，传入class参数以及上个脚本传入的参数
+ run_command class --host s101 --port 7077 --webui-port 8080

# run_command方法内容
+ mode=class
+ shift
+ mkdir -p /tmp
+ '[' -f /tmp/spark-hadoop-org.apache.spark.deploy.master.Master-1.pid ']'
+ '[' '' '!=' '' ']'
+ spark_rotate_log /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out
+ log=/usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out
+ num=5
+ '[' -n '' ']'
+ '[' -f /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out ']'
+ '[' 5 -gt 1 ']'
++ expr 5 - 1
+ prev=4
+ '[' -f /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.4 ']'
+ mv /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.4 /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.5
+ num=4
+ '[' 4 -gt 1 ']'
++ expr 4 - 1
+ prev=3
+ '[' -f /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.3 ']'
+ mv /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.3 /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.4
+ num=3
+ '[' 3 -gt 1 ']'
++ expr 3 - 1
+ prev=2
+ '[' -f /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.2 ']'
+ mv /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.2 /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.3
+ num=2
+ '[' 2 -gt 1 ']'
++ expr 2 - 1
+ prev=1
+ '[' -f /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.1 ']'
+ mv /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.1 /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.2
+ num=1
+ '[' 1 -gt 1 ']'
+ mv /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out.1
+ echo 'starting org.apache.spark.deploy.master.Master, logging to /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out'
starting org.apache.spark.deploy.master.Master, logging to /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-s101.out

# 使用mode值进行匹配
+ case "$mode" in

# 调用execute_command方法
+ execute_command nice -n 0 /usr/local/spark/bin/spark-class org.apache.spark.deploy.master.Master --host s101 --port 7077 --webui-port 8080

#execute_command方法内容
+ '[' -z ']'
+ newpid=3150
+ echo 3150
+ for i in '{1..10}'
++ ps -p 3150 -o comm=

# 调用spark-class脚本
+ nohup -- nice -n 0 /usr/local/spark/bin/spark-class org.apache.spark.deploy.master.Master --host s101 --port 7077 --webui-port 8080
+ [[ bash =~ java ]]
+ sleep 0.5
+ for i in '{1..10}'
++ ps -p 3150 -o comm=
+ [[ java =~ java ]]
+ break
+ sleep 2
++ ps -p 3150 -o comm=
+ [[ ! java =~ java ]]
```

#### 4. spark-daemon.sh再调用spark-class

> sh -x /usr/local/spark/bin/spark-class org.apache.spark.deploy.master.Master --host s101 --port 7077 --webui-port 8080



#### 5. 执行spark-class

> sh -x /usr/local/spark/bin/spark-class org.apache.spark.deploy.master.Master --host s101 --port 7077 --webui-port 8080

```
+ '[' -z /usr/local/spark ']'
+ . /usr/local/spark/bin/load-spark-env.sh
++ '[' -z /usr/local/spark ']'
++ '[' -z '' ']'
++ export SPARK_ENV_LOADED=1
++ SPARK_ENV_LOADED=1
++ export SPARK_CONF_DIR=/usr/local/spark/conf
++ SPARK_CONF_DIR=/usr/local/spark/conf
++ '[' -f /usr/local/spark/conf/spark-env.sh ']'
++ set -a
++ . /usr/local/spark/conf/spark-env.sh
+++ JAVA_HOME=/usr/local/jdk
+++ HADOOP_CONF_DIR=/usr/local/spark/hadoop/etc/hadoop
+++ SPARK_LOCAL_IP=s101
++ set +a
++ '[' -z '' ']'
++ ASSEMBLY_DIR2=/usr/local/spark/assembly/target/scala-2.11
++ ASSEMBLY_DIR1=/usr/local/spark/assembly/target/scala-2.12
++ [[ -d /usr/local/spark/assembly/target/scala-2.11 ]]
++ '[' -d /usr/local/spark/assembly/target/scala-2.11 ']'
++ export SPARK_SCALA_VERSION=2.12
++ SPARK_SCALA_VERSION=2.12
+ '[' -n /usr/local/jdk ']'
+ RUNNER=/usr/local/jdk/bin/java
+ '[' -d /usr/local/spark/jars ']'
+ SPARK_JARS_DIR=/usr/local/spark/jars
+ '[' '!' -d /usr/local/spark/jars ']'
+ LAUNCH_CLASSPATH='/usr/local/spark/jars/*'
+ '[' -n '' ']'
+ [[ -n '' ]]
+ set +o posix
+ CMD=()
+ IFS=
+ read -d '' -r ARG
++ build_command org.apache.spark.deploy.master.Master --host s101 --port 7077 --webui-port 8080
++ /usr/local/jdk/bin/java -Xmx128m -cp '/usr/local/spark/jars/*' org.apache.spark.launcher.Main org.apache.spark.deploy.master.Master --host s101 --port 7077 --webui-port 8080
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
++ printf '%d\0' 0
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ COUNT=12
+ LAST=11
+ LAUNCHER_EXIT_CODE=0
+ [[ 0 =~ ^[0-9]+$ ]]
+ '[' 0 '!=' 0 ']'
+ CMD=("${CMD[@]:0:$LAST}")
+ exec /usr/local/jdk/bin/java -cp '/usr/local/spark/conf/:/usr/local/spark/jars/*:/usr/local/spark/hadoop/etc/hadoop' -Xmx1g org.apache.spark.deploy.master.Master --host s101 --port 7077 --webui-port 8080
2019-10-19 19:32:23 INFO  Master:2612 - Started daemon with process name: 3462@s101
2019-10-19 19:32:23 INFO  SignalUtils:54 - Registered signal handler for TERM
2019-10-19 19:32:23 INFO  SignalUtils:54 - Registered signal handler for HUP
2019-10-19 19:32:23 INFO  SignalUtils:54 - Registered signal handler for INT
2019-10-19 19:32:24 WARN  NativeCodeLoader:62 - Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
2019-10-19 19:32:24 INFO  SecurityManager:54 - Changing view acls to: hadoop
2019-10-19 19:32:24 INFO  SecurityManager:54 - Changing modify acls to: hadoop
2019-10-19 19:32:24 INFO  SecurityManager:54 - Changing view acls groups to: 
2019-10-19 19:32:24 INFO  SecurityManager:54 - Changing modify acls groups to: 
2019-10-19 19:32:24 INFO  SecurityManager:54 - SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(hadoop); groups with view permissions: Set(); users  with modify permissions: Set(hadoop); groups with modify permissions: Set()
2019-10-19 19:32:25 INFO  Utils:54 - Successfully started service 'sparkMaster' on port 7077.
2019-10-19 19:32:25 INFO  Master:54 - Starting Spark master at spark://s101:7077
2019-10-19 19:32:25 INFO  Master:54 - Running Spark version 2.3.3
2019-10-19 19:32:25 INFO  log:192 - Logging initialized @2751ms
2019-10-19 19:32:25 INFO  Server:351 - jetty-9.3.z-SNAPSHOT, build timestamp: unknown, git hash: unknown
2019-10-19 19:32:25 INFO  Server:419 - Started @2854ms
2019-10-19 19:32:25 INFO  AbstractConnector:278 - Started ServerConnector@6697ccfc{HTTP/1.1,[http/1.1]}{s101:8080}
2019-10-19 19:32:25 INFO  Utils:54 - Successfully started service 'MasterUI' on port 8080.
2019-10-19 19:32:25 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@4ca5f5d4{/app,null,AVAILABLE,@Spark}
2019-10-19 19:32:25 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@612dad9d{/app/json,null,AVAILABLE,@Spark}
2019-10-19 19:32:25 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@2d33ea4b{/,null,AVAILABLE,@Spark}
2019-10-19 19:32:25 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@571bbd69{/json,null,AVAILABLE,@Spark}
2019-10-19 19:32:25 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@5bd6b352{/static,null,AVAILABLE,@Spark}
2019-10-19 19:32:25 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@27e45fee{/app/kill,null,AVAILABLE,@Spark}
2019-10-19 19:32:25 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@702f3bde{/driver/kill,null,AVAILABLE,@Spark}
2019-10-19 19:32:25 INFO  MasterWebUI:54 - Bound MasterWebUI to s101, and started at http://s101:8080
2019-10-19 19:32:25 INFO  Server:351 - jetty-9.3.z-SNAPSHOT, build timestamp: unknown, git hash: unknown
2019-10-19 19:32:25 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@58a371ed{/,null,AVAILABLE}
2019-10-19 19:32:25 INFO  AbstractConnector:278 - Started ServerConnector@6753d65d{HTTP/1.1,[http/1.1]}{s101:6066}
2019-10-19 19:32:25 INFO  Server:419 - Started @3096ms
2019-10-19 19:32:25 INFO  Utils:54 - Successfully started service on port 6066.
2019-10-19 19:32:25 INFO  StandaloneRestServer:54 - Started REST server for submitting applications on port 6066
2019-10-19 19:32:25 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@68302393{/metrics/master/json,null,AVAILABLE,@Spark}
2019-10-19 19:32:25 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@6882ec3{/metrics/applications/json,null,AVAILABLE,@Spark}
2019-10-19 19:32:26 INFO  Master:54 - I have been elected leader! New state: ALIVE
```

#### 6. 执行org.apache.spark.deploy.master.Master中的内容

```
/usr/local/jdk/bin/java -cp '/usr/local/spark/conf/:/usr/local/spark/jars/*:/usr/local/spark/hadoop/etc/hadoop' -Xmx1g org.apache.spark.deploy.master.Master --host s101 --port 7077 --webui-port 8080
```

#### 7. Master执行内容

1. 设置处理未被捕获的异常处理器

2. 解析参数，并为sparkConf设置默认参数(默认的参数来自于$SPARK_HOME/conf/spark-defaults.con文件)

3. 向message-dispacture注册Master(RpcEndpoint的一个实例)

   1. 向master end-point发送消息，并在给定的超时时间(默认两分钟)内获取结果，否则抛出异常。返回值为Master RPCEnv、ui端口、rest-server端口

4. master是RpcEndpoint的一个实例

   1. onstart

      1. 根据给定的ui端口启动一个MasterWebUI(可根据**spark.ui.reverseProxy**设置ui代理)
      2. 启动一个后台线程(**newDaemonSingleThreadScheduledExecutor**)，默认每分钟对worker进行检测
      3. spark-rest任务提交端口设置：**spark.master.rest.port**： **6066(默认)**，仅在**standalone**或者**cluster**模式有效
      4. 监控系统的注册与启动
      5. 根据recovery_mode创建持久化引擎和leader

   2. receive

      1. 如果接收到worker的注册信息，将收到的信息组装为**WorkerInfo**，并进行注册

         1. 先删除与当前**worker**的**host**、**port**相等的并且状态为**dead**的已注册的worker

            > ```
            > 从HashSet[WorkerInfo]删除
            > ```

         2. 判断该rpc地址是否已经注册，如果已注册，并且该worker的state为**unknown**，则将之前的workerInfo移除

            > ```
            > addressToWorker = new HashMap[RpcAddress, WorkerInfo]
            > removeWorker(oldWorker, "Worker replaced by a new worker with same address")
            > ```

         3. 进行注册

            ```
            workers = new HashSet[WorkerInfo] += worker
            idToWorker(worker.id) = new HashMap[String, WorkerInfo] = worker
            addressToWorker(workerAddress) = new HashMap[RpcAddress, WorkerInfo] = worker
            ```

      2. 将worker加入持久化引擎

      3. 向worker发送数据

         > ```
         > RegisteredWorker(self, masterWebUiUrl, masterAddress)
         > ```

      4. 调用**schedule()**方法，该方法会在每次有新的application加入或者可用的资源刷新的时候调用，主要作用是调整可用的资源

### 2. 调试start-slaves.sh

#### 1. 执行start-slaves.sh文件

> sh -x start-slaves.sh

```
+ '[' -z /usr/local/spark ']'


+ . /usr/local/spark/sbin/spark-config.sh
++ '[' -z /usr/local/spark ']'
++ export SPARK_CONF_DIR=/usr/local/spark/conf
++ SPARK_CONF_DIR=/usr/local/spark/conf
++ '[' -z '' ']'
++ export PYTHONPATH=/usr/local/spark/python:
++ PYTHONPATH=/usr/local/spark/python:
++ export PYTHONPATH=/usr/local/spark/python/lib/py4j-0.10.7-src.zip:/usr/local/spark/python:
++ PYTHONPATH=/usr/local/spark/python/lib/py4j-0.10.7-src.zip:/usr/local/spark/python:
++ export PYSPARK_PYTHONPATH_SET=1
++ PYSPARK_PYTHONPATH_SET=1


+ . /usr/local/spark/bin/load-spark-env.sh
++ '[' -z /usr/local/spark ']'
++ '[' -z '' ']'
++ export SPARK_ENV_LOADED=1
++ SPARK_ENV_LOADED=1
++ export SPARK_CONF_DIR=/usr/local/spark/conf
++ SPARK_CONF_DIR=/usr/local/spark/conf
++ '[' -f /usr/local/spark/conf/spark-env.sh ']'
++ set -a
++ . /usr/local/spark/conf/spark-env.sh
+++ JAVA_HOME=/usr/local/jdk
+++ HADOOP_CONF_DIR=/usr/local/spark/hadoop/etc/hadoop
+++ SPARK_LOCAL_IP=s101
++ set +a
++ '[' -z '' ']'
++ ASSEMBLY_DIR2=/usr/local/spark/assembly/target/scala-2.11
++ ASSEMBLY_DIR1=/usr/local/spark/assembly/target/scala-2.12
++ [[ -d /usr/local/spark/assembly/target/scala-2.11 ]]
++ '[' -d /usr/local/spark/assembly/target/scala-2.11 ']'
++ export SPARK_SCALA_VERSION=2.12
++ SPARK_SCALA_VERSION=2.12


+ '[' '' = '' ']'
+ SPARK_MASTER_PORT=7077


+ '[' '' = '' ']'
+ case `uname` in
++ uname
++ hostname -f
+ SPARK_MASTER_HOST=s101


+ /usr/local/spark/sbin/slaves.sh cd /usr/local/spark ';' 

# 调用spark-slave.sh文件
/usr/local/spark/sbin/start-slave.sh spark://s101:7077


s101: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out
```

#### 2. start-slaves.sh调用start-slave.sh

> /usr/local/spark/sbin/start-slave.sh spark://s101:7077

#### 3. 执行start-slave.sh

> sh -x /usr/local/spark/sbin/start-slave.sh spark://s101:7077

```
+ '[' -z /usr/local/spark ']'


+ CLASS=org.apache.spark.deploy.worker.Worker


+ [[ 1 -lt 1 ]]
+ [[ spark://s101:7077 = *--help ]]
+ [[ spark://s101:7077 = *-h ]]
+ . /usr/local/spark/sbin/spark-config.sh
++ '[' -z /usr/local/spark ']'
++ export SPARK_CONF_DIR=/usr/local/spark/conf
++ SPARK_CONF_DIR=/usr/local/spark/conf
++ '[' -z '' ']'
++ export PYTHONPATH=/usr/local/spark/python:
++ PYTHONPATH=/usr/local/spark/python:
++ export PYTHONPATH=/usr/local/spark/python/lib/py4j-0.10.7-src.zip:/usr/local/spark/python:
++ PYTHONPATH=/usr/local/spark/python/lib/py4j-0.10.7-src.zip:/usr/local/spark/python:
++ export PYSPARK_PYTHONPATH_SET=1
++ PYSPARK_PYTHONPATH_SET=1
+ . /usr/local/spark/bin/load-spark-env.sh
++ '[' -z /usr/local/spark ']'
++ '[' -z '' ']'
++ export SPARK_ENV_LOADED=1
++ SPARK_ENV_LOADED=1
++ export SPARK_CONF_DIR=/usr/local/spark/conf
++ SPARK_CONF_DIR=/usr/local/spark/conf
++ '[' -f /usr/local/spark/conf/spark-env.sh ']'
++ set -a
++ . /usr/local/spark/conf/spark-env.sh
+++ JAVA_HOME=/usr/local/jdk
+++ HADOOP_CONF_DIR=/usr/local/spark/hadoop/etc/hadoop
+++ SPARK_LOCAL_IP=s101
++ set +a
++ '[' -z '' ']'
++ ASSEMBLY_DIR2=/usr/local/spark/assembly/target/scala-2.11
++ ASSEMBLY_DIR1=/usr/local/spark/assembly/target/scala-2.12
++ [[ -d /usr/local/spark/assembly/target/scala-2.11 ]]
++ '[' -d /usr/local/spark/assembly/target/scala-2.11 ']'
++ export SPARK_SCALA_VERSION=2.12
++ SPARK_SCALA_VERSION=2.12


+ MASTER=spark://s101:7077
+ shift


+ '[' '' = '' ']'
+ SPARK_WORKER_WEBUI_PORT=8081


+ '[' '' = '' ']'
+ start_instance 1
+ WORKER_NUM=1
+ shift
+ '[' '' = '' ']'
+ PORT_FLAG=
+ PORT_NUM=
+ WEBUI_PORT=8081

+ /usr/local/spark/sbin/spark-daemon.sh start org.apache.spark.deploy.worker.Worker 1 --webui-port 8081 spark://s101:7077
starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out
```

#### 4. start-slave.sh调用spark-daemon.sh

>  /usr/local/spark/sbin/spark-daemon.sh start org.apache.spark.deploy.worker.Worker 1 --webui-port 8081 spark://s101:7077

#### 5. 执行spark-daemon.sh

> sh -x /usr/local/spark/sbin/spark-daemon.sh start org.apache.spark.deploy.worker.Worker 1 --webui-port 8081 spark://s101:7077

```
+ usage='Usage: spark-daemon.sh [--config <conf-dir>] (start|stop|submit|status) <spark-command> <spark-instance-number> <args...>'
+ '[' 6 -le 1 ']'
+ '[' -z /usr/local/spark ']'
+ . /usr/local/spark/sbin/spark-config.sh
++ '[' -z /usr/local/spark ']'
++ export SPARK_CONF_DIR=/usr/local/spark/conf
++ SPARK_CONF_DIR=/usr/local/spark/conf
++ '[' -z '' ']'
++ export PYTHONPATH=/usr/local/spark/python:
++ PYTHONPATH=/usr/local/spark/python:
++ export PYTHONPATH=/usr/local/spark/python/lib/py4j-0.10.7-src.zip:/usr/local/spark/python:
++ PYTHONPATH=/usr/local/spark/python/lib/py4j-0.10.7-src.zip:/usr/local/spark/python:
++ export PYSPARK_PYTHONPATH_SET=1
++ PYSPARK_PYTHONPATH_SET=1
+ '[' start == --config ']'
+ option=start
+ shift
+ command=org.apache.spark.deploy.worker.Worker
+ shift
+ instance=1
+ shift
+ . /usr/local/spark/bin/load-spark-env.sh
++ '[' -z /usr/local/spark ']'
++ '[' -z '' ']'
++ export SPARK_ENV_LOADED=1
++ SPARK_ENV_LOADED=1
++ export SPARK_CONF_DIR=/usr/local/spark/conf
++ SPARK_CONF_DIR=/usr/local/spark/conf
++ '[' -f /usr/local/spark/conf/spark-env.sh ']'
++ set -a
++ . /usr/local/spark/conf/spark-env.sh
+++ JAVA_HOME=/usr/local/jdk
+++ HADOOP_CONF_DIR=/usr/local/spark/hadoop/etc/hadoop
+++ SPARK_LOCAL_IP=s101
++ set +a
++ '[' -z '' ']'
++ ASSEMBLY_DIR2=/usr/local/spark/assembly/target/scala-2.11
++ ASSEMBLY_DIR1=/usr/local/spark/assembly/target/scala-2.12
++ [[ -d /usr/local/spark/assembly/target/scala-2.11 ]]
++ '[' -d /usr/local/spark/assembly/target/scala-2.11 ']'
++ export SPARK_SCALA_VERSION=2.12
++ SPARK_SCALA_VERSION=2.12
+ '[' '' = '' ']'
+ export SPARK_IDENT_STRING=hadoop
+ SPARK_IDENT_STRING=hadoop
+ export SPARK_PRINT_LAUNCH_COMMAND=1
+ SPARK_PRINT_LAUNCH_COMMAND=1
+ '[' '' = '' ']'
+ export SPARK_LOG_DIR=/usr/local/spark/logs
+ SPARK_LOG_DIR=/usr/local/spark/logs
+ mkdir -p /usr/local/spark/logs
+ touch /usr/local/spark/logs/.spark_test
+ TEST_LOG_DIR=0
+ '[' 0 = 0 ']'
+ rm -f /usr/local/spark/logs/.spark_test
+ '[' '' = '' ']'
+ SPARK_PID_DIR=/tmp
+ log=/usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out
+ pid=/tmp/spark-hadoop-org.apache.spark.deploy.worker.Worker-1.pid
+ '[' '' = '' ']'
+ export SPARK_NICENESS=0
+ SPARK_NICENESS=0
+ case $option in
+ run_command class --webui-port 8081 spark://s101:7077
+ mode=class
+ shift
+ mkdir -p /tmp
+ '[' -f /tmp/spark-hadoop-org.apache.spark.deploy.worker.Worker-1.pid ']'
+ '[' '' '!=' '' ']'
+ spark_rotate_log /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out
+ log=/usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out
+ num=5
+ '[' -n '' ']'
+ '[' -f /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out ']'
+ '[' 5 -gt 1 ']'
++ expr 5 - 1
+ prev=4
+ '[' -f /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.4 ']'
+ mv /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.4 /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.5
+ num=4
+ '[' 4 -gt 1 ']'
++ expr 4 - 1
+ prev=3
+ '[' -f /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.3 ']'
+ mv /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.3 /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.4
+ num=3
+ '[' 3 -gt 1 ']'
++ expr 3 - 1
+ prev=2
+ '[' -f /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.2 ']'
+ mv /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.2 /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.3
+ num=2
+ '[' 2 -gt 1 ']'
++ expr 2 - 1
+ prev=1
+ '[' -f /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.1 ']'
+ mv /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.1 /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.2
+ num=1
+ '[' 1 -gt 1 ']'
+ mv /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out.1
+ echo 'starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out'
starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-s101.out
+ case "$mode" in



+ execute_command nice -n 0 /usr/local/spark/bin/spark-class org.apache.spark.deploy.worker.Worker --webui-port 8081 spark://s101:7077
+ '[' -z ']'
+ newpid=15002
+ echo 15002
+ for i in '{1..10}'
++ ps -p 15002 -o comm=
+ nohup -- nice -n 0 /usr/local/spark/bin/spark-class org.apache.spark.deploy.worker.Worker --webui-port 8081 spark://s101:7077
+ [[ bash =~ java ]]
+ sleep 0.5
+ for i in '{1..10}'
++ ps -p 15002 -o comm=
+ [[ java =~ java ]]
+ break
+ sleep 2
++ ps -p 15002 -o comm=
+ [[ ! java =~ java ]]
```

#### 6. spark-daemon.sh调用spark-class

> /usr/local/spark/bin/spark-class org.apache.spark.deploy.worker.Worker --webui-port 8081 spark://s101:7077

#### 7. 执行spark-class

> sh -x /usr/local/spark/bin/spark-class org.apache.spark.deploy.worker.Worker --webui-port 8081 spark://s101:7077

```
+ '[' -z /usr/local/spark ']'
+ . /usr/local/spark/bin/load-spark-env.sh
++ '[' -z /usr/local/spark ']'
++ '[' -z '' ']'
++ export SPARK_ENV_LOADED=1
++ SPARK_ENV_LOADED=1
++ export SPARK_CONF_DIR=/usr/local/spark/conf
++ SPARK_CONF_DIR=/usr/local/spark/conf
++ '[' -f /usr/local/spark/conf/spark-env.sh ']'
++ set -a
++ . /usr/local/spark/conf/spark-env.sh
+++ JAVA_HOME=/usr/local/jdk
+++ HADOOP_CONF_DIR=/usr/local/spark/hadoop/etc/hadoop
+++ SPARK_LOCAL_IP=s101
++ set +a
++ '[' -z '' ']'
++ ASSEMBLY_DIR2=/usr/local/spark/assembly/target/scala-2.11
++ ASSEMBLY_DIR1=/usr/local/spark/assembly/target/scala-2.12
++ [[ -d /usr/local/spark/assembly/target/scala-2.11 ]]
++ '[' -d /usr/local/spark/assembly/target/scala-2.11 ']'
++ export SPARK_SCALA_VERSION=2.12
++ SPARK_SCALA_VERSION=2.12
+ '[' -n /usr/local/jdk ']'
+ RUNNER=/usr/local/jdk/bin/java
+ '[' -d /usr/local/spark/jars ']'
+ SPARK_JARS_DIR=/usr/local/spark/jars
+ '[' '!' -d /usr/local/spark/jars ']'
+ LAUNCH_CLASSPATH='/usr/local/spark/jars/*'
+ '[' -n '' ']'
+ [[ -n '' ]]
+ set +o posix
+ CMD=()
+ IFS=
+ read -d '' -r ARG
++ build_command org.apache.spark.deploy.worker.Worker --webui-port 8081 spark://s101:7077
++ /usr/local/jdk/bin/java -Xmx128m -cp '/usr/local/spark/jars/*' org.apache.spark.launcher.Main org.apache.spark.deploy.worker.Worker --webui-port 8081 spark://s101:7077
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
++ printf '%d\0' 0
+ CMD+=("$ARG")
+ IFS=
+ read -d '' -r ARG
+ COUNT=9
+ LAST=8
+ LAUNCHER_EXIT_CODE=0
+ [[ 0 =~ ^[0-9]+$ ]]
+ '[' 0 '!=' 0 ']'
+ CMD=("${CMD[@]:0:$LAST}")
+ exec /usr/local/jdk/bin/java -cp '/usr/local/spark/conf/:/usr/local/spark/jars/*:/usr/local/spark/hadoop/etc/hadoop' -Xmx1g org.apache.spark.deploy.worker.Worker --webui-port 8081 spark://s101:7077
2019-10-20 18:39:22 INFO  Worker:2612 - Started daemon with process name: 15110@s101
2019-10-20 18:39:22 INFO  SignalUtils:54 - Registered signal handler for TERM
2019-10-20 18:39:22 INFO  SignalUtils:54 - Registered signal handler for HUP
2019-10-20 18:39:22 INFO  SignalUtils:54 - Registered signal handler for INT
2019-10-20 18:39:23 WARN  NativeCodeLoader:62 - Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
2019-10-20 18:39:23 INFO  SecurityManager:54 - Changing view acls to: hadoop
2019-10-20 18:39:23 INFO  SecurityManager:54 - Changing modify acls to: hadoop
2019-10-20 18:39:23 INFO  SecurityManager:54 - Changing view acls groups to: 
2019-10-20 18:39:23 INFO  SecurityManager:54 - Changing modify acls groups to: 
2019-10-20 18:39:23 INFO  SecurityManager:54 - SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(hadoop); groups with view permissions: Set(); users  with modify permissions: Set(hadoop); groups with modify permissions: Set()
2019-10-20 18:39:24 INFO  Utils:54 - Successfully started service 'sparkWorker' on port 34114.
2019-10-20 18:39:24 INFO  Worker:54 - Starting Spark worker 172.31.53.15:34114 with 1 cores, 1024.0 MB RAM
2019-10-20 18:39:24 INFO  Worker:54 - Running Spark version 2.3.3
2019-10-20 18:39:24 INFO  Worker:54 - Spark home: /usr/local/spark
2019-10-20 18:39:24 INFO  log:192 - Logging initialized @3007ms
2019-10-20 18:39:24 INFO  Server:351 - jetty-9.3.z-SNAPSHOT, build timestamp: unknown, git hash: unknown
2019-10-20 18:39:24 INFO  Server:419 - Started @3111ms
2019-10-20 18:39:24 INFO  AbstractConnector:278 - Started ServerConnector@5bdc0045{HTTP/1.1,[http/1.1]}{s101:8081}
2019-10-20 18:39:24 INFO  Utils:54 - Successfully started service 'WorkerUI' on port 8081.
2019-10-20 18:39:24 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@2223748e{/logPage,null,AVAILABLE,@Spark}
2019-10-20 18:39:24 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@91f1149{/logPage/json,null,AVAILABLE,@Spark}
2019-10-20 18:39:24 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@1dc6c31d{/,null,AVAILABLE,@Spark}
2019-10-20 18:39:24 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@41eb54b{/json,null,AVAILABLE,@Spark}
2019-10-20 18:39:24 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@75fa79d8{/static,null,AVAILABLE,@Spark}
2019-10-20 18:39:24 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@363e8ecc{/log,null,AVAILABLE,@Spark}
2019-10-20 18:39:24 INFO  WorkerWebUI:54 - Bound WorkerWebUI to s101, and started at http://s101:8081
2019-10-20 18:39:24 INFO  Worker:54 - Connecting to master s101:7077...
2019-10-20 18:39:25 INFO  ContextHandler:781 - Started o.s.j.s.ServletContextHandler@137ca876{/metrics/json,null,AVAILABLE,@Spark}
2019-10-20 18:39:25 INFO  TransportClientFactory:267 - Successfully created connection to s101/172.31.53.15:7077 after 70 ms (0 ms spent in bootstraps)
2019-10-20 18:39:25 INFO  Worker:54 - Successfully registered with master spark://s101:7077
```

#### 8. 执行org.apache.spark.deploy.worker.Worker中的内容

```
/usr/local/jdk/bin/java -cp '/usr/local/spark/conf/:/usr/local/spark/jars/*:/usr/local/spark/hadoop/etc/hadoop' -Xmx1g org.apache.spark.deploy.worker.Worker --webui-port 8081 spark://s101:7077
```

#### 9. Worker执行内容

1. 设置处理未被捕获的异常处理器

2. 解析参数，并未sparkConf设置默认参数(默认的参数来自于$SPARK_HOME/conf/spark-defaults.con文件)

3. 向message-dispacture注册Worker(RpcEndpoint的一个实例)

4. 判断**spark.shuffle.service.enabled**与**SPARK_WORKER_INSTANCES**是否冲突

5. 因为Worker是RpcEndpoint的一个实例

   1. onstart

      1. 启动externalShuffleService

      2. 创建WorkerWebUI

      3. 向master进行注册

         1. 通过从脚本传入的master参数创建newDaemonCachedThreadPool，该线程池的大小与master的数量相同

         2. 线程池中的每个线程通过master的地址与end point name获取masterEndPoint

         3. 向master发送注册信息

            ```
            消息内容：RegisterWorker( workerId, host, port, self, cores, memory, workerWebUiUrl, masterEndpoint.address)
            ```

   2. receive

      1. 如果接受到的是**RegisterWorkerResponse**，则会进行接收到注册响应的逻辑处理

         1. 更改master的相关属性

            ```
            activeMasterUrl: String = masterRef.address.toSparkURL
            activeMasterWebUiUrl : String = uiUrl
            masterAddressToConnect: Option[RpcAddress] = Some(masterAddress)
            master: Option[RpcEndpointRef] = Some(masterRef)
            connected = true
            ```

         2. 通过newDaemonSingleThreadScheduledExecutor守护线程池每**15**秒发送一次心跳

         3. 如果开启了**spark.worker.cleanup.enabled**，则每**30**分钟进行一次工作空间的清理

         4. 向master发送最新的worker状态

            > ```
            > WorkerLatestState(workerId, execs.toList, drivers.keys.toSeq)
            > ```

## 3. 调试spark-submit.sh

## 4. 远程调试/调试源码

1. spark端配置

   ```
   #调试Master，在master节点的spark-env.sh中添加SPARK_MASTER_OPTS变量
   export SPARK_MASTER_OPTS="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=10000"
   #启动Master，程序会卡住，等待调试端进行连接
   sbin/start-master.sh
   #本地启动remote
   
   #调试Worker，在worker节点的spark-env.sh中添加SPARK_WORKER_OPTS变量
   export SPARK_WORKER_OPTS="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=10001"
   #启动Worker
   sbin/start-slave.sh 1 spark://hadoop1:7077
   
   #调试spark-submit + app
   bin/spark-submit --class cn.itcast.spark.WordCount --master spark://hadoop1:7077 --driver-java-options "-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=10002" /root/wc.jar hdfs://mycluster/wordcount/input/2.txt hdfs://mycluster/out2 
   
   #调试spark-submit + app + executor
   bin/spark-submit --class cn.itcast.spark.WordCount --master spark://hadoop1:7077 --conf "spark.executor.extraJavaOptions=-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=10003" --driver-java-options "-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=10002" /root/wc.jar hdfs://mycluster/wordcount/input/2.txt hdfs://mycluster/out2 
   ```

2. idea配置

   > 新增一个remote配置

3. 执行提交脚本

   ```
   bin/spark-submit --class cn.itcast.spark.WordCount --master spark://hadoop1:7077 --driver-java-options "-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=10002" /root/wc.jar hdfs://mycluster/wordcount/input/2.txt hdfs://mycluster/out2 
   ```

   