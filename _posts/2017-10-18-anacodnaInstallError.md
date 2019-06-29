---
layout:     post
title:      "bunzip2: command not found"
subtitle:   "ananconda"
date:       2017-10-18 11:06:20
author:     "TyurinTsien"
header-img: "img/post-bg-anaconda.jpg"
catalog: true
tags:
    - Python
---

# 错误信息：bunzip2: command not found

```
[/root/anaconda2] >>> 
PREFIX=/root/anaconda2
Anaconda2-5.0.0-Linux-ppc64le.sh: line 317: bunzip2: command not found
tar: This does not look like a tar archive
tar: Exiting with failure status due to previous errors
[root@node1 ~]# 
```

## 解决办法： 
* 安装bzip2即可解决
```
[root@node1 ~]# yum install -y bzip2
```

# ImportError: No module named shutil_get_terminal_size

装好anaconda之后，控制台运行ipython 或者 ipython notebook 报错：ImportError: No module named shutil_get_terminal_size

## 解决方法
Google 得到的解决方法如下：
还是谷歌大法好啊，送上链接： 
http://stackoverflow.com/questions/37232446/ipython-console-cant-locate-backports-shutil-get-terminal-size-and-wont-load
```
conda config --add channels conda-forge
conda install backports.shutil_get_terminal_size
```