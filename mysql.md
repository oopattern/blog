---
title: 2020-7-8 mysql
tags: mysql
---

## **DB调试命令**   
+ show processlist;         // 查看客户端连接db的情况
+ show variables like '%conn%';     // 查看目前连接数的配置      

## **DB调试命令**       
+ # grep 'temporary password' /var/log/mysqld.log  
+ # mysqladmin -u root -p'qA,IRrTWC9kE' password ''       
+ > set global validate_password_policy=0;
+ > set global validate_password_mixed_case_count=0; 
+ > set global validate_password_number_count=0;   
+ > set global validate_password_special_char_count=0;
+ > set global validate_password_length=0;
+ > SET PASSWORD FOR 'root'@'localhost' = PASSWORD('');

## **常用DB命令**      
按分组统计字段次数： select f_dragon_id,count(1) from t_gy_dragon_transcode_2020 group by f_dragon_id  order by count(1) desc  limit 10\G              

扩字段：
alter table `t_gy_dragon_transcode` add f_pendimgs varchar(1024) DEFAULT '' after f_headimg;
alter table `t_gy_dragon_transcode` add f_imgstatus varchar(32) DEFAULT '' after f_headimg;

添加索引：      


mysql乱码问题：
https://www.cnblogs.com/chyingp/p/mysql-character-set-collation.html

mysql中select显示乱码问题：
https://www.cnblogs.com/qmfsun/p/4846467.html

批量导入数据到数据库：
1）csv导入：https://www.jianshu.com/p/95b7ef8d284b
2）官方示例：http://www.mysqltutorial.org/mysql-sample-database.aspx

select导数据kill:
https://stackoverflow.com/questions/18104843/mysql-query-with-large-number-of-records-gets-killed

查找字段为数字的列：
select f_transcode,f_donate_time,f_donate_id,f_donate_attr from t_wx_transcode_attr where f_donate_attr<>'' and (f_donate_attr REGEXP '[^0-9.]') =0 order by f_id desc  limit 100;

数据统计（20190201）：
awk使用方法：
1）捐款总额：awk -F'\t' '{if ($3 >= "2019-02-01 00:00:00" && $3 <= "2019-02-01 23:59:59") {sum = sum + $2}} END {print sum}' bless_export_20190218.txt
2）人次：awk -F'\t' '{if ($3 >= "2019-02-01 00:00:00" && $3 <= "2019-02-01 23:59:59") {sum = sum + 1}} END {print sum}' bless_export_20190218.txt
3）人数（去重）：

1）mysql -q -h100.65.202.233 -P3328 -uroot -pgongyi2008\~\!\@ gongyi_wx
2）select * from t_bless_flow limit 1\G
3）捐款人次： select count(*) from t_bless_flow where f_dt>='2019-02-01 00:00:00' and f_dt<='2019-02-01 23:59:59';
4）捐款人数： select count(distinct f_uid) from t_bless_flow where f_dt>='2019-02-01 00:00:00' and f_dt<='2019-02-01 23:59:59';
5）捐款金额： select sum(f_money) from t_bless_flow where f_dt>='2019-02-01 00:00:00' and f_dt<='2019-02-01 23:59:59';

常用mysql命令：
查询最多最多出现次数的一起捐：
select f_donate_id, count(1) as cnt from t_wx_transcode_attr where f_donate_time>='2019-07-01 00:00:00' and f_donate_time<'2019-09-02 00:00:00'  group by f_donate_id order by cnt desc limit 1;

1）数据库基本命令：
1.1）mysql --h
1.2）mysql -h192.168.201.75 -ulandlord -pbigbiglandlord
1.2）mysql> set character_set_results=utf8; // 设置查询编码为utf8
1.2）mysql> status // 查看状态，包括编码格式
1.2）mysql> create database test; //创建数据库
1.4）mysql> CREATE TABLE mytable (name VARCHAR(20), sex CHAR(1), birth DATE, birthaddr VARCHAR(20)); // 创建表
1.5）mysql> truncate table tablename; // 清空表内容
1.6）mysql> show databases; //查看数据库
1.7）mysql> show tables; // 查看表
1.8）mysql> desc qymatch_card; // 查看列
1.9）mysql> use landlord_ingot; // 使用表
1.10）mysql> select * from student where name='user100'; // 查询数据
1.11）mysql> select * from ingot_useringot9; // 查询数据
1.12）mysql> select * from ingot_useringot9 where ingot=1; // 查询数据
1.12）mysql> alter table `t_gy_dragon_transcode` add f_imgstatus varchar(1024) DEFAULT '' after f_headimg;  // 新增字段
1.13）mysql> select COUNT(*) from ingot_useringot9 where ingot=1; // 查询数据
1.14）mysql> insert into qymatch_card set content='hhh', ctime=123, active=0; // 插入数据
1.15）mysql> delete from qymatch_card where ctime=%d and cnum=%d limit 1 // 删除数据
1.16) mysql> update runoob_tbl SET runoob_title='学习 C++' WHERE runoob_id=3 // 修改数据
1.17）mysql> // 多表join
1.18）分组取前N大/小的数据：<http://www.manongjc.com/article/1082.html>
1.19）按照code, name进行降序排序：select * from a order by code, name desc limit 5;
1.20）显示一个表所有索引的SQL语句是： show index from 数据库名.表名
1.21）创建表索引：alter table student add index xxx(id); // xxx为索引名称，id为字段所在列
1.22）删除表索引：alter table student drop index xxx; // xxx为索引名称
1.22）删除主键索引：alter table student drop primary key; 
1.23）将csv导入mysql的操作方法：<https://blog.csdn.net/quiet_girl/article/details/71436108>
​ \# --CSV文件存放路径 --要将数据导入的表名
​ load data local infile 'F:/MySqlData/test1.csv'
​ into table student
​ fields terminated by ',' optionally enclosed by '"' escaped by '"'
​ lines terminated by '\r\n';
1.24）mysql数据类型：<https://www.cnblogs.com/xingmeng/archive/2012/10/24/2737455.html>
1.25）mysql去重distinct: 统计c5列不重复的个数
​ mysql> select count(distinct(c5)) from 99gy;

1）mysql索引注意事项：
​ 1、维护高的列创建索引，性别 不适合建立索引
​ 2、对where、on、group by、order by出现的列使用索引
​ 3、对较小的数据列使用索引
​ 4、为较长的字符串使用前缀索引
​ 5、不要过多使用索引，DML操作影响速度
​ 6、尽量组合索引，比多个单列索引效果好。
​ 7、索引列不存储null值，不能利用索引，变成全表扫描。

2）索引失效的情况：
​ 1、条件有or，如果想使用or，又想索引生效，只能将or条件每个列都加上索引
​ 2、like查询以%开头
​ 3、对于组合索引，不是使用的第一部分，则不会使用索引

3）索引的区别：
​ 主键索引 primary key：alter table `table_name` add primary key (`column`) 唯一索引的特殊类型，和唯一索引不同的是：not null、每个表只能有一个、可作外键、可以是多个字段组合。
​ 唯一索引 unique：alter table `table_name` add unique (`column`) 创建唯一索引的目的往往不是为了提高访问速度，而只是为了避免数据出现重复。
​ 普通索引 index：alter table `table_name` add index index_name(`column`)
​ 全文索引 fulltext：alter table `table_name` add fulltext (`column`)
​ 组合索引 ：alter table `table_name` add index_name(`column1`,`column2`,`column3`) INDEX（A，B，C）可以当做A或（A,B）的索引来使用，但不能当做B、C或（B，C）的索引来使用。
​ 外键索引 ：
​ 说明：因为主键的第二个作用是让其他表的外键引用自己，从而实现关系结构。一旦某个表的主键发生了变化，就会导致所有引用了该表的数据必须全部修改外键。
​ 很多Web应用的数据库并不是强约束（仅仅引用主键但并没有设置外键约束），修改主键会导致数据完整性直接被破坏。

4）数据库表的垂直拆分和水平拆分
​ 垂直拆分：数据表列的拆分，把一张列比较多的表拆分为多张表
​ 拆分原则：
​ 1、不常用的字段单独放在一张表
​ 2、把text、blob等大字段拆分放在附表
​ 3、经常组合查询的列放在一张表
​ 水平拆分：数据表行的拆分，表的行数超过200w，就会变慢，可以把一张表的数据拆成多张表来存放。
​ 拆分原则：
​ 取模拆分：如有一张400w的用户表user，为提高查询效率将其分成4张表user1，user2，user3，user4，按照uid取模的方式进行增删改查。
​ 其他：
​ 部分业务逻辑也可以通过地区、年份等字段来进行归档拆分。

5）MYSQL优化技巧
​ <https://coolshell.cn/articles/1846.html>
​ 1、使用查询缓存、使用$today = date("Y-m-d")代替CURDATE()函数，因为该函数返回会不定易变的值，用变量代替SQL函数，开启缓存
​ 2、EXPLAIN对应的select语句，全表扫描还是索引扫描，查询次数等。
​ 3、当只要一行数据时使用limit 1
​ 4、为搜索字段建立索引，一般是where子句或者order by子句，避免是%xxx%
​ 5、Join表时使用相当类型的例，将其索引
​ 6、千万不要 order by rand()
​ 7、避免select *，需要什么取什么
​ 8、永远为每张表设置一个ID，作为其主键，设置自增的标志
​ 9、使用enum而不是varchar，取值有限且固定的字段建议设置为enum
​ 10、适当时候采用垂直分隔，把不经常读取或者修改的字段、比较大的字段text，blog等字段单端放到其他表中
​ 11、拆分大的delete或者insert语句，因为这2个语句会锁表，如果操作的数据过大，其他操作就无法处理了。

myisam和innodb的区别：
1）https://cloud.tencent.com/developer/article/1181374
2）http://km.oa.com/group/27940/articles/show/365011？kmref=search&from_page=1&no=6
3）mysqldump是否要考虑2种表的不同？

mysql远程访问连接问题：
1）https://stackoverflow.com/questions/14779104/how-to-allow-remote-connection-to-mysql

mysql主从复制的解决方案：
1）（**理解主从复制**）https://www.cnblogs.com/clsn/p/8150036.html
2）主从复制一致性问题解决方案：
![a6bc18da7a3e110f6e60ec38a495ea54.png](en-resource://database/559:1)
3）主从复制操作：http://www.cnblogs.com/huchong/p/10253522.html
4）mysql异步复制、同步复制（主从都提交完事务）、半同步复制的应用和区别：mysql只实现了本地的redo-log和binlog的2PC。参考：https://cloud.tencent.com/developer/article/1005270

**binlog应用场景方案**：
1）http://km.oa.com/group/17017/articles/show/254324?kmref=search&from_page=1&no=6

mysqldump的实现原理：
0）对比测试用例：
0.1）创建一个大的数据库表，包含700w+的数据记录，表名usertb，并创建索引id（查询和修改数据快速），参考链接：https://www.fujieace.com/mysql/test-data.html
0.2）通过mysqldump的常规操作，执行mysqldump的过程中更新数据库记录，确认写操作是否会被阻塞，并查看general_log和binlog相关日志。
```
# mysqldump -uroot -p123456 --databases employees --tables usertb > mysql_dump.sql
```
0.3）通过修改mysqldump命令的方式，增加master和transaction等参数，再次执行mysqldump命令，然后同时更新数据库记录，检查数据是否会卡主。注意innodb和myisam参数不同。
```
# innodb
# mysqldump -uroot -p123456 --single-transaction --master-data=2 --databases employees --tables usertb > mysql_dump_xid.sql
# myisam
# mysqldump -uroot -p123456 --lock-tables --databases employees --tables usertb > mysql_dump_xid.sql
```
0.4）通过该方式检查go-mysql-elastic工具在全量导数据的过程中是否会卡住数据库的写操作。
0.5）如果数据库表的引擎是myisam，执行mysqldump的时候是否会持续锁表，master和transaction参数是否只适用于innodb引擎？《高性能mysql》10.2.4章节。
![f9f958e2afce1c6316b44218d21417fb.png](en-resource://database/557:1)
0.51）参考：http://www.itpub.net/thread-1739304-1-1.html
1）基本操作命令：
导出制定表：# mysqldump -uroot -proot --databases db1 --tables a1 a2 >/tmp/db1.sql
高级的用法：
```
mysqldump –uuser -p --skip-opt -q -R  --single-transaction --default-character-set=utf8 --master-data=2  --create-option --no-autocommit –S ${sock} -B ${DBName}  > backup.sql
# 参数1： --master-data=2， binlog选项，可以瞬间锁表?
# 参数2： --single-transaction，事务id选项，过滤该事务之后的事务对数据库的更新操作。
# 只针对innodb引擎有效么？
```
2）查看日志方式：在执行mysqldump命令前，在MySQL中执行set global general_log = on。
    mysql > show variables like '%general_log%'; -- 查看日志是否开启和文件位置
    mysql > set global general_log=on; -- 开启日志功能
    mysql > set global general_log_file='/tmp/mysql_general.log'
3）存在问题和规避方法：mysqldump产生的备份，最终是要结合binlog进行恢复。mysqldump也可以准确得到binlog的恢复点。
3.1）mysqldump命令通过 --master-data=2 参数，改变锁表的方式，从记录的binlog位置开始进行快照（mysql> show master status;），这样可以避免锁表？导致mysqldump的时候也可以更新数据库？然后再结合快照时binlog的位置恢复数据库的内容？在导出的sql文件中可以找到master_log_pos的内容，对应的就是快照时binlog中'show master status;'的数据。
3.2）如果有--master-data=2参数表示瞬间锁表，不会长时间锁表。
3.3）如果没有--master-data=2 参数，mysqldump操作期间会锁表，导致无法进行写操作。
4）性能瓶颈和优化方式：
5）通过mysqldump一张大表来验证执行mysqldump是否会阻塞数据库的更新操作？
5.1）参考参数：http://www.ducea.com/2006/10/26/dumping-large-mysql-innodb-tables/
5.2）**官方说明**：https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html
5.3）大数据表例子：https://stackoverflow.com/questions/12180902/where-can-i-get-a-large-sample-database-for-practising-mysql-query-optimization
5.4）数据表：http://www.ntu.edu.sg/home/ehchua/programming/sql/sampledatabases.html


binlog操作：
1）开启binlog操作：
    mysql > set global log-bin=/var/lib/mysql/mysql-bin
1.1）查看binlog原始内容：
方式1： # mysqlbinlog mysql-bin.000001 -vv
方式2：mysql> show binlog events in 'mysql-bin.000001';
          mysql> show master status\G  -- 查看当前正在写入的binlog文件
          mysql> show binary logs; -- 获取binlog文件列表
2）binlog的原理的问题：
2.1）binlog是否会循环覆盖？还是说binlog文件只会越来越大？binlog存活是根据时间配置，可以选择保留多少天之前的日志。


## **参考资料** ##    
