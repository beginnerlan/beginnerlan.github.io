---
layout: post
title: "ora-01537 occurs when create new pluggable database"
date: 2014-07-29 00:26:48 -0700
comments: true
categories: 
---

今天在测试SQL Developer的时候，需要创建一个pdb，产生的SQL是这样的：    
```sql
CREATE PLUGGABLE DATABASE NEW_PDB ADMIN USER Admin IDENTIFIED BY top_secret
 FILE_NAME_CONVERT=(
  'D:\APP\ORACLE\ORADATA\ORA12CR1\PDBSEED\SYSTEM01.DBF', 'D:\APP\ORACLE\ORADATA\ORA12CR1\PDBSEED\SYSTEM01.DBF.clone',
  'D:\APP\ORACLE\ORADATA\ORA12CR1\PDBSEED\SYSAUX01.DBF', 'D:\APP\ORACLE\ORADATA\ORA12CR1\PDBSEED\SYSAUX01.DBF.clone',
  'D:\APP\ORACLE\ORADATA\ORA12CR1\PDBSEED\PDBSEED_TEMP01.DBF', 'D:\APP\ORACLE\ORADATA\ORA12CR1\PDBSEED\PDBSEED_TEMP01.DBF.clone'
  )
 STORAGE UNLIMITED
```

出现了ORA-01537（cannot add file 'D:\APP\ORACLE\ORADATA\ORA12CR1\PDBSEED\SYSTEM01.DBF.clone' - file already part of database），其实这次已经不是第一次遇见这个错误了，每次都不记得。因为是测试环境，我就想关掉数据库，找到这个文件然后删掉，重启一下。先找一下它们属于哪个表空间：
```sql
SQL> select distinct a.name tsname, b.name fname from v$tablespace a, v$datafile
 b where a.ts#=b.ts# and b.name like '%CLONE%';

TSNAME     FNAME
---------- ------------------------------------------------------------
SYSTEM     D:\APP\ORACLE\ORADATA\ORA12CR1\PDBSEED\SYSTEM01.DBF.CLONE
SYSAUX     D:\APP\ORACLE\ORADATA\ORA12CR1\PDBSEED\SYSAUX01.DBF.CLONE
```
