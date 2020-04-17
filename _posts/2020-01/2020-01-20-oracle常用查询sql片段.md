---
title: oracle常用查询sql片段
categories: oracle
tags: oracle
date: 2020-01-20 09:33:33
---

# sql片段：

## 日期

###   TO_CHAR

- TO_CHAR(SYSDATE - 1, 'yyyymmdd') TJRQ  --- 日日期
- TO_CHAR(T.OPENTIME,'yyyymmdd') BETWEEN '20190926' AND '20191002'
- TO_CHAR(gztime, 'yyyy-mm-dd hh24:mi:ss') 关注时间,  -- 日期转字符
- TO_CHAR(ADDTIME, 'YYYY-MM-DD') BETWEEN '2019-09-01' AND '2019-09-30'
- TO_CHAR(SYSDATE-7,'yyyymmdd')||'-'||TO_CHAR(SYSDATE-1,'yyyymmdd') TJRQ   --周日期
- TO_CHAR(ADD_MONTHS(SYSDATE,-1),'yyyymm') TJRQ   --月日期
- TO_CHAR(TO_DATE(OPENTIME,'yyyy/mm/dd hh24:mi:ss'),'yyyy/mm/dd hh24:mi:ss') 开户时间

### TO_DATE

- t1.ADDTIME BETWEEN TO_DATE('2019-10-16 00:00:00', 'yyyy-mm-dd hh24:mi:ss') AND TO_DATE('2019-10-16 23:59:59', 'yyyy-mm-dd hh24:mi:ss')

### TRUNC

- OPENTIME BETWEEN TRUNC(SYSDATE-7) AND TRUNC(SYSDATE-1)+0.99999; --周
- TRUNC(OPENTIME,'mm') = TRUNC(ADD_MONTHS(SYSDATE,-1),'mm'); --月
- A.CREATETIME BETWEEN TRUNC((ADD_MONTHS(SYSDATE,-1)),'month') AND 
TRUNC(SYSDATE,'month')  --在一个月之间
- sendtime  BETWEEN TRUNC((ADD_MONTHS(SYSDATE,-4)),'month') AND  TRUNC((ADD_MONTHS(SYSDATE,-3)),'month')  --指定间隔月数

## 其他

- T2.ADDTIME BETWEEN T1.OPENTIME  AND  T1.OPENTIME + 20/(24*60)  -- 时间在20分钟后 

## 清除与新建表

```sql
  DROP TABLE CGS_WX_USER_INFO_TEMP1 PURGE;
  CREATE TABLE CGS_WX_USER_INFO_TEMP1 NOLOGGING AS （SELECT.......）
```

## 创建与插入表

方式一：
```sql
--CREATE TABLE  CGS_WX_MP_USER_INFO_XY  NOLOGGING AS
INSERT INTO CGS_WX_MP_USER_INFO_XY
SELECT * FROM CGS_WX_MP_USER_INFO_XY_TEMP1;
Commit;
```
方式二：

```sql
ALTER TABLE cgs_service_channel_qd1 NOLOGGING;
INSERT /*+append*/ INTO cgs_service_channel_qd1 
SELECT  * FROM ....................
COMMIT;
```

## 改变表增加字段及更新

```sql
-- ALTER TABLE CGS_WX_USE_INFO_TEMP ADD(ZHYWGLS NUMBER);
ALTER TABLE CGS_WX_USER_INFO_JZMON_TMP ADD(SCXZHY VARCHAR2(10));
UPDATE CGS_WX_USE_INFO_TEMP A SET ZHYWGLS = (SELECT ZHYWGLS FROM  CGS_WX_USER__INFO_TEMP1 B  WHERE B.CITYNAME = 'blank')  
WHERE A.DCITYNAMED = 'blank' ; 
COMMIT; 
```

## 清除服务量为空的数据

```sql
DELETE FROM CGS__WX_SER_HYFWL T WHERE NVL(T.FWL,0)=0 and TJRQ = TO_CHAR(SYSDATE-1,'yyyymmdd');
COMMIT;
```
## 清除blank数据
```
DELETE FROM CGS__WX_SER_HYFWL  WHERE CITY = 'blank' AND TJRQ = TO_CHAR(SYSDATE-1,'yyyymmdd');
COMMIT;
```

## CASE WHEN

```sql
CASE WHEN OPENPAGE LIKE 'findps-kd%' THEN '找回密码'
    WHEN OPENPAGE LIKE 'findnetno-kd%' THEN '找回账号'
    WHEN OPENPAGE LIKE 'findnetno-itv%' THEN '找回I帐号'
    WHEN OPENPAGE LIKE 'findps-itv%'    THEN '找回I号密码'   
END I_TYPE,
```

## 多表联合判断是更新还是插入

```sql
merge into CGS_WX_MP_USER_INFO_SDLH_TEMP5 t5 using CGS_WX_MP_USER_INFO_SDLH_EMP20 t20 on (T5.OPENPAGE = T20.OPENPAGE AND T5.MYCITY = T20.MYCITY )
WHEN MATCHED THEN
    UPDATE SET t5.scgl = t20.scgl
WHEN NOT MATCHED THEN
      INSERT  (T5.TJRQ , T5.OPENPAGE,T5.MYCITY,T5.ZXGZ,T5.SCGZ,T5.ZXGL,T5.SCGL) VALUES (T20.TJRQ , T20.OPENPAGE,T20.MYCITY,T20.ZXGZ,T20.SCGZ,T20.ZXGL,T20.SCGL);
COMMIT;
```

## 去重；
单独
```
SELECT * FROM （
SELECT
 T .ADD_TIME,
 T .CHANGE_TYPE,
 T .CONTRACT_EXPIRE_TIME,
 T .CONTRACT_ID,
 ROW_NUMBER() OVER(PARTITION BY T .CONTRACT_ID  ORDER BY T .CONTRACT_ID) AS MYNUM
FROM
  T_WX_MP_BREAK_PARAM T
WHERE
  T .ADD_TIME 
BETWEEN 
TO_DATE (
    '2019-12-12 00:00:00',
    'yyyy-MM-dd hh24:mi:ss'
  ) AND 
TO_DATE (
  '2019-12-18 23:59:59',
  'yyyy-MM-dd hh24:mi:ss'
) 
）WHERE MYNUM = 1;
```
多个
```
SELECT DISTINCT USERNO,SENDTIME,PROCESSTYPE FROM CGS_WX_SERS_FWL_TEMP8 A
WHERE  A.ROWID = ( SELECT MAX(B.ROWID) FROM CGS_WX_SERS_FWL_TEMP8 B WHERE A.USERNO = B.USERNO );
```
## 加索引
- CREATE INDEX WEIXINNO2_INDEX ON CGS_WX_BS_TEMP2(WEIXINNO);

## 报表开头

```sql
SELECT '实名友统计（月报）----结束' FROM dual;
SELECT TO_CHAR(SYSDATE,'yyyy-mm-dd hh24:mi:ss') FROM dual;
```
## 去除字符串里的换行和空格

- REPLACE(REPLACE(T.MSG,CHR(13),''),CHR(10),'')

<html>
CHR(10)表示换行 CHR(13)表示回车 //ASCII 码
EG: SELECT CHR(44) 符号  FROM DUAL;<br>
换行(\n)就是光标下移一行却不会移到这一行的开头,回车(\r)就是回到当前行的开头却不向下移一行. Enter键按下后会执行\n\r这样就是我们看到的一般意义的回车了,所以你用16进制文件查看方式看一个文本,就会在行尾发现"\n\r"
</html>

## ROWID

```sql
--ROWID最后一位的字母越靠前，ROWID就越小
--dbms_rowid.rowid_row_number(rowid)的作用就是解析出ROWID的行号
SELECT dbms_rowid.rowid_row_number(ROWID) n,ROWID,t.func FROM cgs_wx_sers_fwl_temp6 t;

--用rowid方法
--据据oracle带的rowid属性，进行判断，是否存在重复,语句如下：
--查数据:
SELECT * FROM table1 A WHERE ROWID
!=(SELECT MAX(ROWID)
FROM table1 b WHERE A.name1=b.name1 AND
A.name2=b.name2......)
--删数据：
DELETE FROM table1 A WHERE ROWID
!=(SELECT MAX(ROWID)
FROM table1 b WHERE A.name1=b.name1 AND
A.name2=b.name2......)
2.GROUP by方法
--查数据:
SELECT count(num), MAX(NAME) FROM student --列出重复的记录数，并列出他的name属性
GROUP BY num
HAVING count(num) >1 --按num分组后找出表中num列重复，即出现次数大于一次
--删数据：
DELETE FROM student
GROUP BY num
HAVING count(num) >1
--这样的话就把所有重复的都删除了。
--3.用distinct方法 -对于小的表比较有用
CREATE TABLE table_new AS SELECT DISTINCT *
FROM table1 minux
TRUNCATE TABLE table1;
INSERT INTO table1 SELECT * FROM table_new;
```

## 使用UNPIVOT和PIVOT函数实现“列转行”和“行转列“

```sql
--创建测试表

create table email_signup(
user_account varchar2(100),
signup_date date,
user_email varchar2(100),
friend1_email varchar2(100),
friend2_email varchar2(100),
friend3_email varchar2(100)
);

insert into email_signup
values
  ('Rjbryla',
   to_date('2009-08-21', 'yyyy-mm-dd'),
   'rjbryla@example.com',
   'rjbdba@example.com',
   'pensivepenman@example.com',
   'unclebob@example.com');

insert into email_signup
values
  ('johndoe',
   to_date('2009-08-22', 'yyyy-mm-dd'),
   'janedoe@example.com',
   null,
   'dog@example.com',
   null);
commit;
--查询表中的内容

SELECT
	* 
FROM
	EMAIL_SIGNUP;
--需求：需要将USER_EMAIL、FRIEND1_EMAIL、FRIEND2_EMAIL、FRIEND3_EMAIL都转到一行上
/*实现的格式如下：
USER_ACCOUNT | SIGNUP_DATE | EMAIL_ADDRESS
*/--使用UNPIVOT函数实现列转行
SELECT
	USER_ACCOUNT,
	SIGNUP_DATE,
	SRC_COL_NAME,
	FRIEND_EMAIL 
FROM
	EMAIL_SIGNUP UNPIVOT ( ( FRIEND_EMAIL ) FOR SRC_COL_NAME IN ( USER_EMAIL, FRIEND1_EMAIL, FRIEND2_EMAIL, FRIEND3_EMAIL ) );

-- 行转列
create table tmp as select * from (
select '张三' student,'语文' course ,78 score from dual union all 
select '张三','数学',87 from dual union all 
select '张三','英语',82 from dual union all 
select '张三','物理',90 from dual union all 
select '李四','语文',65 from dual union all 
select '李四','数学',77 from dual union all 
select '李四','英语',65 from dual union all 
select '李四','物理',85 from dual);

select * from tmp;

select 
  student, 
  max(decode(course, '语文', score)) 语文, 
  max(decode(course, '数学', score)) 数学, 
  max(decode(course, '英语', score)) 英语, 
  max(decode(course, '物理', score)) 物理, 
  sum(score) total 
from tmp
group by student;
-----------------------------------------
select 
  student,
  max(case when course = '语文' then score end) 语文, 
  max(case when course = '数学' then score end) 数学, 
  max(case when course = '英语' then score end) 英语, 
  max(case when course = '物理' then score end) 物理, 
  sum(score) total 
from tmp
  group by student;


select * from tmp;
-- pivot 行转列
-- create table tmp2 as 
select *  from tmp pivot ( max(score) for course in ('语文' as 语文 , '数学' as 数学, '英语' as 英语,'物理' as 物理) )  ;

select * from tmp2;
-- unpivot 列转行
select student,科目,成绩 from tmp2 unpivot (成绩 for 科目 in (语文, 数学, 英语, 物理));

```

## 函数

```sql
DECODE(CITYNAME,null,'blank',CITYNAME)   判断 是null吗 cname=null 取blank 反之为本身原始数据

TO_CHAR(ADD_MONTHS(SYSDATE,-1),'yyyymm')    to_char函数的功能是将数值型或者日期型转化为字符型

ADD_MONTHS(SYSDATE,-1)      这个函数用于计算在时间x之上机上Y个月后的时间值，要是Y的值为负数的话就是在，这个时间点之间的时间值（这个时间-Y个月）。

trunc(sysdate,'yyyy') --返回当年第一天.
trunc(sysdate,'mm') --返回当月第一天.
trunc(sysdate,'d') --返回当前星期的第一天.

用rowid查找重复列原理
 select rowid from t a where a.rowid <> (select min(b.rowid) from t b where a.x=b.x)
假设表的数据是这样的
rowid X
1 a
2 b
3 b
4 c

第2，3条数据重复，但是rowid不一样，连接后结果，这两条记录是重复的
a.rowid b.rowid a.x b.x
2 2 b b
2 3 b b
3 2 b b
3 3 b b
那么最小的rowid的那条记录是第一条，你的sql语句的结果会将除去第一条的所有重复记录显示出来了

```

































