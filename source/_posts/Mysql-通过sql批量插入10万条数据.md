title: Mysql 通过sql批量插入10万条数据
author: MurasakiSeiFu.
date: 2018-07-10 11:42:28
tags:
---
## 简介
最近在做数据同步中间件项目，老大希望测试同步10万条数据的准确性，特此记录一下sql。
## SQL
* 开始创建一个函数

``` Mysql
delimiter ;;
create procedure myproc ()

begin
declare num int ;
set num = 1 ;
while num < 100000 do
    insert into demo_20180423 (id, value)
values
    (num, num) ;
set num = num + 1 ;
end
while ;
end;;
```
* 执行函数

``` sql
call myproc();
```
* 删除函数

``` sql
drop procedure myproc;
```

* 上图
![](https://ws1.sinaimg.cn/large/006tNc79ly1ft4woyrx3tj31gs0qk0vf.jpg)
![](https://ws4.sinaimg.cn/large/006tNc79ly1ft4wpkj39mj31gs0w0tao.jpg)

还是很方便的，就是有点点慢。