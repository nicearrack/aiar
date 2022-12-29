---
title: "Pentaho-Kettle里踩过的坑"
description: 那些在Kettle里踩过的坑，说多了都是泪。。
date: 2021-04-28T11:32:48+08:00
image:
math: 
license: 
hidden: false
comments: true
draft: true
categories:
    - Tech
tags:
    - ETL
---

## 连接数据库使用的是 sid
创建Oracle的DB连接时，kettle使用的是 sid(实例名instance_name) 来建立连接，sid默认和数据库名一致，但有可能更改，所以当使用数据库名连接不同时，可以使用一个高权限的用户登录目标数据库，输入下面语句以查询。

``` sql
select instance_name from  V$instance;
```

## 环境变量的引号问题
kettle中的环境变量比较坑，等号后面的环境变量不会自动给自己加引号，但是函数括号里的环境变量会给自己加引号，比如下面这段不会报错

``` sql
SELECT TO_DATE(${DQSJC},'YYYYMMDDHH24MISS') AS RUNNINGTIME
FROM SJJR_TAB_TRAN_INFO
WHERE TRAN_ID = '${TRANID}' AND JOB_ID = '${JOBID}';
```

而去掉where后的那些引号，就会报错；或者给括号里的变量加上引号后，将不被识别为环境变量(大概是这样，可能需要测试的同事反复实验确认)。

所以建议给等号后面的环境变量加上引号，括号里的就别加了。当然数字的也就看情况了。

## 设置变量长度很重要
设置变量时尽量给出长度类型，也去掉左右空格，不然 js 或其他脚本里面拼接变量时会前后有空格啥的

## 资源库文件夹
资源库可以添加文件夹，本地跑时只需要修改执行的当前目录等环境变量即可。

但是，如果要发布到远程服务器跑，那即使配置了环境变量，远程服务器也找不到位置。

## kettle 8.3 无法使用表导入
## kettle 9.2 无法多个客户端同时打开资源库

