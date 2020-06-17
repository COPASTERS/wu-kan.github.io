---
title: ftp动态传输前一天数据的window脚本
categories: window bat
tags:  window bat
date: 2020-06-17 11:54:23
---

# 背景

传输需要每天进行，这时的传输的文件地址就必须动态获取拼接获得，涉及时间的获取前一天。

# 脚本及注释

```bat
::参数准备
::判断当前日期的前一天
::首先直接把天数减1天
::如果出来的是0天就把月减1天，天数是当月的最后一天。
::如果出来的是0月就把年减1年，月数是当年的最后一月。
@echo off&setlocal enabledelayedexpansion
set yyyy=%date:~0,4%
set mm=%date:~5,2%
set dd=%date:~8,2%
set /a od=!dd!-1
if !od!==0 call :dd0
if !mm!==0 call :mm0
:: 所需的日期的变量
set yyyymmdd=!yyyy!!mm!!od!
:: 执行传输任务
:: 数据上传
curl --ftp-pasv -u 账号:密码 -T  "D:\data\wxcz\eee_edata_"%yyyymmdd%".csv"  ftp://199.199.299.99:80/abss/ss


:: 函数1
:dd0
set /a mm=!mm!-1
for %%a in (1 3 5 7 8 10 12)do set %%add=31
set /a pddd=!yyyy!*10/4
set pd2d=!pddd:~-1,1!
set 2dd=28
if !pd2d!==0 set 2dd=29
for %%b in (4 6 9 11)do set %%bdd=30
set od=!%mm%dd!
goto :eof
:: 函数2
:mm0
set /a yyyy=!yyyy!-1
set mm=12 && set od=31
goto :eof
```