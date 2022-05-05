# flume docker 部署

#####1.查看docker hub中的flume

> [root@localhost ~]# docker search flume
> NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
> probablyfine/flume                Self-contained Docker image containing Java …   30                                      [OK]
> bigcontainer/flume                Bigcontainer Flume docker image                 7                                       [OK]
> bde2020/flume                     BDE Docker Image for Apache Flume               4                                       [OK]
> anchorfree/flume                  Flume agent                                     4
> vungle/flume                                                                      3
> mpidlisnyi/flume                  Apache Flume-NG image with S3 sink supporting   2                                       [OK]
> mengluo668/flume                  Docker image for Apache Flume                   2                                       [OK]
> szaharici/flume-es6               Flume with Rest ES sink compatible with rece…   1                                       [OK]
> 0079123/flume                     Docker image for Apache Flume                   1
> eskabetxe/flume                   Docker for Apache Flume                         0                                       [OK]
> avastsoftware/flume-hdfs          Apache Flume with HDFS support                  0                                       [OK]
> zhijunwoo/flume_exporter          Prometheus exporter for flume                   0
> aminoapps/flume-docker                                                            0
> telefonica/flume                  Docker image containing Flume.                  0
> kealanmaas/flume_kafka_hdfs       Simple Flume Agent for testing                  0
> regexpress/flume                  Flume test image.                               0                                       [OK]
> cogniteev/flume                   Provides Apache Flume images that helps you …   0                                       [OK]
> flumecloudservices/file-storage   Golang File Storage server via HTTP (by Flum…   0
> flumecloudservices/cache          Golang key/value Database via HTTP (by Flume…   0
> elek/flume                        Apache Flume with advanced configuration        0                                       [OK]
> alonsodomin/flume                 Docker image for Apache Flume                   0                                       [OK]
> flumecloudservices/database       Golang Database via HTTP using LevelDB and J…   0
> klustera/flume                    Apache Flume para ingesta de archivos           0                                       [OK]
> lokkju/flume-hdfs                 Docker for Apache Flume with HDFS support       0                                       [OK]
> smartislav/flume                                                                  0                                       [OK]

##### 2.制作Dockerfile

> mkdir -p  /home/docker_build_apps/flume
>
> vim Dockerfile

```shell
FROM probablyfine/flume
RUN  apt-get update
RUN  apt-get install -y apt-utils
RUN  apt-get install -y inetutils-telnet
RUN  apt-get install -y vim
RUN  echo alias ll='ls $LS_OPTIONS -l' >> ~/.bashrc
RUN  apt-get install -y lsof
#ADD flume-example.conf /var/tmp/flume-example.conf
EXPOSE 44444
EXPOSE 44445
#ENTRYPOINT [ "flume-ng", "agent",  "-c", "/opt/flume/conf", "-f", "/var/tmp/flume-example.conf", "-n", "docker","-Dflume.root.logger=INFO,console" ]
```

制作镜像

> [root@localhost flume]# docker build -t  youzhiqiang/flume:0.0.2  .

查看镜像

> [root@localhost flume]# docker images
> REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
> **youzhiqiang/flume      0.0.2               b2464d65f878        4 days ago          563MB**
> wurstmeister/kafka     latest              2dd91ce2efe1        4 months ago        508MB
> zookeeper              latest              36c607e7b14d        4 months ago        278MB
> mysql                  5.7                 c20987f18b13        4 months ago        448MB
> bde2020/spark-master   3.2.0-hadoop3.2     3d161dc0595b        4 months ago        545MB
> bde2020/spark-worker   3.2.0-hadoop3.2     cb570e0e37b4        4 months ago        545MB
> dushixiang/kafka-map   latest              b711e1fb5c60        5 months ago        295MB
> hello-world            latest              feb5d9fea6a5        7 months ago        13.3kB
> probablyfine/flume     latest              b7be214a488a        3 years ago         474MB

> [root@localhost ~]# docker pull probablyfine/flume:2.0.0
> 2.0.0: Pulling from probablyfine/flume
> f49cf87b52c1: Pull complete
> 5ec17d7da55e: Pull complete
> e59e25d245e8: Pull complete
> Digest: sha256:751e2e61862fd7808cf8f642c3884f09da9bdb4464f6c38a452c472dcdbc5cc8
> Status: Downloaded newer image for probablyfine/flume:2.0.0

##### 3.创建本地目录供持久化

> [root@localhost docker-flume]# mkdir -p flume1/conf
> [root@localhost docker-flume]# mkdir -p flume1/logs
> [root@localhost docker-flume]# mkdir -p flume1/flume-log

##### 4.启动flume

```sh
docker run \
--name flume1 \
--restart always \
--network zkkf-net \
--ip 172.19.0.12 \
--p 14444:4444 \
-p 4141:4141 \ 
-v  /home/docker_apps/docker-flume/flume1/conf:/opt/flume-config/flume.conf  \
-v  /home/docker_apps/docker-flume/flume1/flume-log:/var/tmp/flume_log \
-v  /home/docker_apps/docker-flume/flume1/logs:/opt/flume/logs \
-e FLUME_AGENT_NAME="agent" \
-d youzhiqiang/flume:0.0.2
##--p 4141:4141将agent的source端口映射虚拟机上
##--p 14444:4444 将flume监控服务的监控端口映射到虚拟机上
```



> [root@localhost docker-flume] docker run \
>
> --name flume1 \
> --restart always \
> --network zkkf-net \
> --ip 172.19.0.12 \
> -p 4141:4141 \
> -p 14444:4444 \
> -v  /home/docker_apps/docker-flume/flume1/conf:/opt/flume-config/flume.conf  \
> -v  /home/docker_apps/docker-flume/flume1/flume-log:/var/tmp/flume_log \
> -v  /home/docker_apps/docker-flume/flume1/logs:/opt/flume/logs \
> -e FLUME_AGENT_NAME="agent" \
> -d youzhiqiang/flume:0.0.2

查看容器：

> [root@localhost flume]# docker ps
> CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS       NAMES
> **03a91ff80946        youzhiqiang/flume:0.0.2   "start-flume"            10 minutes ago      Up 10 minutes       0.0.0.0:4141->4141/tcp, 44444-44445/tcp, 0.0.0.0:14444->4444/tcp   flume1 **
> 32f1cd11b9d4        youzhiqiang/flume:0.0.2   "start-flume"            3 days ago          Up About an hour    0.0.0.0:4545->4545/tcp, 44444-44445/tcp       flume2
> 22c146f3f95d        wurstmeister/kafka        "start-kafka.sh"         13 days ago         Up About an hour    0.0.0.0:9093->9093/tcp       kafka2
> b0ae34bb9e08        wurstmeister/kafka        "start-kafka.sh"         13 days ago         Up About an hour    0.0.0.0:9092->9092/tcp       kafka1
> 0d411f8cfc64        wurstmeister/kafka        "start-kafka.sh"         13 days ago         Up About an hour    0.0.0.0:9094->9094/tcp       kafka3
> e3fb4bf16d66        zookeeper                 "/docker-entrypoint.…"   13 days ago         Up About an hour    2888/tcp, 3888/tcp, 8080/tcp, 0.0.0.0:2182->2181/tcp       zoo2
> 2651a9e09552        zookeeper                 "/docker-entrypoint.…"   13 days ago         Up About an hour    2888/tcp, 3888/tcp, 0.0.0.0:2181->2181/tcp, 8080/tcp       zoo1
> 714f78bb216e        zookeeper                 "/docker-entrypoint.…"   13 days ago         Up About an hour    2888/tcp, 3888/tcp, 8080/tcp, 0.0.0.0:2183->2181/tcp       zoo3

#####5.创建agent conf文件

进入容器：

> [root@localhost flume]# docker exec -it flume1 bash

在/opt/flume-config/flume.conf中创建a1.conf

```shell
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
##docker部署方式，此处的source绑定的ip必须为0.0.0.0，否则外部访问不了
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 4141

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

##### 6.启动a1 agent

bin/flume-ng agent --conf conf --conf-file /opt/flume-config/flume.conf/a1.conf --name a1 -Dflume.root.logger=INFO,console -Dflume.monitoring.type=http -Dflume.monitoring.port=4444

` -Dflume.monitoring.type=http` 表示使用http的方式监控Flume

`Dflume.monitoring.port=4444` 表示监控程序监听4444 端口。

> root@03a91ff80946:/opt/flume# **bin/flume-ng agent --conf conf --conf-file /opt/flume-config/flume.conf/a1.conf --name a1 -Dflume.root.logger=INFO,console -Dflume.monitoring.type=http -Dflume.monitoring.port=4444**
> Info: Including Hive libraries found via () for Hive access
>
> + exec /opt/java/bin/java -Xmx20m -Dflume.root.logger=INFO,console -Dflume.monitoring.type=http -Dflume.monitoring.port=4444 -cp '/opt/flume/conf:/opt/flume/lib/*:/lib/*'-Djava.library.path= org.apache.flume.node.Application --conf-file /opt/flume-config/flume.conf/a1.conf --name a1
>   2022-05-05 11:24:39,341 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.node.PollingPropertiesFileConfigurationProvider.start(PollingPropertiesFileConfigurationProvider.java:62)] Configuration provider starting
>   2022-05-05 11:24:39,347 (conf-file-poller-0) [INFO - org.apache.flume.node.PollingPropertiesFileConfigurationProvider$FileWatcherRunnable.run(PollingPropertiesFileConfigurationProvider.java:134)] Reloading configuration file:/opt/flume-config/flume.conf/a1.conf
>   2022-05-05 11:24:39,358 (conf-file-poller-0) [INFO - org.apache.flume.conf.FlumeConfiguration$AgentConfiguration.addProperty(FlumeConfiguration.java:930)] Added sinks: k1Agent: a1
>   2022-05-05 11:24:39,358 (conf-file-poller-0) [INFO - org.apache.flume.conf.FlumeConfiguration$AgentConfiguration.addProperty(FlumeConfiguration.java:1016)] Processing:k1
>   2022-05-05 11:24:39,358 (conf-file-poller-0) [INFO - org.apache.flume.conf.FlumeConfiguration$AgentConfiguration.addProperty(FlumeConfiguration.java:1016)] Processing:k1
>   2022-05-05 11:24:39,381 (conf-file-poller-0) [INFO - org.apache.flume.conf.FlumeConfiguration.validateConfiguration(FlumeConfiguration.java:140)] Post-validation flume configuration contains configuration for agents: [a1]
>   2022-05-05 11:24:39,382 (conf-file-poller-0) [INFO - org.apache.flume.node.AbstractConfigurationProvider.loadChannels(AbstractConfigurationProvider.java:147)] Creating channels
>   2022-05-05 11:24:39,392 (conf-file-poller-0) [INFO - org.apache.flume.channel.DefaultChannelFactory.create(DefaultChannelFactory.java:42)] Creating instance of channel c1type memory
>   2022-05-05 11:24:39,400 (conf-file-poller-0) [INFO - org.apache.flume.node.AbstractConfigurationProvider.loadChannels(AbstractConfigurationProvider.java:201)] Created channel c1
>   2022-05-05 11:24:39,401 (conf-file-poller-0) [INFO - org.apache.flume.source.DefaultSourceFactory.create(DefaultSourceFactory.java:41)] Creating instance of source r1, type netcat
>   2022-05-05 11:24:39,420 (conf-file-poller-0) [INFO - org.apache.flume.sink.DefaultSinkFactory.create(DefaultSinkFactory.java:42)] Creating instance of sink: k1, type: logger
>   2022-05-05 11:24:39,424 (conf-file-poller-0) [INFO - org.apache.flume.node.AbstractConfigurationProvider.getConfiguration(AbstractConfigurationProvider.java:116)] Channelc1 connected to [r1, k1]
>   2022-05-05 11:24:39,438 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:137)] Starting new configuration:{ sourceRunners:{r1=EventDrivenSourceRunner: { source:org.apache.flume.source.NetcatSource{name:r1,state:IDLE} }} sinkRunners:{k1=SinkRunner: { policy:org.apache.flume.sink.DefaultSinkProcessor@7efedb9f counterGroup:{ name:null counters:{} } }} channels:{c1=org.apache.flume.channel.MemoryChannel{name: c1}} }
>   2022-05-05 11:24:39,453 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:144)] Starting Channel c1
>   2022-05-05 11:24:39,463 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:159)] Waiting for channel: c1 to start. Sleeping for 500 ms
>   2022-05-05 11:24:39,553 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.register(MonitoredCounterGroup.java:119)] Monitored counter group for type: CHANNEL, name: c1: Successfully registered new MBean.
>   2022-05-05 11:24:39,554 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.start(MonitoredCounterGroup.java:95)] Component type: CHANNEL, name: c1 started
>   2022-05-05 11:24:39,968 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:171)] Starting Sink k1
>   2022-05-05 11:24:39,985 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:182)] Starting Source r1
>   2022-05-05 11:24:40,004 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.source.NetcatSource.start(NetcatSource.java:155)] Source starting
>   2022-05-05 11:24:40,069 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.source.NetcatSource.start(NetcatSource.java:166)] Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/**0.0.0.0:4141**]
>   2022-05-05 11:24:40,107 (conf-file-poller-0) [INFO - org.mortbay.log.Slf4jLog.info(Slf4jLog.java:67)] Logging to org.slf4j.impl.Log4jLoggerAdapter(org.mortbay.log) via org.mortbay.log.Slf4jLog
>   2022-05-05 11:24:40,167 (conf-file-poller-0) [INFO - org.mortbay.log.Slf4jLog.info(Slf4jLog.java:67)] jetty-6.1.26
>   2022-05-05 11:24:40,200 (conf-file-poller-0) [INFO - org.mortbay.log.Slf4jLog.info(Slf4jLog.java:67)] Started SelectChannelConnector@**0.0.0.0:4444**

##### 7.测试连接agent

外部连接：

> [root@localhost conf]# telnet 172.19.0.12 4141
> Trying 172.19.0.12...
> Connected to 172.19.0.12.
> Escape character is '^]'.
> hello
> OK
> world
> OK
> 11
> OK
> 22
> OK

容器内部：

> 2022-05-05 11:28:14,083 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 6865 6C 6C 6F 0D                               **hello**. }
> 2022-05-05 11:28:16,484 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 776F 72 6C 64 0D                              **world**. }
> 2022-05-05 11:28:18,379 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 3131 0D                                        **11**. }
> 2022-05-05 11:28:19,136 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 3232 0D                                        **22**. }