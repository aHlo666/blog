---
layout: post
title: ThinkPHP5 SQL注入漏洞和敏感信息泄露漏洞
tags: ThinkPHP5
categories: SQL注入
---

# ThinkPHP5 SQL注入漏洞和敏感信息泄露漏洞

## 漏洞描述

传入的某参数在绑定编译指令的时候又没有安全处理，预编译的时候导致SQL异常报错。然而thinkphp5默认开启debug模式，在漏洞环境下构造错误的SQL语法会泄漏数据库账户和密码。

## 漏洞复现

首先使用GitHub中的[vulhub](!https://github.com/vulhub/vulhub.git)项目docker一键部署。
环境启动后，访问`http://your-ip/index.php?ids[]=1&ids[]=2`，可以看到显示的用户名

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/ThinkPHP/ThinkPHP5%20SQL%E6%B3%A8%E5%85%A5/15933392121615.jpg)

在ids[]中插入sql语句，页面显示了报错信息得到数据库名

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/ThinkPHP/ThinkPHP5%20SQL%E6%B3%A8%E5%85%A5/15933394620529.jpg)

同时在下方的debug页面显示了数据库的用户名密码，属于敏感信息泄露

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/ThinkPHP/ThinkPHP5%20SQL%E6%B3%A8%E5%85%A5/15933396157399.jpg)
