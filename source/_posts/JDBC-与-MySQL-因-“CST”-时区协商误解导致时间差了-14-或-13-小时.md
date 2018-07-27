title: JDBC与MySQL 因“CST”时区协商误解导致时间差了14或13小时
author: MurasakiSeiFu.
tags: []
categories:
  - Exception
date: 2018-04-16 11:40:00
---
##### 问题描述
新项目是用的springboot+mybatis+mysql 6.0.6版本的驱动包来搭建的，在使用的过程中遇到以下2个问题

1.从mysql取的的数据日期时间，与真实的时间往后错乱了14个小时。

2.springboot jason序例日期时发现与真实的时间向前推了8小时。

第一个问题解决的方案有很多 -。-，造成这个问题原因主要是因为mysql 6.x以上版本的驱动包，连接字符串默认时区不是东八区造成的。。

1.如果你有所在库的权限，或者你们的DBA不是个懒蛋。。可以直接用sql去调~
{% codeblock  %}
SET GLOBAL time_zone = '+8:00';
SET time_zone = '+8:00';
FLUSH PRIVILEGES
{% endcodeblock %}
  	
2.在连接字符串上加上serverTimezone=Asia/Shanghai
{% codeblock  %}
jdbc:mysql://127.0.0.1:3306/test?serverTimezone=Asia/Shanghai
    
mysql 5.x的版本不存在的这个问题，所以遇到这个问题，可以选择用上面的方案解
决，也可以用mysql 5.x的版本驱动包解决。当然降版本不是好办法哦~
{% endcodeblock %}  
第二个问题应该是序列化的原因，可能springboot自身的小bug吧~
{% codeblock  %}
    在application.properties 配置文件里添序列化时区配置：spring.jackson.time-zone=GMT+8
{% endcodeblock %} 
    
这里有位大佬对原因进行了剖析，感兴趣的朋友可以看一下~
[原因](https://juejin.im/post/5902e087da2f60005df05c3d?utm_source=tuicool&utm_medium=referral)