---
layout: post
title: 解决GitHub等访问问题 
description: 解决GitHub等访问问题 
keywords: Github GitHub DNS 
categories: tools
---

国内github经常访问不到，除了付费vpn。需要去查询github的可用DNS来解决访问问题。分为两个步骤：1.查询DNS 2. 配置hosts

## 一. 利用whois 和 dig获取DNS信息
 whois github.com 查询网站IP和所有者信息   
 dig @8.8.8.8 github.com  (@8.8.8.8 时Google的公共DNS解析服务器地址)   


## 更新 etc/hosts
确认是否有编辑权限, 没有的话通过 sudo chmod a+w /etc/hosts 来添加写入权限    
之后通过vi /etc/hosts 追加 xxx.xxx.xxx.xxx github.com   
通过 source /etc/hosts 使之生效


