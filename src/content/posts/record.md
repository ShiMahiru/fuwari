---
category: 记录
description: 一些零散的记录，不定时更新
draft: false
image: https://www.loliapi.com/acg/pc/
published: 2025-08-27
tags:
- 记录
title: 记录
---

# Cloudflare 优选方法

### A、AAAA、CNAME

1. B域名直接套CDN指向源站
2. B域名开启SaaS，设置回退源为指向源站的域名，自定义主机名为A域名
3. B域名指向优选域名，不套CDN
4. A域名指向B域名指向的优选域名的域名 ，不套CDN

### Cloudflare Pages

1. 直接在Pages创建自定义域
2. 更改子域NS到阿里云云解析DNS
3. 在阿里云云解析DNS设置解析分流

### Cloudflare Workers

1. 直接在Workers创建路由，如：example.com/*
2. 将被设置路由的域名解析到优选域名

---

# FreeConvert文件格式转换器

https://www.freeconvert.com/zh

---

# Markdown官方教程

https://markdown.com.cn/

---

# 查询低价域名后缀
> 注册域名的时候大家肯定会到各个服务商比价，追求性价比，但是一个一个查也太麻烦了！ 所以我找到了个网站，可以一键查询低价域名，它就是哪煮米

https://www.nazhumi.com/
