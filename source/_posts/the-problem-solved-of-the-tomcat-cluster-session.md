---
title: "疑难杂症处理系列 (一) —— Too many Cluster redirections 的问题"
date: 2017-10-26 00:49:13
category: 程序人生
tags: [Program,ProblemFixed]
---
疑难杂症
<!--more-->
**1、配置 Tomcat 集群 session 共享**

之前系统多节点的运行，之间的 session 共享的策略使用 nginx 的 ip hash 的策略，这种方式是配置简单，只需要在 nginx 的配置文件中添加 ip_hash 即可实现按照 IP 轮询的方式根据客户端的 IP 进行负载均衡。

但是这种方式有个弊端，就是当某些定时任务的调用，如果采用的是固定 ip 的方式，则会使得请求只落在某一台固定的节点，除非重启机器，没有达到真正的负载均衡。

进过查阅相关资料，发现可以利用 tomcat 集群的方式配置 session 共享，具体配置如下：

一、从开源组件 [tomcat-redis-session-manager](https://github.com/ran-jit/TomcatClusterRedisSessionManager) 的 git 仓库下载所需要的 jar 包：

```properties
commons-logging-1.2.jar

commons-pool2-2.4.1.jar

jedis-2.8.0.jar
```

二、将下载下来的 jar 包拷贝到各个节点的 tomcat 的 lib 目录下

三、修改 tomcat 的 conf 目录下的 ` context.xml` 文件：

![](http://p8bc1hri5.bkt.clouddn.com/the-problem-solved-of-the-tomcat-cluster-session-1.png)

完成以上之后，重启服务器，session 共享就生效了，同时可以使用 Gzip 压缩和静态文件缓存。

**2、Too many Cluster redirections 的问题**

随着接手了其他的项目，虽然也使用了这种方式达到 session 共享的目的，但是时常会有反馈说是会页面现实如下的报错信息：

![](http://p8bc1hri5.bkt.clouddn.com/the-problem-solved-of-the-tomcat-cluster-session-2.png)

一开始是在网上查阅了出现该报错的原因，也语焉不详，没说清具体什么原因，就说是集群地址配置的有问题，又加之我们使用的 tomcat 集群地址确实发生过改变（因为集群内存不够，从三主三从变成了五主），于是就改了下集群地址试了下，发现仍然没有解决，于是就开始查看源代码了。

根据错误堆栈信息，查看源代码中对应的 JedisClusterCommand 类的 runWithRetries 方法，经过分析发现该方法的作用递归调用来达到重试来获取集群连接：

```java
private T runWithRetries(byte[] key, int redirections, boolean tryRandomNode, boolean asking) {

   if (redirections <= 0) {

     throw new JedisClusterMaxRedirectionsException("Too many Cluster redirections?");

   }

   Jedis connection = null;

   try {

     if (asking) {
       // TODO: Pipeline asking with the original command to make it
       // faster....
       connection = askConnection.get();
       connection.asking();
    
       // if asking success, reset asking flag
       asking = false;
     } else {
       if (tryRandomNode) {
         connection = connectionHandler.getConnection();
       } else {
         connection = connectionHandler.getConnectionFromSlot(JedisClusterCRC16.getSlot(key));
       }
     }
    
     return execute(connection);

   } catch (JedisConnectionException jce) {

     if (tryRandomNode) {

       // maybe all connection is down

       throw jce;

     }

     // release current connection before recursion
     releaseConnection(connection);
     connection = null;
    
     // retry with random connection
     return runWithRetries(key, redirections - 1, true, asking);

   } catch (JedisRedirectionException jre) {

     // if MOVED redirection occurred,

     if (jre instanceof JedisMovedDataException) {

       // it rebuilds cluster's slot cache

       // recommended by Redis cluster specification

       this.connectionHandler.renewSlotCache(connection);

     }

     // release current connection before recursion or renewing
     releaseConnection(connection);
     connection = null;
    
     if (jre instanceof JedisAskDataException) {
       asking = true;
       askConnection.set(this.connectionHandler.getConnectionFromNode(jre.getTargetNode()));
     } else if (jre instanceof JedisMovedDataException) {
     } else {
       throw new JedisClusterException(jre);
     }
    
     return runWithRetries(key, redirections - 1, false, asking);

   } finally {

     releaseConnection(connection);

   }

 }
```

迭代异常的出口就是系统页面看到的错误，当重试次数超过预设的 redirections 的值之后，就抛错了：

![](http://p8bc1hri5.bkt.clouddn.com/the-problem-solved-of-the-tomcat-cluster-session-3.png)

但是为何会超过次数，集群地址配置有误可能会导致这个问题，如果正好检查到配置有误的地址，一旦超过次数就会如此，但是这个集群地址已经改掉了，却还是报错。

就只剩最后一项了，重新检查了下配置文件，发现有个属性 timeout 的配置为 3，很可疑，因为一般默认时间类型的配置大都是毫秒的，而假设这个属性的单位是毫秒，可能会由于网络延时，使得 3 ms 的设置超时，一旦超过那个重试次数，也会报错。
于是就把这个参数值改成了 3000，重启试了一下，果然好了！

后来问了一下架构部的同事，原来是他们当时编写参数配置文档的时候，一开始把那个 timeout 的单位搞错了，后来更新文档晚了，导致出现了这个问题，真是前人挖坑后人踩坑。。。

PS：还有一个注意的地方，此组件不支持配置session永不过期的项目（web.xml中有配置session-timeout=0），就是要使得配置生效，必须配置 session-timeout 的值不能为0
