title: springboot 根据角色权限生成菜单树
author: MurasakiSeiFu.
tags: []
categories: []
date: 2018-04-20 14:03:00
---
在之前我们已经完成了通过递归的方式生成菜单树：[递归生成菜单树](/2018/04/18/springboot-登录技巧/),然而在登录功能中，不可能所有用户的菜单权限都一样，因此在这基础上，我们要完成权限菜单的功能。
## 准备数据
我们需要两张表，权限表和权限菜单关联表。加上之前的菜单表，现在我们已经有3张表了。
### 角色表
``` SQL
CREATE TABLE `c_role` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `role_code` VARCHAR(100) DEFAULT NULL COMMENT '角色编号',
  `role_name` VARCHAR(100) DEFAULT NULL COMMENT '角色名称',
  `sort` INT(5) DEFAULT '0' COMMENT '角色排序',
  `iscancel` INT(2) DEFAULT '0' COMMENT '是否作废',
  `type` INT(5) DEFAULT '1' COMMENT '类型',
  `state` INT(5) DEFAULT '1' COMMENT '状态',
  `remark` VARCHAR(200) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8 COMMENT='角色表';
```
#### 初始化角色表
``` SQL
INSERT  INTO `c_role`(`id`,`role_code`,`role_name`,`sort`,`iscancel`,`type`,`state`,`remark`) VALUES 
(1,'A0001','超级管理员',0,0,1,1,'超级管理员'),
(2,'A0002','普通管理员',1,0,1,1,'普通管理员'),
(3,'B0001','监控观察者',2,0,1,1,'观察者'),
(4,'C0001','访客',4,0,1,1,'访客');
```
### 角色菜单关联表
``` SQL
CREATE TABLE `c_role_menu` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `role_code` VARCHAR(100) DEFAULT NULL COMMENT '角色id',
  `menu_code` VARCHAR(100) DEFAULT NULL COMMENT '菜单code',
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT='角色菜单关联表';
```
#### 初始化角色菜单关联表
``` SQL
INSERT INTO `c_role_menu` VALUES (1, 'A0002', 'F001');
INSERT INTO `c_role_menu` VALUES (2, 'A0002', 'J001');
INSERT INTO `c_role_menu` VALUES (3, 'A0002', 'J001001');
INSERT INTO `c_role_menu` VALUES (4, 'A0002', 'N001');
INSERT INTO `c_role_menu` VALUES (5, 'A0002', 'N001001');
INSERT INTO `c_role_menu` VALUES (6, 'A0002', 'B001');
INSERT INTO `c_role_menu` VALUES (7, 'A0002', 'B001001');
INSERT INTO `c_role_menu` VALUES (8, 'A0002', 'B001002');
INSERT INTO `c_role_menu` VALUES (9, 'A0002', 'M001');
INSERT INTO `c_role_menu` VALUES (10, 'A0002', 'M001001');
INSERT INTO `c_role_menu` VALUES (11, 'A0002', 'M001002');
INSERT INTO `c_role_menu` VALUES (12, 'A0002', 'M001003');
INSERT INTO `c_role_menu` VALUES (13, 'B0001', 'F001');
INSERT INTO `c_role_menu` VALUES (14, 'B0001', 'M001');
INSERT INTO `c_role_menu` VALUES (15, 'B0001', 'M001001');
INSERT INTO `c_role_menu` VALUES (16, 'B0001', 'M001002');
INSERT INTO `c_role_menu` VALUES (17, 'B0001', 'M001003');
INSERT INTO `c_role_menu` VALUES (18, 'C0001', 'F001');
INSERT INTO `c_role_menu` VALUES (19, 'C0001', 'M001');
INSERT INTO `c_role_menu` VALUES (20, 'C0001', 'M001001');
INSERT INTO `c_role_menu` VALUES (21, 'C0001', 'M001002');
INSERT INTO `c_role_menu` VALUES (22, 'C0001', 'M001003');
```
你一定会发现，关联表里并没有超级管理员的信息，因为超级管理员拥有所有菜单的权限，所以我们只需要在查询时判断一下，如果是超级管理员我们就查询所有菜单就好啦，这也是个小技巧~
## 接口
### CMenuServiceImpl
在我们的菜单接口实现类里，我们完成一个接受角色code的方法。
``` Java
@Override
public CMenu menuTree(String roleCode) {
    List<CMenu> menulist = cmenuMapper.findByRoleCode(roleCode);
    return parentMenu(menulist);
}
```
parentMenu()方法就是我们之前完成的递归方法的入口。
## xml
``` xml
<!-- 根据权限查询菜单 -->
<select id="findByRoleCode" resultType="com.datas.manager.core.entity.CMenu">
    select t.id, t.code, t.fathercode, t.name, t.menu_url AS menuUrl,
        t.menu_image AS menuImage, t.level, t.sort, t.iscancel, t.type, t.isleaf
        from c_menu t
    <where>
        1 = 1
        <if test=" roleCode != 'A0001'">
            AND code IN (select menu_code from c_role_menu a where role_code = #{roleCode})
        </if>
    </where>
</select>
```

这样，我们就完成根据不同角色权限生成菜单树，这些都是准备工作，下一篇我们将完成初始化加载这些菜单，慢慢就会完成登录功能了~