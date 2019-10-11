---
layout:     post
title:      Linux优化-根据阿里云java开发手册
subtitle:   根据阿里云java开发手册优化linux，使得java更优运行
date:       2019-05-11
author:     Sunsj
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - java
---

1. 高并发服务器建议调小TCP协议的time_wait超时时间。

```
$ sudo vi  /etc/sysctl.conf

#在最下方添加
net.ipv4.tcp_fin_timeout = 30

```

2. 调大服务器所支持的最大文件句柄数（File Descriptor，简写为fd）。

```
# 系统默认的是1024
$ sudo ulimit -n 4096 

$ sudo uimin -a  | grep "open files"

open files                      (-n) 4096

```

3. 给JVM设置-XX:+HeapDumpOnOutOfMemoryError参数，让JVM碰到OOM场景时输出dump信息。在线上生产环境，JVM的Xms和Xmx设置一样大小的内存容量，避免在GC 后调整堆大小带来的压力。


```
#前台运行
java -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -Xms1024m -Xmx1024m -Xmn256m -Xss256k -XX:SurvivorRatio=8 -XX:+UseConcMarkSweepGC -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=${目录}  -jar /jar包路径 

# 后台运行
nohup java -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -Xms1024m -Xmx1024m -Xmn256m -Xss256k -XX:SurvivorRatio=8 -XX:+UseConcMarkSweepGC -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=${目录}  -jar /jar包路径 
```


> 参考链接：https://zhuanlan.zhihu.com/p/31803182