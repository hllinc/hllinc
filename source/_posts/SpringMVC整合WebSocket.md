---
title: SpringMVC整合WebSocket
date: 2015-10-13 10:45:19
tags: Java
---
Spring4已经加入了对Websocket支持。
- 引入Spring4的jar包，以及Spring websocket的jar包
- web.xml文件用3.0版本。
```xml
< web-app
      xmlns:xsi= “http://www.w3.org/2001/XMLSchema-instance”
      xmlns= “http://java.sun.com/xml/ns/javaee”
      xmlns:web= “http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd”
      xsi:schemaLocation= “http://java.sun.com/xml/ns/javaee
                     http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd”
      version= “3.0” >
```
- Tomcat用7或以上，JDK用7或以上，其他Spring配置文件按默认即可，这里用的是注解方式。
- WebSocketConfig.java
```java
package com.noter.websocket.config;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;
import com.noter.websocket.handler.SystemWebSocketHandler;
@Configuration
@EnableWebMvc
@EnableWebSocket
public class WebSocketConfig extends WebMvcConfigurerAdapter implements
           WebSocketConfigurer {
      @Override
      public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
           registry.addHandler(systemWebSocketHandler(), “/webSocketServer.do” );
     }
      @Bean
      public WebSocketHandler systemWebSocketHandler() {
            return new SystemWebSocketHandler();
     }
}
```
- SystemWebSocketHandler.java
```java
package com.noter.websocket.handler;
import java.io.IOException;
import java.util.ArrayList;
import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.WebSocketMessage;
import org.springframework.web.socket.WebSocketSession;
public class SystemWebSocketHandler implements WebSocketHandler {
    private static final ArrayList<WebSocketSession> users = new ArrayList<WebSocketSession>();;
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
     System. out.println( “ConnectionEstablished” );
        users.add(session);
    }
    @Override
    public void handleMessage(WebSocketSession session, WebSocketMessage<?> message) throws Exception {
     System. out.println( “session id:” + session.getId());
     sendMessageToUsers(session.getId(),(TextMessage) message);
    }
    @Override
    public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
        if(session.isOpen()){
            session.close();
        }
        users.remove(session);
    }
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception {
        users.remove(session);
    }
    @Override
    public boolean supportsPartialMessages() {
        return false;
    }
    /**
     * 给所有在线用户发送消息
     *
     * @param message
     */
    public void sendMessageToUsers(String excep, TextMessage message) {
        for (WebSocketSession user : users) {
            try {
                if (user.isOpen()&& !user.getId().equals(excep)) {
                    user.sendMessage(message);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
 
}
```
- 前台JSP代码：
```
<%@ page language = “java” contentType= “text/html; charset=UTF-8”
      pageEncoding= “UTF-8” %>
<%
     String path = request.getContextPath();
     String basePath = request.getScheme() + “://”
     + request.getServerName() + “:” + request.getServerPort()
     + path + “/”;
%>
<! DOCTYPE html PUBLIC “-//W3C//DTD HTML 4.01 Transitional//EN” “http://www.w3.org/TR/html4/loose.dtd” >
< html>
< head>
< meta http-equiv= “Content-Type” content = “text/html; charset=UTF-8”>
< title> WebSocket/SockJS Echo Sample (Adapted from Tomcat’s echo sample)</ title>
    <style type = “text/css”>
        #connect-container {
            float: left;
            width: 400px
        }
        #connect-container div {
            padding: 5px;
        }
        #console-container {
            float: left;
            margin-left: 15px;
            width: 400px;
        }
        #console {
            border: 1px solid #CCCCCC ;
            border-right-color: #999999;
            border-bottom-color: #999999;
            height: 170px;
            overflow-y: scroll;
            padding: 5px;
            width: 100%;
        }
        #console p {
            padding: 0;
            margin: 0;
        }
    </style >
    <script src = “../assets/js/sockjs-0.3.min.js”></ script >
    <script type = “text/javascript”>
        var ws = null;
        var url = null;
        var transports = [];
       
        var basePath = “ <%= basePath%> “;
        var wsPath = basePath.replace( “http://”, “ws://” );
 
        function setConnected(connected) {
            document.getElementById( ‘connect’ ).disabled = connected;
            document.getElementById( ‘disconnect’ ).disabled = !connected;
            document.getElementById( ‘echo’ ).disabled = !connected;
        }
        function connect() {
            if (!url) {
                alert( ‘Select whether to use W3C WebSocket or SockJS’);
                return ;
            }
            ws = new WebSocket(wsPath+‘webSocketServer.do’ ); /* (url.indexOf(‘sockjs’) != -1) ?
                new SockJS(url, undefined, {protocols_whitelist: transports}) :  */
            ws.onopen = function () {
                setConnected( true );
                log( ‘Info: connection opened.’ );
            };
            ws.onmessage = function (event) {
                 for ( var i in event.originalTarget){
                     console.log( “event[“ +i+“]:” +event.originalTarget[i]);
                }
                log( ‘Received: ‘ + event.data);
            };
           
            ws.onclose = function (event) {
                setConnected( false );
                log( ‘Info: connection closed.’ );
                log(event);
            };
        }
 
        function disconnect() {
            if (ws != null) {
                ws.close();
                ws = null ;
            }
            setConnected( false );
        }
 
        function echo() {
            if (ws != null) {
                var message = document.getElementById(‘message’ ).value;
                log( ‘Sent: ‘ + message);
                ws.send(message);
            } else {
                alert( ‘connection not established, please connect.’);
            }
        }
 
        function updateUrl(urlPath) {
            if (urlPath.indexOf(‘sockjs’ ) != -1) {
                url = urlPath;
                document.getElementById(‘sockJsTransportSelect’ ).style.visibility = ‘visible’;
            }
            else {
              if (window.location.protocol == ‘http:’) {
                  url = ‘ws://’ + window.location.host + urlPath;
              } else {
                  url = ‘wss://’ + window.location.host + urlPath;
              }
              document.getElementById(‘sockJsTransportSelect’ ).style.visibility = ‘hidden’;
            }
        }
 
        function updateTransport(transport) {
          transports = (transport == ‘all’ ) ?  [] : [transport];
        }
       
        function log(message) {
            var console = document.getElementById( ‘console’);
            var p = document.createElement( ‘p’);
            p.style.wordWrap = ‘break-word’ ;
            p.appendChild(document.createTextNode(message));
            console.appendChild(p);
            while (console.childNodes.length > 25) {
                console.removeChild(console.firstChild);
            }
            console.scrollTop = console.scrollHeight;
        }
    </script >
</ head>
< body>
< noscript>< h2 style=” color: #ff0000“ >Seems your browser doesn’t support Javascript! Websockets
    rely on Javascript being enabled. Please enable
    Javascript and reload this page!</ h2></ noscript >
< div>
    <div id = “connect-container”>
        <input id = “radio1” type= “radio” name= “group1” onclick =“updateUrl(‘ ${contextPath} /webSocketServer’);”>
            <label for = “radio1”> W3C WebSocket</ label >
        <br >
        <input id = “radio2” type= “radio” name= “group1” onclick =“updateUrl(‘/spring-websocket-test/sockjs/echo’);” >
            <label for = “radio2”> SockJS</ label >
        <div id = “sockJsTransportSelect” style=” visibility: hidden;” >
            <span >SockJS transport: </ span>
            <select onchange =“updateTransport(this.value)” >
              <option value = “all”> all</ option >
              <option value = “websocket”> websocket</ option >
              <option value = “xhr-polling”> xhr-polling </option >
              <option value= “jsonp-polling” >jsonp-polling </ option>
              <option value= “xhr-streaming” >xhr-streaming </ option>
              <option value= “iframe-eventsource” >iframe– eventsource</ option >
              <option value= “iframe-htmlfile” >iframe– htmlfile</ option >
            </ select>
        </div >
        <div >
            <button id = “connect” onclick =“connect();” >Connect </ button>
            <button id = “disconnect” disabled= “disabled” onclick =“disconnect();” >Disconnect </ button>
        </div >
        <div >
            <textarea id = “message” style=” width: 350px“ >Here is a message!</ textarea>
        </div >
        <div >
            <button id = “echo” onclick= “echo();” disabled =“disabled” >Echo message </ button>
        </div >
    </div >
    <div id = “console-container”>
        <div id = “console”></ div>
    </div>
</ div>
</ body>
</ html>
```
<font color="red">注：最后只要注意你定义的springmvc拦截的路径规则和请求路径即可，我一开始总是报404错误，就是路径写错了。</font>