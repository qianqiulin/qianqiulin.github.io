---
layout:     post
title:      "mac OSX中安装启动zookeeper"
subtitle:   "brew 安装 zookeeper"
date:       2017-08-23 20:24:49
author:     "TyurinTsien"
header-img: "img/post-bg-osxInstallZookeeper.jpg"
catalog: true
tags:
    - Zookeeper
---

## mac OSX中安装启动zookeeper

最近在自己mac上折腾各个大数据相关的组件，这篇文章也算给自己留个备忘。

### 安装

+ mac中支持brew安装zookeeper

```
$brew install zookeeper
==> Downloading https://homebrew.bintray.com/bottles/zookeeper-3.4.6_1.mavericks.bottle.2.tar.gz
######################################################################## 100.0%
==> Pouring zookeeper-3.4.6_1.mavericks.bottle.2.tar.gz
==> Caveats
To have launchd start zookeeper at login:
  ln -sfv /usr/local/opt/zookeeper/*.plist ~/Library/LaunchAgents
Then to load zookeeper now:
  launchctl load ~/Library/LaunchAgents/homebrew.mxcl.zookeeper.plist
Or, if you don't want/need launchctl, you can just run:
  zkServer start
```

+ 安装后，在/usr/local/etc/zookeeper/目录下，已经有了缺省的配置文件。

```
ls /usr/local/etc/zookeeper
defaults        log4j.properties    zoo.cfg         zoo_sample.cfg
```
+ 缺省配置[/usr/local/etc/zookeeper/zoo.cfg] 内容如下

```
bogon:zookeeper TyurinTsien$ more zoo.cfg
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/usr/local/var/run/zookeeper/data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
```

### 启动服务

+ 执行命令zkServer

```
$zkServer 
JMX enabled by default
Using config: /usr/local/etc/zookeeper/zoo.cfg
Usage: ./zkServer.sh {start|start-foreground|stop|restart|status|upgrade|print-cmd}

$ zkServer  status
JMX enabled by default
Using config: /usr/local/etc/zookeeper/zoo.cfg
Error contacting service. It is probably not running.

$ zkServer  start
JMX enabled by default
Using config: /usr/local/etc/zookeeper/zoo.cfg
Starting zookeeper ... STARTED
```

### 查看zookeeper运行及状态

+ 安装后，可以看到zookeeper提供了zkCli等工具进行.

```
$zkCli
Connecting to localhost:2181

Welcome to ZooKeeper!
JLine support is enabled
[zk: localhost:2181(CONNECTING) 0] 
[zk: localhost:2181(CONNECTING) 0] 
WATCHER::

WatchedEvent state:SyncConnected type:None path:null

[zk: localhost:2181(CONNECTED) 0] ls
[zk: localhost:2181(CONNECTED) 1] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 2] ls /zookeeper
[quota]
[zk: localhost:2181(CONNECTED) 3] ls /zookeeper/quota
[]
[zk: localhost:2181(CONNECTED) 4] 
```