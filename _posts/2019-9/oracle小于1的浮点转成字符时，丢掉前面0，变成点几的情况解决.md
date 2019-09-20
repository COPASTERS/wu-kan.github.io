---
title: oracle小于1的浮点转成字符时，丢掉前面0，变成点几的情况解决
categories: oracle
tags: oracle
date: 2019-09-20 17:06:12
---

问题出现：

       select round(1/300*100,4)||'%' from dual;

结果是： .3333%

现象： 个位0丢失

原因：  oracle小于1的浮点转成字符时，丢掉前面0

解决： 

     select concat(to_char(round(1/300*100,4),'fm990.9999'),'%') from dual;

fm表示去掉空格和后面的占位0

0表示在对应位置返回对应的字符,如果没有则以'0'填充.

9 表示在小数位,则表示转换为对应字符,如果没有则以0表示;在整数位,没有对应则不填充字符.

```app
select to_char(.516,'fm999999990.9999999') from dual t; -- 0.516
select to_char(.516,'999999990.9999999') from dual t;   --          0.5160000   前面有空格，fm的作用就是去掉前面空格和后面的占位0
select to_char(.516,'fm999999990.000000') from dual t;  -- 0.516000
 
SELECT to_char(2,'fm00') FROM dual;                     -- 02
SELECT to_char(2,'fm9990.000000') FROM dual;            -- 2.000000
SELECT to_char(2,'fm9990.999999') FROM dual;            -- 2.      fm去掉前面的空格，同时还去掉了小数位的占位0
SELECT to_char(2,'9990.999999') FROM dual;              --      2.000000 
SELECT trim(to_char(2,'9990.999999')) FROM dual;        -- 2.000000 trim可以去掉前面的空格，并且保持勒后面的占位，避免 2. 没有 小数位0的情况
```