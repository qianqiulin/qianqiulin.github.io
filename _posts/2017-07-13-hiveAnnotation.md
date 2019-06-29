---
layout:     post
title:      "Hive表中文注释乱码"
subtitle:   " 安装使用中发现的"
date:       2017-07-13 17:19:21
author:     "TyurinTsien"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - Hive
---


##  Hive表中文注释乱码

+ TyurinTsien 
+ 2017年07月13日17:19:21

最近在使用Hive是发现desc查看表时，注释中的中文都是一问号的形式显示。经过查资料知道，hive的元数据时存储在MySQL中的，因此我们需要对mysql中相关表的字符编码进行修改。特在此备忘：

### Hive默认情况下我们需要将数据库的编码设置为lanin1.
```markdown
mysql> alter database metastore character set latin1;
```

### 但为了以下是为了支持hive建表时插入中文注释 需要在mysql中做如下设置：

+ //修改字段注释字符集
```markdown
mysql> alter table COLUMNS_V2 modify column COMMENT varchar(256) character set utf8;
```

+ //修改表注释字符集
```markdown
mysql> mysql alter table TABLE_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;
```

+ //修改分区注释字符集
```markdown
mysql> alter table PARTITION_KEYS modify column PKEY_COMMENT varchar(4000) character set utf8;
```

不过改完之后还有一个问题:之前已经在hive中创建的表，中文仍然显示问号，但是新创建的表则可以正常显示中文。此问题尚未找到更好的解决方法，若有解决方法，请[留言](mailto:qianqiulin@outlook.com)告知，谢谢！

