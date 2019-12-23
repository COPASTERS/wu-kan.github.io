---
title: navicat导出csv文件使用微软office打开乱码
categories: oracle
tags: oracle
date: 2019-12-23 17:41:02
---

## 问题描述：

在使用navicat导出csv文件后，wps打开没有出现乱码，但是传给同事后打开乱码，询问后得知他使用的是微软的excel打开，今天我换成微软的excel打开也是乱码。

## 分析：

应该是微软的excel不自动转换编码吧！！

## 解决：

使用电脑的记事本打开导出的csv文件，此时文件是不乱码的，然后点 文件——>另存为： 编码选 ANSI ——>保存,最后excel打开就可以了。

如图：

![2019-12-23 17:41:02](https://ae01.alicdn.com/kf/He7c6b5064b504087854e1b90ba7281084.png)