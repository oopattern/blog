---
title: 2020-7-10 login 
tags: token, session
---

## **背景**     
+ 背景：    
	1）HTTP每个请求无状态（服务器记不住你），cookie和session用于关联多个http请求？      
	2）安全问题，校验是否为用户本人操作？    
	3）利用cookie和session保存登录信息，下次自动登录？      

## **cookie**  
+ cookie 的作用就好比服务器给你贴个标签，然后你每次向服务器再发请求时，服务器就能够 cookie 认出你。       
+ 保存在浏览器客户端中（ cookie 字段是存储在 HTTP header ），类似name=value的映射。          
+ 有过期时间。       
+ 和session搭配来跟踪会话，服务器生成session映射会存储在客户端的cookie中。
+ session搭配cookie可以减少网络带宽？           

## **session**    
+ 文本文件形式存储在服务端，保证了安全问题。    
+ 会消耗服务器资源，有过期时间。     
+ 和客户端浏览器绑定映射关系。      
+ session 是一个数据结构，由网站的开发者设计，所以可以承载各种数据。     

## **token**    
+ 服务器生成。     
+ 安全性更高，每个请求有签名（uid+时间戳+签名），可以保障重放攻击。
+ tokens can be exchanged with another client) 这个怎么理解？    
+ 生成token的方式：    
   1）hash：token = user_id|expiry_date|HMAC(user_id|expiry_date, k) ，k只有server知道。            
   2）AES：token = AES(user_id|expiry_date, x)，x代表加密/解密秘钥。     
   3）RSA：token = RSA(user_id|expiry_date, private key)       

## **参考资料** ##       
+ 基础知识： https://medium.com/@sherryhsu/session-vs-token-based-authentication-11a6c5ac45e4            
+ session和token对比：https://security.stackexchange.com/questions/81756/session-authentication-vs-token-authentication          
+ 耗子哥的干货总结：https://coolshell.cn/articles/19395.html      


