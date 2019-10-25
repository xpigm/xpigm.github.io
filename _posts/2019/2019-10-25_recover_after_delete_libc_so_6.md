---
layout: post
title:  解决centos7下libc.so.6被误删
categories: 
tags: []
no-post-nav: true
comments: true
---

误删或损坏`libc.so.6` 会导致系统大部分命令基本无法使用

### 解决思路

删除并重建损坏的libc软连接

> LD_PRELOAD是Linux系统的一个环境变量，它可以影响程序的运行时的链接（Runtime linker），它允许你定义在程序运行前优先加载的动态链接库。

```shell
[root@localhost ~]# ldconfig --help
Usage: ldconfig [OPTION...]
Configure Dynamic Linker Run Time Bindings.

  -c, --format=FORMAT        Format to use: new, old or compat (default)
  -C CACHE                   Use CACHE as cache file
  -f CONF                    Use CONF as configuration file
  -i, --ignore-aux-cache     Ignore auxiliary cache file
  -l                         Manually link individual libraries.
  -n                         Only process directories specified on the command
                             line.  Don't build cache.
  -N                         Don't build cache
  -p, --print-cache          Print cache
  -r ROOT                    Change to and use ROOT as root directory
  -v, --verbose              Generate verbose messages
  -X                         Don't generate links
  -?, --help                 Give this help list
      --usage                Give a short usage message
  -V, --version              Print program version

```



### 解决方案

1. 误删除：使用ldconfig自动重建软连接

   ```shell
   ldconfig -l -v /lib64/libc-2.18.so
   ```

2. 文件损坏：使用LD_PRELOAD指定正确的libc，删除并重建软连接

   ```shell
   #删除损坏的文件
   LD_PRELOAD=/lib64/libc-2.18.so rm /lib64/libc.so.6
   #重建软连接
   LD_PRELOAD=/lib64/libc-2.18.so ln -s /lib/libc-2.18.so /lib/libc.so.6
   ```

   

