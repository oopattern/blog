---
title: 2020-5-6 UUID
tags: uuid，guid
---

## **UUID实现**    
+ 版本1：MAC地址+时间戳        
+ 版本2：MAC地址+时间戳+POSIX UID/GID      
+ 版本3：MD5散列值       
+ 版本4：linux随机设备数 /dev/urandom，基于linux机器状态位，可以理解为绝对的唯一ID。      
+ 版本5：SHA-1散列值      
+ 结论：只有版本4能实现绝对的唯一id，版本1、2会出现多进程并发的问题，版本3的MD5较大可能导致相同的MD5。版本5的问题类似版本3？版本1、2默认带有自增序号，所以版本1、2只需要解决多进程的问题（可以通过增加进程id的方式实现？）      

## **分布式ID**     



## **参考**     
+ go实现版本： https://github.com/satori/go.uuid      