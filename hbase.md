---
title: 2020-7-8 hbase
tags: hbase
---

2）Hbase row-key 设计

3）hbase命令示例：         
3.1）常用命令： https://my.oschina.net/gently/blog/674386coun             
							https://learnhbase.wordpress.com/2013/03/02/hbase-shell-commands/            
3.2）shell命令示例：      
hadoop@master:/mysoftware/hbase-1.2.1$ bin/hbase shell            
hbase(main):013:0> list           
hbase(main):013:0> create_namespace 'volunteer'           
hbase(main):013:0> create 'volunteer:vol_task',{NAME => 'cf1', VERSIONS => 1}                
hbase(main):013:0> create "tablename", {VERSION=>1,NAME=>"cf1"}          
hbase(main):014:0> describe 'userinfo'            
hbase(main):012:0> count 'userinfo'             
hbase(main):012:0> deleteall 'company:company_activity_list', '0000505365ecard38454013899HHUE1KWRCTT'              
hbase(main):027:0* get 'gy_trans', '0026201820180803150803150026'                  
hbase(main):027:0* scan 'gongyi_user:gy_user_summary', {LIMIT=>1}           
hbase(main):027:0* scan 'company:company_activity_list', {LIMIT=>1}                 
hbase(main):027:0* scan 'gy_process:month_process', {LIMIT=>1, STARTROW => '0000505365ecard'}               
hbase(main):027:0* scan 'gongyi_user:gy_user_summary', {COLUMNS => 'cf1:money', LIMIT=>5}             
hbase(main):050:0* scan 'gongyi_user:gy_user_summary', {COLUMNS => 'cf1:money:toLong', LIMIT=>5}             
hbase(main):050:0* hbase> put 't1', 'rk1', 'f1:col1', 'value1'             
hbase(main):050:0* scan 'bless:bless_flow', {COLUMNS => 'cf1:code', STARTROW => 'sSeaRaRACj7fXWF6ueci77jJorpo', LIMIT=>2}          
hbase(main):050:0* get 'bless:bless_flow', 'sSeaRaRACj7fXWF6ueci77jJorpo'            

2.1）导入导出数据：          
查看hdfs文件系统：hadoop fs -ls hdfs://gytest/user/gongyi/ecard_sakulali_20190111_yyy.txt           
注意：导入数据到hbase表只能针对空表，如果导入数据后删除或修改表数据，再次尝试导入数据会出现问题，无法再次导入数据。             

导出数据：        
echo "scan 'company:company_activity_list', {LIMIT=>1, STARTROW => '0000505365ecard'}" | /data/hbase/bin/hbase shell >           /data/gongyi/company_sakulali_20190107.txt           

2.1.1）hbase导出数据到hdfs文件系统：         
/data/hbase/bin/hbase org.apache.hadoop.hbase.mapreduce.Export -Dhbase.mapreduce.scan.row.start="01201807122548622482" -Dhbase.mapreduce.scan.row.stop="062018071226114891240001" "ecard:ecard_activity_info" "ecard_sakulali_20190111_yyy.txt"           
2.1.2）hdfs文件系统导入本地文件系统        
hadoop fs -get hdfs://gytest/user/gongyi/ecard_sakulali_20190111_yyy.txt ~/sakulali_xxx           
2.1.3）本地文件系统导入hdfs文件系统           
hadoop fs -put ~/sakulali_xxx hdfs://gytest/user/gongyi/sakulali_xxx            
2.1.4）hdfs文件系统导入数据hbase：           
/data/hbase/bin/hbase org.apache.hadoop.hbase.mapreduce.Import "gy_process:month_process" hdfs://gytest/user/gongyi/ecard_lidi_1542        
2.1.5）import问题：       
https://stackoverflow.com/questions/24407827/hbase-import-command             

Exception in thread "main" org.apache.hadoop.mapred.FileAlreadyExistsException: Output directory hdfs://gytest/user/gongyi/ecard_sakulali_20190111_yyy.txt already exists           

hbase org.apache.hadoop.hbase.mapreduce.Export -Dhbase.mapreduce.scan.row.start=0 -Dhbase.mapreduce.scan.row.stop=6 "mytable" "/export/mytable"             
https://blogs.msdn.microsoft.com/data_otaku/2016/12/21/working-with-the-hbase-import-and-export-utility/

2.2）hdfs命令（hadoop shell命令）：              
https://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html             

2.3）导出命令              
https://blog.csdn.net/LiuY_ang/article/details/78303785             

2.4）scan过滤条件：             
https://issues.apache.org/jira/browse/HBASE-14925（重要！）             
https://social.msdn.microsoft.com/Forums/azure/en-US/589412c6-532c-4be8-a8be-4db718f9a2ec/hbase-scan-commands-into-a-text-file-in-windows?forum=hdinsight            
http://blog.cloudera.com/blog/2016/01/how-to-create-and-use-a-custom-formatter-in-the-apache-hbase-shell/             
https://stackoverflow.com/questions/10035475/get-output-from-scans-in-hbase-shell           
https://stackoverflow.com/questions/11013197/scan-htable-rows-for-specific-column-value-using-hbase-shell             
https://www.cloudera.com/documentation/enterprise/5-9-x/topics/admin_hbase_filtering.html               
https://blog.csdn.net/cnweike/article/details/42920547             
https://learnhbase.wordpress.com/2013/03/02/hbase-shell-commands/             
4.1）示例：           
ValueFilter (!=, ‘binary:Value’)            
FILTER => "ValueFilter( =, 'binaryprefix:value')            

5）hbase在SecureCRT不能回退的问题：             
http://blog.51cto.com/10120275/1637637       

查询hbase行数：          
通过 $HBASE_HOME/bin/hbase 命令执行：         
[root@cdh1 ~]# hbase org.apache.hadoop.hbase.mapreduce.RowCounter 'sda_crm_calls20180102'               

## **参考资料** ##      
