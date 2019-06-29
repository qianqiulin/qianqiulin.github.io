---
layout:     post
title:      "Install hive on Mac with Homebrew"
subtitle:   " BigData Hive"
date:       2017-07-12 10:19:31
author:     "TyurinTsien"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - Hive
---


# Install hive on Mac with Homebrew

+ TyurinTsien
+ 2017年07月12日10:19:31

## Prerequisite
MySQL 5.6.22 is already installed.

## Versions
+ hadoop 2.8.0
+ hive 2.1.1

  were installed by Homebrew.

## Install Hive and Hadoop
```markdown
 $ brew update
 $ brew install hive
```

## Configure enviromental variables
```markdown
# ~/.bashrc
export HADOOP_HOME=/usr/local/Cellar/hadoop/2.8.0
export HIVE_HOME=/usr/local/Cellar/hive/2.1.1/libexec
```

## Download JDBC
[Go to mysql page and download the latest jdbc (sign up is required)](http://dev.mysql.com/downloads/connector/j/)
```markdown
$ tar zxvf mysql-connector-java-5.1.35.tar.gz
$ sudo cp mysql-connector-java-5.1.35/mysql-connector-java-5.1.35-bin.jar /usr/local/Cellar/hive/2.1.1/libexec/lib/
```

## Setup MySQL database
```markdown
$ mysql
mysql> CREATE DATABASE metastore;
mysql> USE metastore;
mysql> CREATE USER 'hiveuser'@'localhost' IDENTIFIED BY 'password';
mysql> GRANT SELECT,INSERT,UPDATE,DELETE,ALTER,CREATE ON metastore.* TO 'hiveuser'@'localhost';
```

## Copy hive-default-xml to hive-site.xml
```markdown
$ cd /usr/local/Cellar/hive/1.1.1/libexec/conf
$ cp hive-default.xml.template hive-site.xml
```

## Edit following lines in hive-site.xml
```markdown
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://localhost/metastore</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>hiveuser</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>password</value>
</property>
<property>
  <name>datanucleus.fixedDatastore</name>
  <value>false</value>
</property>
<property>
    <name>hive.exec.local.scratchdir</name>
    <value>/tmp/hive</value>
    <description>Local scratch space for Hive jobs</description>
</property>
<property>
    <name>hive.downloaded.resources.dir</name>
    <value>/tmp/hive</value>
    <description>Temporary local directory for added resources in the remote file system.</description>
</property>
<property>
    <name>hive.querylog.location</name>
    <value>/tmp/hive</value>
    <description>Location of Hive run time structured log file</description>
</property>
```

## Run hive
```markdown
$hive
hive > show tables;
```


- [refs](https://noobergeek.wordpress.com/2013/11/09/simplest-way-to-install-and-configure-hive-for-mac-osx-lion/)