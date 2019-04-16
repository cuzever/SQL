# SQL数据库

## 1.概念及基本操作：

- 三范式：列不可拆分，唯一标识，引用主键

- 关系及存储:

    1. 1对1:关系存在A，B都可

    2. 1对多:关系存在B中

    3. 多对多:关系存入关系表中

- 登录方式：

    1. windows身份验证

    2. SQL账户密码登录

- 界面操作：

    1. 数据库：创建，删除，分离，附加，生成脚本（架构，数据）

    2. 表：创建、修改、删除

        a. 字段类型：int,bit,datetime,decimal,char/varchar/nvarchar

        b. 字符串n的区别：有n的为unicode编码，每个字符占一个字节；非unicode编码，英文或数字占一个字节，中文占两个字节

        c. 字符串var的区别：有var表示可变长度，没有var表示不可变长度，如果长度不够会补全

    3. 约束：实现数据的有效性检查

        主键，非空，唯一，默认，检查，外键：检查关系是否合理

    4. 脚本操作

        a. 不区分大小写，字符串使用单引号，某位不需要加分号

        b. 按照功能分类：DDL（数据定义语言，如创建数据库对象create、alter、drop）；DML（数据管理语言，如：增删改查insert、update、delete、select）；DCL（数据控制语言，权限分配）

        c. --单行注释，/**/多行注释

        d. 数据库：创建、删除:可以通过查看master数据库中sysdatabases表，来了解当前存在的数据库

        ```SQL
        create database 数据库名
        on primary
        (
            name = 'stuDB_data',--主数据文件的逻辑名称
            filename = 'D:\stuDB_data.mdf',--主数据文件的物理名称
            size = 5mb,--主数据文件的初始大小
            maxsize = 100mb,--主数据文件增长的最大值
            filegrowth = 15%--主数据文件的增长率
        )
        log on
        (
            name='stuDB_log',
            filename = 'D:\stuDB_log.ldf',
            size = 2mb,
            fliegrowth = 1mb
        )
        ```

        ```SQL
        select * from sysdatabases,--在master下查看所有数据库
        drop database dbtest,--删除dbtest数据库
        create database dbtest,--创建dbtest数据库
        ```

## 2. 使用脚本操作增删改查

- 基本插入insert操作：

```SQL
select * from UserInfo
-- 21232F297A57A5A743894A0E4A801FC3 admin
insert UserInfo(UserName,UserPwd)
values('小笼包','21232F297A57A5A743894A0E4A801FC3')
--为所有列，按照默认顺序赋值，可以使用如下简写
insert UserInfo
values('大头贴','21232F297A57A5A743894A0E4A801FC3')
insert UserInfo(UserName)
values('尹志平')
```

- 插入多行：

```SQL
--一次性写多个数据
select * from ClassInfo
insert into ClassInfo
values('1班'),('2班'),('3班'),('4班')
```

- 修改update操作：

```SQL
update  表名 set 列名1=值1,列名2=值2... where ...

update UserInfo set UserPwd='admin' --为指定列修改
update UserInfo set UserPwd='admin' where UserId>1 --为指定行进行修改，where行筛选
```

- 删除数据：

```SQL
delete from 表名 where ... --删除id由数据库自己维护，会出现跳号
truncate table 表名  --清空操作，from关键词可省略，通常实现：逻辑删除，物理删除，重置（重新从1开始），有外键会报错
```

- 查询操作：

    1. 为表起别名as，可省略

    2. 别名、全部列、部分列

    ```SQL
    select UserInfo.UserName Uname,UserInfo.UserPwd pwd from UserInfo

    select UserName,UserPwd from UserInfo
    ```

- 查询前n部分数据及排序：

```SQL
select top 2 percent * from StudentInfo --前2%
select top 2 * from StudentInfo --前2行
select top 2 * from StudentInfo order by cid asc(desc)，sId asc（desc） --从小到大，从大到小，先cid排再sid排
```

- 消除重复行distinct（理解什么算重复，什么不算重复）：

```SQL
select distinct cid from StudentInfo
```

- where 条件查询：返回BOOL类型，有逻辑运算（and，or，not(优先级最高)），比较运算（=，<，>，！），beetween...and...(闭区间)等

```SQL
select sname from StudentInfo where sId>5
```

- in

```SQL
select * from StudentInfo where cid in (1,3,5) --取班级编号为1、3、5学生信息

select * from StudentInfo where cid=1 or cid=3 or cid=5
```

- 模糊查询，运算符包括：like %(零到多个任意字符) _（1个任意字符） []（给定范围内给定字符） ^（非）；%_写在[]中表示本身含义

```SQL
select * from StudentInfo where sName like '%三%' --查询名字中包含三的学生信息

select * from StudentInfo where sName like '张%' --查询姓张学生

select * from StudentInfo where sName like '黄_' --查询姓名2个字并姓黄

select * from StudentInfo where sPhone like '13%' --查询电话号码13开头

select * from StudentInfo where sPhone like '1[^0-4]%' --查询第2位为非0-4

select * from StudentInfo where sEmail like '%@qq%' --查询用QQ邮箱的学生信息

select * from StudentInfo where sPhone is (not) null --电话为（非）null的学生信息
```

- 连接查询（当需要结果从多张表中取时）：join 表名 on 关联条件

    1. 内连接inner join：两表中完全匹配的数据
    2. 左、右、完全外连接 left/right/full outer join:两表中完全匹配的数据；左表中特有的数据 / 右表中特有的数据 / 左表中特有的数据，右表中特有的数据

```SQL
--查询学生姓名及所在班级名称
--StudentInfo
--ClassInfo
--关系：StudentInfo.cid=>ClassInfo.cid
--关键问题：哪些表，关系
select * from StudentInfo inner join ClassInfo on StudentInfo.cid=ClassInfo.cid
```

```SQL
--学生姓名、科目名称、分数
--StudentInfo,SubjectInfo,ScoreInfo
--sid=stuid,sid=subid
select * from ScoreInfo as score
inner join StudentInfo as stu on score.stuId=stu.sId
inner join SubjectInfo as sub on score.subId=sub.sId
```

- 聚合函数：对行数据进行合并（sum,avg,count,max,min但null不做统计计算)一般是对数字类型的列进行操作

```SQL
select count(sphone) as count1 from StudentInfo where cid=1

select MAX(scorevalue) from ScoreInfo where subId=2 --数学成绩的最高分

--求语文科目的平均分
--SubjectInfo,ScoreInfo,avg
select AVG(scoreValue) from SubjectInfo
inner join ScoreInfo on subId=SubjectInfo.sId
where sTitle='语文'
```

- 开窗函数：over()将统计出来的数据分布到原表中的每一行中；和聚合函数、排名函数一起使用

```SQL
select ScoreInfo.*,avg(scorevalue) over() from ScoreInfo where subId=1 --将学生信息显示和平均值分布到原表每一行
```

- 分组（可多列分组，根据需求先看如何分组再看需要何值）：group by 列名1，列名2

```SQL
select sGender,count(*) from StudentInfo group by sGender --统计男女生人数

select subId AVG(scorevalue) from ScoreInfo group by subId --每科平均分

select sGender,cid,count(*) from StudentInfo group by sGender,cid --多列分组

select cid,sGender,count(*) from StudentInfo where sId>2 group by cid,sGender having count(*)>3--统计学生编号大于2的各班级的各性别的学生人数大于3的信息，having在分组之后再进行筛选
```

```SQL
--完整的SQL语句
select distinct top n *
from t1 join t2 on ...
where ...
group by ... having ...
order by ...
```

- 联合查询（对结果集合并成一个结果集，列数及列类型一致）:union（并集）、union all（并+交）、except(差集)、intersect（交集）

```SQL
select cid from ClassInfo
union
select sid from StudentInfo
```

## 3.其他脚本操作

- 快速备份

    a. 向未有表备份：select 列名 into 备份表名 from 源表名 --说明：备份表可以不存在，会新建表，表的结构完全一致，但是不包含约束，如果只包含结构不包含数据，可以加top 0

    ```SQL
    select * into test1 from ClassInfo --可以复制标识，但不包含约束

    select * into test2 from ClassInfo where 1=2  --不复制数据
    ```

    b. 向已有表备份： insert into 备份表名 select 列名 from 源表名

    ```SQL
    insert into test2(cTitle) select cTitle from ClassInfo --要指定列名插入
    ```

- 字符串函数（cast,convert/ascii,char（ascii和字符互转）/left,right,substring:字符串截取,索引从1开始，而不是0/len,lower,upper：字符串长度，大小写互转/ltrim,rtrim/去空格）

```SQL
select CAST(89.00000 as decimal(4,1))--转成4位，小数后1位,CAST将任意类型转到任意类型

--CONVERT:将任意类型转到任意类型，如果目标是字符串，则style可以设置格式
```

-日期函数：getDate(获取当前日期时间)，dateAdd(日期加)，dateDiff(日期减)，datePart(取日期某部分)，注意：dateAdd,dateDiff,datePart第一个参数使用双引号

```SQL
select datepart("Dayofyear",GETDATE())
```

```SQL
--以“2015-1-1”的格式显示日期
select * from StudentInfo