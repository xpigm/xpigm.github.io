---
layout: post
title:  centos7升级libc.so.6
categories: 
tags: []
no-post-nav: true
comments: true
---

**升级libc有风险**



运行在libc版本较高的系统下编译的so，往往会遇到以下错误

```
libc.so.6: version `GLIBC_2.18' not found
```



优先考虑重新编译so。或升级当前系统libc。

使用strings查看当前系统libc包含的版本 

```
strings /lib64/libc.so.6 |grep GLIBC_
```



### 升级libc

下载源码

```shell
wget http://mirrors.ustc.edu.cn/gnu/libc/glibc-2.18.tar.gz
```



编译、安装

```shell
tar -zxvf glibc-2.18.tar.gz
cd glibc-2.18

mkdir build
cd build

#编译
../configure --prefix=/usr
make -j4
make install
```



验证是否升级成功

```shell
strings /lib64/libc.so.6 |grep GLIBC_
```





