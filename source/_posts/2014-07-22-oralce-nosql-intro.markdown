---
layout: post
title: "Oralce NoSQL介绍"
date: 2014-07-22 05:01:57 -0700
comments: true
categories: 
---

SQLDev从4.1开始加了几个特性，其中一个就是对Oracle NoSQL的支持，这种事情当然就会交给我去做。估计再过一两个月就开始测了，我需要开始了解Oracle NoSQL。其实，说到NoSQL我首先会想到Redis、MongoDB和HBase那些，Oracle NoSQL之前没听过。


所有的分布式的东西估计都会提到高可靠性、高可用性和可扩展性，Oracle NoSQL当然也不能例外。数据已键值对的形式，通过哈希存储到各个节点（Storage Node），各个节点都会做备份来提供高可用性，默认是3份。这里我从单节点开始。    

从官网下载了企业版（kv-ee-3.0.9.tar.gz），直接在/home/xilan/用tar zxvf kv-ee-3.0.9.tar.gz解压得到kv-3.0.9文件夹。进入kv-3.0.9，该目录下有doc、examples、exttab、lib、LICENSE.txt和README.txt，前面四个是文件夹。

为了快速了解，我启动KVLite（Oracle NoSQL的单节点，单shard，单进程store）：   
```sh
java -Xmx256m -Xms256m -jar lib/kvstore.jar kvlite
Created new kvlite store with args:
-root ./kvroot -store kvstore -host myhost -port 5000 -admin 5001
```

store参数指定store的名字，host指定主机名字（我这里其实是localhost），在当前目录生成kvroot目录（因为第一次启动，以前还没有），用来存放NoSQL数据，默认在5000端口上监听，管理进程在5001端口打开。

还可以ping一下看看kvlite是否已经起来了：    
```sh
java -jar lib/kvstore ping -host localhost -port 5000
kvstore comprises 10 partitions and 1 Storage Nodes
Storage Node [sn1] on hostname:5000     Zone: [name=KVLite id=zn1 type=PRIMARY]    Status: RUNNING   Ver: 12cR1.3.0.9 2014-05-02 11:52:34 UTC  Build id: c9e715456512
        Rep Node [rg1-rn1]      Status: RUNNING,MASTER at sequence number: 37 haPort: 5006
```

接下来当然是准备HelloWorld了： 
```sh
bash-4.1$ javac -cp examples:lib/kvclient.jar examples/hello/HelloBigDataWorld.java
bash-4.1$ java -cp examples:lib/kvclient.jar hello.HelloBigDataWorld
Hello Big Data World!
```

好了，接下来可以开始看看examples下的例子，熟悉一下API，当然，最终还是要跑到几个节点上。
