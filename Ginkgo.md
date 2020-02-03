---
title: 2019-12-24 go test 和 Ginkgo测试框架的使用
tags: go test, Ginkgo
renderNumberedHeading: true
grammar_cjkRuby: true
---


**go test**
 说明：通过go test执行项目对应的模块测试用例。
 1、go build命令不会包含格式为"\_test.go"的文件。go test会执行go build，然后根据运行测试框架，通过-run参数指定具体的测试函数。
 2、go test如果指定某个go文件，但是该go文件依赖其他的go文件，需要包含其他文件才能运行。go test 可以直接指定目录，表示执行该目录下所有test文件的Testxxx(* test.Testing)函数。
 3、示例：执行go-sql-driver项目连接DB的测试函数，由于引用了当前目录其他go文件，因此go test需要制定路径为当前目录"./", -v表示打印详细信息。
``` javascript
# cd /data/xxx/go_path/pkg/mod/github.com/go-sql-driver/mysql@v1.4.1
# go test -v ./ -run=TestCustomDial
```
![enter description here](./images/1577152280644.png)

**go test运行报错**
1、在源代码根目录运行raft包的测试函数，**执行失败**，报错说找不到包：
``` shell
# go test -v ./  -run TestFindConflict
testing: warning: no tests to run
```
![enter description here](./images/1578538815733.png)
2、指定raft包，go test指定包名（相对目录名称）重新运行测试用例的函数，执行成功：
![enter description here](./images/1578539032262.png)
3、直接到指定包运行测试函数，也能执行成功：
![enter description here](./images/1578539203665.png)
4、查看官方命令解释说明，需要指定包名（文件夹相对目录）：
![enter description here](./images/1578539738882.png)


**Ginkgo**

