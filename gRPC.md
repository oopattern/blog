---
title: 2020-1-30 gRPC
tags: gRPC
renderNumberedHeading: true
grammar_cjkRuby: true
---

**gRPC适用场景**
1、集群节点少量数据之间的通信，通过接口方式同步数据。
2、通过提供服务的方式提供给不同系统之间进行通信，即微服务。

**gRPC组成**
![enter description here](./images/1580380884261.png)

**gRPC vs REST**
1、REST是基于http的请求，并返回json格式的数据。REST表示标准的方式（GET、POST、PUT、DELETE）和其他服务进行通信交互。
A RESTful API is an application program interface (API) that uses HTTP requests to GET, PUT, POST and DELETE data.
GET：从服务器获取数据。
POST：提交数据给服务器。
PUT：更新数据给服务器。
RESTful API是无状态的。
2、gRPC也是基于http的请求，并返回pb格式的数据。

**gRPC整理**
![enter description here](./images/1580393028156.png)

**FAQ**
1、gRPC通信采用PB协议，能否采用json或者xml等其他协议？
2、gRPC中client与server之间通过channel连接，该channel是长连接还是短连接？
3、SSL/TLS证书的用法和实现？
4、gRPC和http的关系？
**答：** gRPC是基于http/2协议上封装pb数据进行网络传输，不是基于原生的tcp连接封装pb数据进行数据传输。
5、gRPC和REST的关系？
**答：** gRPC没有REST通用，是一个新生的事物。REST适用于不同的系统进行通信交流。gRPC适用于同一个系统的通信交流（如集群节点的数据通信）。是否采用gRPC需要看客户端是否支持，如果存在多种客户端程序，建议使用REST。

**参考**
https://grpc.io/docs/guides/
https://developers.google.com/protocol-buffers/docs/overview