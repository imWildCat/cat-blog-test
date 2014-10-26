---
layout: post
title: "Rails 部署时遇到的问题（其实是 MIME type 错误）"
description: 
headline: 
date:   2014-10-13 22:14
category: Development
tags: []
image: 
  feature: 
comments: true
mathjax: 
---

这两天在学习 Rails ，昨晚第一次尝试部署，遇到一个奇葩的问题：  
CSS/JavaScript 能被加载，但是在页面上就是不渲染。尝试 Google 各种 keyword （比如 rails css not working / rails css not rendered ）都是没有符合的结果。纠结了好久。  
<!-- more -->
今天中午的时候终于发现 CSS 的 Content Type 不对，竟然是 `text/plain` ，于是去 nginx.conf 里寻错。原因是我把以前 nginx.conf 删掉了，新的配置文件内没有了 `include /etc/nginx/mime.types;` 。重新修改文件结构，重启 nginx ，**问题依旧** 。  
我以为还是没有找到问题，但是突然想到 HTTP 头内有设置缓存，于是换了个浏览器，CSS最终正确显示，问题得到解决。  

虽然是个低级的问题，但是被纠结了大半天，一开始也没有考虑到 Content Type 的问题。