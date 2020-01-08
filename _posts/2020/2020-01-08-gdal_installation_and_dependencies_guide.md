---
layout: post
title:  GDAL构建指南
categories: other
tags: [other]
no-post-nav: true
comments: true
---

## 目标
本文编写目的是搭建适用于 Centos 7 + tomcat 的 GDAL 运行环境

## Prerequisite
1. 基于 centos 7 构建
2. 系统内已有 Java 环境，建议版本高于 1.8.0，确保 $JAVA_HOME环境变量正确设置
3. 系统内已有 gcc 编译构建环境

## 安装 libproj 依赖(optional)

```
ldconfig -p | grep proj
yum install proj
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/
wget https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/p/proj-4.8.0-4.el7.x86_64.rpm
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install proj
ldconfig -p | grep proj
ls /lib64/libproj*
ln -s /lib64/libproj.so.0.7.0 /lib64/libproj.so
```

## 下载 GDAL source 包

首先下载基础包, 并解压到自定义目录

```
wget http://download.osgeo.org/gdal/2.3.1/gdal-2.3.1.tar.gz
tar -xzvf gdal-2.3.1.tar.gz
```

其次, 构建 gdal binaries

```
cd gdal-2.3.1
./configure --with-java=yes
make
make install
# Install 之后，可以执行 gdalinfo --version 命令验证
```

然后, 构建 gdal Java libs
```
cd swig
cd java

# 修改 java.opt 文件中的 'JAVA_HOME' 变量为系统java配置
# JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk/
make
make install

# 拷贝当前目录 .libs/ 下的 *.so 文件到 /usr/local/lib/ 目录
cp .lib/*.so /usr/local/lib/

# 拷贝构建的 gdal.jar 到指定目录确保 Java runtime 可以找到此 jar 文件
cp gdal.jar $JAVA_HOME/jre/lib/ext/

```

## 配置系统动态库查找路径

```
echo "/usr/local/lib" > /etc/ld.so.conf.d/gdal.conf
ldconfig
```

```
# ldconfig -p | grep gdal
# 确保以下库可以被搜索到
  libgdalalljni.so.20 (libc6,x86-64) => /usr/local/lib/libgdalalljni.so.20
  libgdalalljni.so (libc6,x86-64) => /usr/local/lib/libgdalalljni.so
  libgdal.so.20 (libc6,x86-64) => /usr/local/lib/libgdal.so.20
  libgdal.so (libc6,x86-64) => /usr/local/lib/libgdal.so
```

## 配置 Tomcat
修改 Tomcat catalina.sh 文件配置选项

添加 `-Djava.library.path=/usr/local/lib/` 到 `JAVA_OPTS` 参数中,
例如：

```
JAVA_OPTS="-server  -Duser.timezone=GMT+08 -Djava.library.path=/usr/local/lib/ -Xms512m -Xmx4096m"
```

## 注意的问题
请尽量将 Tomcat 的 Java虚拟机的内存设置的高一些，如上节的 `4096m`.
