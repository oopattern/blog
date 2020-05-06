---
title: 2020-5-6 UUID
tags: uuid，guid
---

## **UUID实现**    
+ 版本1：MAC地址+时间戳        
+ 版本2：MAC地址+时间戳+POSIX UID/GID      
+ 版本3：MD5散列值       
+ 版本4：linux随机设备数 /dev/urandom      
+ 版本5：SHA-1散列值      


## **参考**     
+ go实现版本： https://github.com/satori/go.uuid      