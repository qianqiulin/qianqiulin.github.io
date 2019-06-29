---
layout:     post
title:      "pip安装python包过程中遇到的坑"
subtitle:   "pip install"
date:       2017-10-18 08:15:24
author:     "TyurinTsien"
header-img: "img/post-bg-python.jpg"
catalog: true
tags:
    - Python
---

## EnvironmentError:mysql config not found
* pip install MySQL-python 出现mysql config not found 的错误，解决方法如下，安装 mysql-devel
```
sudo yum install mysql-devel
```

## command 'gcc' failed with exit status 1 
* 明明装了gcc的，怎么会不行呢，然后发觉是failed不是not found，这说明这个错误个gcc没多大关系，应该是缺少某些功能模块，然后谷歌了一下，先后安装了python-devel,libffi-devel后还是不行，最后发觉要安装openssl-devel才行
可如下命令行安装：
```
yum install gcc libffi-devel python-devel openssl-devel
```