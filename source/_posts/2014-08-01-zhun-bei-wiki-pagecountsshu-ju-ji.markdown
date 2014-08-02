---
layout: post
title: "准备wiki pagecounts数据集"
date: 2014-08-01 23:47:33 -0700
comments: true
categories: 
---

一直想跟着潮流想在业余学习一下Spark，老是入不了门的感觉，环境都搭好了，但是也不清楚可以用来作做什么。Google发现ampcamp有一个练习叫[Data Exploration Using Spark](http://ampcamp.berkeley.edu/3/exercises/data-exploration-using-spark.html)。讲到通过RDD上的一下操作看看wikipedia一些页面的pagecount数据。其实真正想开始实验还是要经过一些步骤的。我打算在自己的一个小集群上学习一下，先看看怎么准备数据吧。    

我把2009年5月份的数据先下载下来。下载的时间比较长，我是通过ssh连到Linux机器上的，差不多要从公司回去了，我需要把wget放到后台去跑，不然我断开ssh时wget就跟着被杀掉了（是这样的吧？）。
```sh
mkdir tmp && cd tmp
wget -brq --no-parent https://dumps.wikimedia.org/other/pagecounts-raw/2009/2009-05/
```
所有这一个月的数据有40G左右，全部解压完得200G以上，磁盘空间没那么大，我就准备拿1号到10号的数据看看就好。新建一个目录叫wikidata，把十天的数据拷贝过来。而且直接下下来的数据不是ampcamp给的那个数据格式，少了date_time这一列，所以还需要做些处理，让数据格式变为：<date_time> <project_code> <page_title> <num_hits> <page_size>
```sh
for f in `ls pagecounts-200905*`
do
gunzip $f
done

for f in `ls pagecounts-200905*`
do
sed -i '/^[*]/d' $fn
datetime=`echo $fn|awk -F'pagecounts-' '{print $2}'`
#用双引号datatime的值才能被读到
sed -i "s/^/$datetime /" $fn
done

mv _pagecounts pagecounts
hdfs dfs -mkdir /wiki
hdfs dfs -put pagecounts /wiki/

bash-4.1$ hdfs dfs -ls -h /wiki/pagecounts
-rw-r--r--   3 xilan supergroup      60.9 G 2014-08-02 07:38 /wiki/pagecounts
```
这个过程可能需要很长的时间，你可以先去做点别的事情。这里我还把很多小文件cat出来append到一个文件，得到一个大文件，其实也可以不用，SparkContext有API可以读很多文件。
我顺便把pagecounts放到HDFS上去。然后就可以开始跟着那些步骤，开始走入RDD的API。
```scala
bash-4.1$ pwd
/scratch/xilan/spark-1.0.0

bash-4.1$ bin/spark-shell

scala> sc
res0: org.apache.spark.SparkContext = org.apache.spark.SparkContext@55bdc1ed

scala> val pagecounts = sc.textFile("hdfs://namenode:9000/wiki/pagecounts")
14/08/02 07:48:11 INFO storage.MemoryStore: ensureFreeSpace(218411) called with curMem=0, maxMem=309225062
14/08/02 07:48:11 INFO storage.MemoryStore: Block broadcast_0 stored as values to memory (estimated size 213.3 KB, free 294.7 MB)
pagecounts: org.apache.spark.rdd.RDD[String] = MappedRDD[1] at textFile at <console>:12

scala> pagecounts.take(10)
...
14/08/02 07:50:22 INFO spark.SparkContext: Job finished: take at <console>:15, took 0.09938809 s
res0: Array[String] = Array(20090501-000000 aa.b Main_Page 3 16288, 20090501-000000 aa.b Special:RecentChangesLinked/User:Az1568 1 5745, 20090501-000000 aa.b Special:Recentchangeslinked/User:Az1568 1 1013, 20090501-000000 aa.b Special:Statistics 1 840, 20090501-000000 aa.b Template:Delete 1 26601, 20090501-000000 aa.d Main_Page 2 5442, 20090501-000000 aa Main_Page 4 19955, 20090501-000000 aa User:Alecs.bot 1 13388, 20090501-000000 ab %D0%90%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D0%BD%D0%B5%D1%82 1 465, 20090501-000000 ab %D0%98%D1%85%D0%B0%D0%B4%D0%BE%D1%83_%D0%B0%D0%B4%D0%B0%D2%9F%D1%8C%D0%B0 1 16098)

scala> pagecounts.take(10).foreach(println)
...
20090501-000000 aa.b Main_Page 3 16288
20090501-000000 aa.b Special:RecentChangesLinked/User:Az1568 1 5745
20090501-000000 aa.b Special:Recentchangeslinked/User:Az1568 1 1013
20090501-000000 aa.b Special:Statistics 1 840
20090501-000000 aa.b Template:Delete 1 26601
20090501-000000 aa.d Main_Page 2 5442
20090501-000000 aa Main_Page 4 19955
20090501-000000 aa User:Alecs.bot 1 13388
20090501-000000 ab %D0%90%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D0%BD%D0%B5%D1%82 1 465
20090501-000000 ab %D0%98%D1%85%D0%B0%D0%B4%D0%BE%D1%83_%D0%B0%D0%B4%D0%B0%D2%9F%D1%8C%D0%B0 1 16098

scala> pagecounts.count
...
14/08/02 07:56:47 INFO scheduler.DAGScheduler: Stage 1 (count at <console>:15) finished in 142.603 s
14/08/02 07:56:47 INFO spark.SparkContext: Job finished: count at <console>:15, took 142.687984303 s
res2: Long = 1080525124
```
