title: Springboot mybatis复制表结构和数据到指定表
author: MurasakiSeiFu.
date: 2018-04-23 18:18:27
tags:
---
## 简介
在项目中新增了一个功能：

    明天凌晨2点启动定时器，删除前天以前的数据，删除的数据转移到新的表中 
    名称=表名_yyyyMMdd  时间为当前时间。
看到这个功能，首先我们得知道mysql中复制表结构和数据的语句怎么写。
## 复制表结构和数据的语句
### 复制表结构及数据到新表 第一种
```SQL 
CREATE TABLE 新表 SELECT * FROM 旧表
```
这一条语句就可以搞定复制表表结构和数据到新表，但是这种方法的一个最不好的地方就是新表中没有旧表的字段属性。比如primary key、auto等。
而且，这条语句如果直接执行会报一个错误：
	
    MYSQL Statement violates GTID consistency: CREATE TABLE ... SELECT. 错误代码： 1786 问题
解决办法
```test
在 my.cnf 中将
gtid_mode = ON
enforce_gtid_consistency = ON

改为

gtid_mode = OFF
enforce_gtid_consistency = OFF
```
一般我们是没有这种DBA权限的，所以这种方法我们pass。
### 复制表结构及数据到新表 第二种
没法一条语句就过，我们就得用两条语句来分别执行。
#### 只复制表结构到新表
```SQL
create table new表 like old表
```
#### 复制旧表的数据到新表(当两个表结构一样)
```SQL
insert into new表 select * from old表 where ...
```
## mybatis
最好是能在一次调用时就把这两条语句都执行了，mybatis也为我们提供了这个方法。
首先，在jdbc连接串上我们需要加上allowMultiQueries=true
```jdbc
jdbc:mysql://127.0.0.1:3306/test?allowMultiQueries=true
```
然后在xml文件里，用“;”隔开两个sql语句就可以啦。
``` xml
    <!-- 复制表结构和数据到新表 -->
    <update id="createTable">
        create table ${mrJobTasksMonitorName} like mr_job_tasks_monitor;
        insert into ${mrJobTasksMonitorName} select * from mr_job_tasks_monitor where monitor_date &lt; #{newDate};
    </update>
```
## ${} #{}
#{} 这种取值是编译好SQL语句再取值
${} 这种是取值以后再去编译SQL语句 一般用于传入数据库对象，比如表名。
## mapper
``` Java

    /**
     * 复制表结构和数据到新表
     *
     * @param mrNodesScheduleName
     * @param newDate
     */
    void createTable(@Param("mrNodesScheduleName") String mrNodesScheduleName, 
                     @Param("newDate") String newDate);
```

核心内容就是这些了，配合定时器就可以完成这个功能了。