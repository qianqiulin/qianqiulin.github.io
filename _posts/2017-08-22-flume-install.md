---
layout:     post
title:      "Flume 安装配置"
subtitle:   " BigData Flume"
date:       2017-08-22 10:20:27
author:     "TyurinTsien"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
    - Flume
---


## Flume 安装配置

+ TyurinTsien
+ 2017年08月22日10:20:27

### 背景
Cloudera 开发的分布式日志收集系统 Flume，是 hadoop 周边组件之一。其可以实时的将分布在不同节点、机器上的日志收集到 hdfs 中。Flume 初始的发行版本目前被统称为 Flume OG（original generation），属于 cloudera。但随着 FLume 功能的扩展，Flume OG 代码工程臃肿、核心组件设计不合理、核心配置不标准等缺点暴露出来，尤其是在 Flume OG 的最后一个发行版本 0.94.0 中，日志传输不稳定的现象尤为严重，这点可以在 BigInsights 产品文档的 troubleshooting 板块发现。为了解决这些问题，2011 年 10 月 22 号，cloudera 完成了 Flume-728，对 Flume 进行了里程碑式的改动：重构核心组件、核心配置以及代码架构，重构后的版本统称为 Flume NG（next generation）；改动的另一原因是将 Flume 纳入 apache 旗下，cloudera Flume 改名为 Apache Flume。

### 准备工作：

安装java并设置java环境变量，在`/etc/profile`中加入
```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64
export PATH=$PATH:$JAVA_HOME/bin
```

### 注意事项
需要启动多个shell脚本交互客户端进行验证，运行中的客户端不要停止。

### 安装flume
+ 下载：
```
wget http://mirrors.hust.edu.cn/apache/flume/1.7.0/apache-flume-1.7.0-bin.tar.gz
```

+ 解压：
```
tar -zxvf apache-flume-1.7.0-bin.tar.gz
```

+ 
移动到指定目录：
```
mv apache-flume-1.7.0 /usr/local/share/applications/
```

### 配置
```
cp conf/flume-conf.properties.template conf/flume-conf.properties
cp conf/flume-env.sh.template conf/flume-env.sh
```
+ fluem-env.sh

```
# Enviroment variables can be set here.
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64

# Give Flume more memory and pre-allocate, enable remote monitoring via JMX
# export JAVA_OPTS="-Xms100m -Xmx2000m -Dcom.sun.management.jmxremote"

# Note that the Flume conf directory is always included in the classpath.
FLUME_CLASSPATH=/usr/local/share/applications/apache-flume-1.7.0-bin/lib
```

### 单机配置
在conf目录下创建single-node.conf文件，并将下面内容复制粘贴
```
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 8888

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

### 运行

```
bin/flume-ng agent --conf conf --conf-file conf/single-node.conf --name a1 -Dflume.root.logger=INFO,console
```

+ 命令行输出

```
[root@centos-a apache-flume-1.7.0-bin]# bin/flume-ng agent --conf conf --conf-file conf/single-node.conf --name a1 -Dflume.root.logger=INFO,console
Info: Sourcing environment configuration script /usr/local/share/applications/apache-flume-1.7.0-bin/conf/flume-env.sh
Info: Including Hive libraries found via () for Hive access
+ exec /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/bin/java -Xmx20m -Dflume.root.logger=INFO,console -cp '/usr/local/share/applications/apache-flume-1.7.0-bin/conf:/usr/local/share/applications/apache-flume-1.7.0-bin/lib/*:/usr/local/share/applications/apache-flume-1.7.0-bin/lib:/lib/*' -Djava.library.path= org.apache.flume.node.Application --conf-file conf/single-node.conf --name a1
2017-08-21 20:35:11,191 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.node.PollingPropertiesFileConfigurationProvider.start(PollingPropertiesFileConfigurationProvider.java:62)] Configuration provider starting
2017-08-21 20:35:11,199 (conf-file-poller-0) [INFO - org.apache.flume.node.PollingPropertiesFileConfigurationProvider$FileWatcherRunnable.run(PollingPropertiesFileConfigurationProvider.java:134)] Reloading configuration file:conf/single-node.conf
2017-08-21 20:35:11,214 (conf-file-poller-0) [INFO - org.apache.flume.conf.FlumeConfiguration$AgentConfiguration.addProperty(FlumeConfiguration.java:930)] Added sinks: k1 Agent: a1
2017-08-21 20:35:11,214 (conf-file-poller-0) [INFO - org.apache.flume.conf.FlumeConfiguration$AgentConfiguration.addProperty(FlumeConfiguration.java:1016)] Processing:k1
2017-08-21 20:35:11,214 (conf-file-poller-0) [WARN - org.apache.flume.conf.FlumeConfiguration$AgentConfiguration.addProperty(FlumeConfiguration.java:1046)] Invalid property specified: conf
2017-08-21 20:35:11,216 (conf-file-poller-0) [WARN - org.apache.flume.conf.FlumeConfiguration.<init>(FlumeConfiguration.java:101)] Configuration property ignored: mple.conf = A single-node Flume configuration
2017-08-21 20:35:11,216 (conf-file-poller-0) [INFO - org.apache.flume.conf.FlumeConfiguration$AgentConfiguration.addProperty(FlumeConfiguration.java:1016)] Processing:k1
2017-08-21 20:35:11,240 (conf-file-poller-0) [WARN - org.apache.flume.conf.FlumeConfiguration$AgentConfiguration.isValid(FlumeConfiguration.java:320)] Agent configuration for 'mple' does not contain any channels. Marking it as invalid.
2017-08-21 20:35:11,240 (conf-file-poller-0) [WARN - org.apache.flume.conf.FlumeConfiguration.validateConfiguration(FlumeConfiguration.java:127)] Agent configuration invalid for agent 'mple'. It will be removed.
2017-08-21 20:35:11,240 (conf-file-poller-0) [INFO - org.apache.flume.conf.FlumeConfiguration.validateConfiguration(FlumeConfiguration.java:140)] Post-validation flume configuration contains configuration for agents: [a1]
2017-08-21 20:35:11,240 (conf-file-poller-0) [INFO - org.apache.flume.node.AbstractConfigurationProvider.loadChannels(AbstractConfigurationProvider.java:147)] Creating channels
2017-08-21 20:35:11,253 (conf-file-poller-0) [INFO - org.apache.flume.channel.DefaultChannelFactory.create(DefaultChannelFactory.java:42)] Creating instance of channel c1 type memory
2017-08-21 20:35:11,270 (conf-file-poller-0) [INFO - org.apache.flume.node.AbstractConfigurationProvider.loadChannels(AbstractConfigurationProvider.java:201)] Created channel c1
2017-08-21 20:35:11,271 (conf-file-poller-0) [INFO - org.apache.flume.source.DefaultSourceFactory.create(DefaultSourceFactory.java:41)] Creating instance of source r1, type netcat
2017-08-21 20:35:11,304 (conf-file-poller-0) [INFO - org.apache.flume.sink.DefaultSinkFactory.create(DefaultSinkFactory.java:42)] Creating instance of sink: k1, type: logger
2017-08-21 20:35:11,308 (conf-file-poller-0) [INFO - org.apache.flume.node.AbstractConfigurationProvider.getConfiguration(AbstractConfigurationProvider.java:116)] Channel c1 connected to [r1, k1]
2017-08-21 20:35:11,328 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:137)] Starting new configuration:{ sourceRunners:{r1=EventDrivenSourceRunner: { source:org.apache.flume.source.NetcatSource{name:r1,state:IDLE} }} sinkRunners:{k1=SinkRunner: { policy:org.apache.flume.sink.DefaultSinkProcessor@3b1f88a2 counterGroup:{ name:null counters:{} } }} channels:{c1=org.apache.flume.channel.MemoryChannel{name: c1}} }
2017-08-21 20:35:11,336 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:144)] Starting Channel c1
2017-08-21 20:35:11,360 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:159)] Waiting for channel: c1 to start. Sleeping for 500 ms
2017-08-21 20:35:11,453 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.register(MonitoredCounterGroup.java:119)] Monitored counter group for type: CHANNEL, name: c1: Successfully registered new MBean.
2017-08-21 20:35:11,453 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.start(MonitoredCounterGroup.java:95)] Component type: CHANNEL, name: c1 started
2017-08-21 20:35:11,861 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:171)] Starting Sink k1
2017-08-21 20:35:11,863 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:182)] Starting Source r1
2017-08-21 20:35:11,869 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.source.NetcatSource.start(NetcatSource.java:155)] Source starting
2017-08-21 20:35:11,881 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.source.NetcatSource.start(NetcatSource.java:169)] Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/0:0:0:0:0:0:0:1:8888]
```

### 测试
+ 新窗口执行telnet命令

```
[root@centos-a ~]# telnet localhost 8888
Trying ::1...
Connected to localhost.
Escape character is '^]'.
hello
OK
TyurinTsien
OK
```

+ 在原来启动flume的命令行中可以看到：

```
2017-08-21 21:22:54,680 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 68 65 6C 6C 6F 0D                               hello. }
2017-08-21 21:48:12,889 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 54 79 75 72 69 6E 54 73 69 65 6E 0D             TyurinTsien. }
```

