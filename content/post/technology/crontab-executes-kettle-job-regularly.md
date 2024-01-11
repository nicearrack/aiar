---
title: "crontab定时执行Kettle Job"
description: 使用 crontab 定时调度 Kettle 作业
date: 2024-01-11T10:21:10+08:00
image: https://s11.ax1x.com/2024/01/09/pFp61fK.png
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - 技术
tags:
    - Kettle
    - Linux
---

**注：使用 corntab 进行调度务必请将Kettle自身调度和循环关闭**

## 1.环境准备
### 1.1 检查资源库配置文件 `.kettle/repositories.xml` 存在性
``` shell
$ cat /home/etl/.kettle/repositories.xml
```
不存在则以 `1.2` 中内容模板新建文件
### 1.2 检查资源库路径是否符合预期
将 `<base_directory></base_directory>` 标签中的目录位置修改为期望的资源库路径
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<repositories>
  <repository>
    <id>KettleFileRepository</id>
    <name>Resources</name>
    <description>File repository-Resources</description>
    <is_default>true</is_default>
    <base_directory>/home/etl/data-integration/Resources</base_directory>
    <read_only>N</read_only>
    <hides_hidden_files>N</hides_hidden_files>
  </repository>
</repositories>
```
### 1.3 上传文件到服务器资源库
将配置好的作业和转换xml文件上传至服务器资源库，例如使用 `xftp` 等工具

## 2.调试配置调度
### 2.1 测试单次调用
使用 `curl` 工具通过Kettle自带接口触发作业单次执行。

``` shell
$ curl -u {用户名}:{密码} "http://{ip}:{port}/kettle/executeJob/?rep={资源库名称}&job={作业名称}"`
```

说明：
1. 用户名和密码是Kettle子服务器的密码
2. `{}` 代表替换参数，url中有 `&` 所以整体需要加上 `""`
3. Kettle自带接口的触发机制不在本文讨论范围，只需要知道可以以get请求固定格式url即可执行指定作业就好了

实例：触发执行本机Resources资源库的0494J作业
``` shell
$ curl -u cluster:cluster "http://127.0.0.1:8080/kettle/executeJob/?rep=Resources&job=0494J"
```

[这里](https://www.ruanyifeng.com/blog/2019/09/curl-reference.html)是curl的用法指南。

通了会显示如下信息，异常则会显示相应异常信息:
``` xml
<webresult>
  <result>OK</result>
  <message>Job started</message>
  <id>d996f743-bc27-46da-862b-6ea8be6cf4d2</id>
</webresult>
```
### 2.2 配置循环调度
使用 `crontab` 工具实现循环调度。
``` shell
$ crontab -l  # 显示当前用户所有cron任务
$ crontab -e  # 编辑修改增删cron任务，编辑器走VISUAL或EDITOR环境变量，可通过 `echo $EDITOR` 命令查看
$ tail -f /var/log/cron  # 查看cron任务执行日志
```
实例：创建一个每5分钟执行一次的调度任务
``` shell
$ crontab -e
*/5 * * * * /usr/bin/curl -u cluster:cluster "http://127.0.0.1:8080/kettle/executeJob/?rep=Resources&job=0494J"
```
[这里](https://crontab.guru/)是crontab的用法指南。
[这里](https://cloud.tencent.com/developer/article/1951645)是vim的用法指南。
