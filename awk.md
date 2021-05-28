---
title: 2020-4-18 awk 
tags: awk
---

## awk     
+ 检查a文件的数字是否b文件也存在（aaa内容大于bbb内容）：    
awk -F'' 'NR==FNR{a[$1]=$0}   NR>FNR{if($1 in a) {print $0 >>"in.txt"} else {print $0>"out.txt"}}'  $F1 $F2

awk 'NR==FNR {a[$1] = $1; next} {if(!($1 in a)) print $1}' aaa bbb        
awk 'FNR==NR {a[$2];next} {if (!($2 in a)) print}' lr_20200418.log wx_20200418.log   
awk -F'\"desc\":\"' '{print $2}' trans_redis.log |awk -F'",' '{print $1}'| awk -F'%2F' '{print $5}' | awk 'length($0)>10' > rds_all.txt      
比较两个文件cf2 和cf1，cf1存在于cf2的记录放在in.txt   否则放在out.txt；     
awk -F'\t' 'NR==FNR{a[$1]=$0} NR>FNR{if($1 in a){print a[$1]"\t"$0>>"in.txt"}else{print $0>>"out.txt"} }' cf2.txt cf1.txt      

    
+ awk数据统计：     
+ 捐款总额：awk -F'\t' '{if ($3 >= "2019-02-01 00:00:00" && $3 <= "2019-02-01 23:59:59") {sum = sum + $2}} END {print sum}' bless_export_20190218.txt         
+ 人次：awk -F'\t' '{if ($3 >= "2019-02-01 00:00:00" && $3 <= "2019-02-01 23:59:59") {sum = sum + 1}} END {print sum}' bless_export_20190218.txt            
+ 人数（去重）：awk -F'\t' '{if ($3 >= "2019-02-01 00:00:00" && $3 <= "2019-02-01 23:59:59") {a[$1]++}} END {print length(a)}' bless_export_20190218.txt            

+ awk常用示例：         
+ 截取匹配的文件内容：     
    awk -F'%2F' '{if(NR==FNR) A[$1]=$1;else{if($5 in A) print$0}}' pic_pattern.txt trans_mysql.log >1.txt        
+ 截取两个/之间的内容$(NF-1)表示倒数第2列:       
	cat -n 1 DARGON-IMG-SUCC-100.dat.all  | awk -F'/' '{print $(NF-1)}'         
+ 以空格为分隔符，输出第n个列的数据：           
    grep SendUserReviveSuccess * | grep uid | awk -F '|' '{print $9}'                 
+ awk统计netstat中TIME_WAIT的数量：                
    netstat -anop | awk '{a[$6]++} END {for (i in a) if(i ~ /TIME_WAIT/) print i, a[i]}'               
+ awk统计2个文件相同的数字：               
    cat a.txt b.txt > merge.txt            
    cat merge.txt | awk '{a[$1]++ } END {for (i in a) if(a[i] > 1) print i, a[i]}'             
+ awk判断时间范围是否匹配：             
    cat 99_test.txt | awk -F'\t' '{if ($12 >= "2018-09-09 23:59:00" && $12 <= "2018-09-09 23:59:59") {print $12}}'            
+ awk统计总和：                      
    awk -F'\t' '{if ($5 == "201594" && $12 >= "2018-09-07 00:00:00" && $12 <= "2018-09-09 23:59:59") {sum = sum + $10}} END {print sum}' 99_test.txt            
+ awk下面语句用于合并两个文件，对于第1列相同的那些行，输出第一个文件和第二个文件：        
    awk 'NR==FNR {a[$1] = $0; next} {if($1 in a) print $1 FS a[$1] FS $2}' file1 fiel2           
    1、NR表示从awk开始执行后，按照记录分隔符读取的数据次数，默认的记录分隔符为换行符，因此默认的就是读取的数据行数，NR可以理解为Number of Record的缩写。          
    2、在awk处理多个输入文件的时候，在处理完第一个文件后，NR并不会从1开始，而是继续累加，因此就出现了FNR，每当处理一个新文件的时候，FNR就从1开始计数，FNR可以理解为File Number  of Record。         
    3、NF表示目前的记录被分割的字段的数目，NF可以理解为Number of Field。          
     （重要！！！）NR和FNR的典型应用：https://www.cnblogs.com/irockcode/p/7044722.html              
+ awk支持shell变量 -v参数：          
    awk -F'\t' -v starttime="$starttime" -v endtime="$endtime" '{if ($3 >= starttime && $3 <= endtime) {a[$1]++}} END {print length(a)}' wx_bless_export_20190415.txt           
    
## 参考资料         
+ shell 特殊符号：https://www.cnblogs.com/xuxm2007/archive/2011/10/20/2218846.html           
+《LINUX SHELL脚本攻略 》          
+《Shell脚本学习指南》       
+《The_AWK_Programming_Language_zh_CN》         
+ awk置顶教程：https://coolshell.cn/articles/9070.html（陈皓-酷壳博客！！）        
+ awk经典书籍：https://github.com/wuzhouhui/awk/blob/master/The_AWK_Programming_Language_zh_CN.pdf（github中文翻译版！！）      
+ awk基本用法：https://www.jianshu.com/p/cae3cccd2ee6        

