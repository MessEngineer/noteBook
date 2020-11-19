# Tomcat处理请求过程

Tomcat作为网络编程框架，在Tomcat 7版本迎来一个分水岭，开始支持nio，但是默认还是BIO，Tomcat 8以后都是默认nio，Tomcat 7 server.xml需要增加以下配置开启nio

```xml
<!--默认的BIO-->
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
<!--配置protocol后io模型切换为nio -->
<Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol"
               connectionTimeout="20000" redirectPort="8443"/>
```

## socket Api溯源

为什么说socket(只讨论Tcp)是网络编程的最小操作单元，通过整个socket的调用链，可以理解为os层把Tcp层该做的都做了，三次握手建立连接，暴露给应用层的就是对外的socket接口，对于os来说，Tomcat算应用，Jvm也算应用，创建Tcp连接只能由各个操作系统实现，对外统一提供的就是socket接口，具体可以查看例如linux的socket.c文件。对于java编程来说，只到了native这一层，jni通过native方法对应调用openjdk中的C或者C++方法，openjdk中最后会调用到对应os的api

### socket各个api的调用栈

- 客户端创建socket

  ```java
  socket(host, port) : TCPClient.java
      |---this.createImpl(stream) : socket.class
          |---this.impl.create(stream) : socket.class
              |---protected synchronized void create(boolean stream) : AbstractPlainSocketImpl.class
                  |---this.socketCreate() : AbstractPlainSocketImpl.class
                      |---native void socketCreate(boolean var1) : PlainSocketImpl.class
                                           ------JVM------
                          |---Java_java_net_PlainSocketImpl_socketCreate : PlainSocketImol.c
                              |---fd=JVM_Socket(domain, type, 0) : PlainSocketImol.c
                                  |---os::socket(domain, type, protocol) : jvm.cpp
  ```

  

- 客户端bind

- 客户端connect

- 

