---
title: 2019-12-24 Linux常用命令和调试方法
tags: Linux    
---


**设置环境变量**
1、当前shell有效：
export设置当前shell的环境变量，然后通过env查看该环境变量的值。
``` javascript
# export MYSQL_TEST_DBNAME=test_db
# env | grep test_db
```       
+ 打包指定类型文件： # tar -zcvf code.tar.gz $( find -name "*.cpp" -or -name "*.h" -or -name "*makefile*" -or -name "*makefile.def*" )            
+ 打包排除指定类型： # tar -zcvf cgi-bin.tar.gz * --exclude='*.sh'       
+ 定义shell变量： CODE_PATH='/data/xxx/code/weself'           


