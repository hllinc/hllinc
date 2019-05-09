---
title: SpringMVC整合WebSocket
date: 2019-05-09 10:45:19
tags:
---
Spring4已经加入了对Websocket支持。
1. 引入Spring4的jar包，以及Spring websocket的jar包
2. web.xml文件用3.0版本。
```xml
< web-app
      xmlns:xsi= “http://www.w3.org/2001/XMLSchema-instance”
      xmlns= “http://java.sun.com/xml/ns/javaee”
      xmlns:web= “http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd”
      xsi:schemaLocation= “http://java.sun.com/xml/ns/javaee
                     http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd”
      version= “3.0” >
```

