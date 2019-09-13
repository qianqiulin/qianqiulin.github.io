---
layout:     post
title:      "centos7 升级 glibc-2.23"
subtitle:   "gcc"
date:       2019-09-13 21:58:11
author:     "TyurinTsien"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - Centos
---

# CentOS 7 升级 GCC
- 生产中在添加环境变量 LD_LIBRARY_PATH 以后出现所有命令都报错,是因为项目编译的时候使用了高版本的glibc,centos7 自带的glibc版本比较低，需要升级

## 下载 glibc-2.23
http://ftp.gnu.org/gnu/libc/

## 安装
```
cd /usr/local/src
tar xf glibc-2.23.tar.gz
mkdir glibc-build
cd glibc-build (一定要在新建的目录中操作)
../glibc-2.23/configure --prefix=/usr
make
make install
```
## 验证
```
ldd --version
```
## 报错解决：
- 在make install 时会跳出错误（类似的应该是因为软链接的版本不对造成的）
```
gawk: error while loading shared libraries: /lib64/libm.so.6: invalid ELF header
make[2]: *** [/disk1/software/gcc/glibc-2.23/build/math/stubs] Error 127
make[2]: Leaving directory `/disk1/software/gcc/glibc-2.23/math‘
make[1]: *** [math/subdir_install] Error 2
```
- 解决办法(在另外的窗口执行)：
```
cd /lib64
unlink libm.so.6
ln -s libm-2.23.so libm.so.6
```