---
layout: post
title:  JNA调用C++函数时关于字节序的一些问题
categories: exception
tags: [java, jna, experience]
no-post-nav: true
comments: true
---

# JNA调用C++函数时关于字节序的一些问题

最近在用JNA调用C++函数的时候，遇到了一些问题。jvm采用的端模式为`big-endian` ，但是处理器一般采用 `little-endian` 模式进行数据存放，这里对于字节流的处理比较容易出问题。
规避这些问题，需要对下面这些概念有一定的了解。

### **字节序**

>字节顺序，又称端序或尾序（英语：Endianness），在计算机科学领域中，指存储器中或在数字通信链路中，组成多字节的字的字节的排列顺序。

字节序与大小端模式的概念相似。

### **端模式**

> 字节的排列方式有两个通用规则。例如，一个多位的整数，按照存储地址从低到高排序的字节中，如果该整数的最低有效字节在最高有效字节的前面，则称小端序；反之则称大端序。

- `big-endian`模式，比较符合我们的阅读习惯，高位字节存储在低位地址，低位字节存储在高位地址
- `little-endian`模式，高位字节存储在高位地址，低位字节存储在低位地址



如值 `0x01020304`

*大端模式在内存中的表示*

```
00000001 00000010 00000011 00000100
```

*小端模式在内存中的表示*

```
00000100 00000011 00000010 00000001
```



------

因为不同机器类型可能采用不同标准的端模式，所以在设计需要进行网络传输或跨语言调用的程序时需要考虑端模式的问题。
1. C、C++的语言的字节序依赖于机器。

2. 由于JVM的存在，java的字节序与机器平台无关，一般的JVM实现中字节序都是大端。

3. 在网络传输中一般采用网络字节顺序，也就是大端。

   

### JNA对于字节序的处理

对于**JNA**，这里分为两种情况：
	1. 对于如long、double之类的基本类型或指针类型（ByReference），JNA会自动进行转换，可以忽略字节序带来的影响。
	2. 对于非基本类型，就需要手动对字节序进行处理了。

一般来讲JVM实现中字节序都是大端，因此可以通过判断CPU字节序是否为大端来决顶是否需要进行大小端转换。对byte数组和String的转换比较简单，如果是long之类数值类型，转换的时候就会复杂一些。
关于CPU字节序的判断，JDK中提供了一个很好的示例
	
```java
//java.nio.Bits
static {
    long a = unsafe.allocateMemory(8);
    try {
        unsafe.putLong(a, 0x0102030405060708L);
        byte b = unsafe.getByte(a);
        switch (b) {
        case 0x01: byteOrder = ByteOrder.BIG_ENDIAN;     break;
        case 0x08: byteOrder = ByteOrder.LITTLE_ENDIAN;  break;
        default:
            assert false;
            byteOrder = null;
        }
    } finally {
        unsafe.freeMemory(a);
    }
}
```

