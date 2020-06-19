---
title: 'Apache Http Client 使用注意事项 '
date: 2020-06-18 10:36:17
tags: Java
---
### Apache Http Client 使用注意事项 

> The typical reason to set a `SO_LINGER` timeout of zero is to avoid large numbers of connections sitting in the `TIME_WAIT` state, tying up all the available resources on a server. 
>
>  When a TCP connection is closed cleanly, the end that initiated the close ("active close") ends up with the connection sitting in `TIME_WAIT` for several minutes. So if your protocol is one where the *server* initiates the connection close, and involves very large numbers of short-lived connections, then it might be susceptible to this problem. 
>
>  This isn't a good idea, though - `TIME_WAIT` exists for a reason (to ensure that stray packets from old connections don't interfere with new connections). It's a better idea to redesign your protocol to one where the client initiates the connection close, if possible. 

<escape><!-- more --></escape>

Socket Option 

```java
RequestConfig requestConfig = RequestConfig.custom() 
                                    .setConnectionRequestTimeout(10000) 
                                    .setConnectTimeout(10000) 
                                    .setSocketTimeout(10000) 
                                    .build();
```

- setConnectTimeout: 设置与服务器建立连接从超时时间 
- setConnectionRequestTimeout:从连接池获取一个连接的等待时间
- setSocketTimeout:设置从建立连接到收到服务器响应的第一个数据包之间的时间 

Http Client 底层设置 Socket 超时时间的代码

```java
Class：DefaultHttpClientConnectionOperator 的 connectSocket 方法 
Socket sock = sf.createSocket(context); sock.setSoTimeout(socketConfig.getSoTimeout());
```

可以看到在配置中取用的socketConfig 中的soTimeOut,而不是requestConfig，所以调用 setSocketTimeout 其实是不生效的。 所以需要手动设置这个值

```java
SocketConfig socketConfig = SocketConfig.custom().setSoTimeout(3000).build(); CloseableHttpClient httpclient = HttpClients.custom().setDefaultSocketConfig(socketConfig).setRetryHandler(getHttpRequestRetryHandler()).build(); 
```

