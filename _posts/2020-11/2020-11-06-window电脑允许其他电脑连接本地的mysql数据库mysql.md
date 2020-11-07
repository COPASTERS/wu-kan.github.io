---
title: window电脑允许其他电脑连接本地的mysql数据库
categories: window mysql 
tags:  window mysql 
date: 2020-11-06 17:46:13
---

### 背景

有时候自己本地的数据库希望共享给其他小伙伴用一下，但是发现小伙伴不能连上

### 准备

- 和连接数据库的电脑在同一个局域网

- 安装mysql数据库的电脑防火墙是关闭的

### 实现

  1. 首先关闭自己电脑的防火墙

  2. 在navicat里 点击 工具 -> 命令列界面 

  新建用户 remote 密码也是remote (给其他电脑登陆的账号)

```txt
  CREATE USER 'remote'@'%' IDENTIFIEDremote BY 'remote';
```
  
  注意： 不要使用root账号作为其他电脑的登陆账号，使用的不了的，亲测不行。 user 表里的host字段要为%

  赋予remote权限  (ALL 指所有的权限) （*.* 所有数据库所用表）

```text
  GRANT ALL ON *.* TO 'remote'@'%';
```

### 连接

其他小伙伴就可以使用你创建的remote账号和你的本地地址进行登陆你的数据库了。
