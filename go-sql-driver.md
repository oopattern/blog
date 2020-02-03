---
title: 2019-12-23 go-sql-driver/mysql
tags: go,mysql
renderNumberedHeading: true
grammar_cjkRuby: true
---


**层次结构** 


**命令类图**
![enter description here](./images/1577106761713.png)


**测试用例**
1、函数参数为函数，也就是闭包，通过将函数作为参数，实现延时绑定的功能，也就是让闭包的运行延后执行。
2、客户端实例在协程间安全，每个协程可以并发的调用同一个客户端实例。

**字符集**
通过mysql命令导出字符乱码, 需要检查数据库和数据库表的字符集是否一致, 如果不一致， 需要通过mysql命令行 > set names utf8; 来指定对应导出的字符，和数据库表的字符集保持一致。
``` shell
// 全局字符集
MySQL [gongyi_donate]> show variables like '%char%';
// 单个数据库字符集
MySQL [gongyi_donate]> show create database gongyi_donate;
// 单个数据库表的字符集
MySQL [gongyi_donate]> show create table `gy_organization`\G
// 导出中文文本字段,在终端执行导出命令
# mysql  -hxxx -uyyy -pzzz database -Pport -s -e "select f_name from t_201999_undirectional_act limit 100" > test_name.txt
```


