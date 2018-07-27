title: springboot java递归生成菜单树
author: MurasakiSeiFu.
date: 2018-04-18 16:43:10
tags:
categories:
  - Springboot
---
首先我们要准备数据.
{% codeblock 菜单表sql lang:SQL %}
CREATE TABLE `c_menu` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `code` VARCHAR(10) DEFAULT NULL COMMENT '编号',
  `fathercode` VARCHAR(10) DEFAULT '-1' COMMENT '父类编号',
  `name` VARCHAR(100) DEFAULT NULL COMMENT '目录名称',
  `menu_url` VARCHAR(200) DEFAULT '#' COMMENT '目录路径',
  `menu_image` VARCHAR(200) DEFAULT '#' COMMENT '图片',
  `level` INT(5) DEFAULT '-1' COMMENT '目录等级',
  `sort` INT(5) DEFAULT '0' COMMENT '目录排序',
  `isleaf` int(2) DEFAULT NULL COMMENT '是否叶子 1是 0不是',
  `iscancel` INT(2) DEFAULT '0' COMMENT '是否作废',
  `type` INT(5) DEFAULT '1' COMMENT '类型',
  `state` INT(5) DEFAULT '1' COMMENT '状态',
  `remark` VARCHAR(200) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT='菜单目录表';
{% endcodeblock %}
###### ** 叶子节点 **
我们一级菜单的fathercode为 #，二级菜单的fathercode指向一级菜单的code，多级菜单以此类推。在这里有一个重要的字段，就是 isleaf(是否为叶子节点)，这个字段就是作为递归查询时的条件。当一个目录被定义为叶子节点，也就是说这个目录下就没有叶子目录了。

###### ** 准备工作 **
我们要组装的数据格式是在菜单实体中组装一个菜单list，每一个list也就是对应着一个级别的菜单。
{% codeblock %}
（菜单实体）      {一级菜单list}  {二级菜单list}
CMenu-list————---CMenu-list————---CMenu
               |	       |--CMenu
               |	       |--CMenu
               |
               |-CMenu-list————---CMenu
               		       |--CMenu
               		       |--CMenu
{% endcodeblock %}
基本的数据格式我们考虑好了之后，就可以开始完成功能了。
##### ** CMenu **
在CMenu实体中，加上一个初始化的list和初始化是否是叶子节点的构造方法。
{% codeblock list属性 lang:Java %}
public CMenu(Integer isleaf) {
	this.isleaf = isleaf;
}
private List<CMenu> menus = new ArrayList<>();
//对应的get/set方法
{% endcodeblock %}
##### ** CMenuServiceImpl **
我们查询出所有的菜单。
{% codeblock CMenu实现类 lang:Java %}
@Override
public CMenu findAll() {
	List<CMenu> menulist = cmenuMapper.findAll();
	return parentMenu(menulist);
}
{% endcodeblock %}
parentMenun()方法就是进入我们递归的方法啦。
{% codeblock lang:Java %}
  //菜单树
  private CMenu parentMenu(List<CMenu> menulist) {
      //初始化菜单的子节点
      CMenu parentMenu = new CMenu(0);
      //初始父亲节点为 -1，为了查出一级菜单
      parentMenu.setMenus(menus("-1", listtomap(menulist)));
  
      return parentMenu;
  }

  //组装map
  private Map<String, List<CMenu>> listtomap(List<CMenu> menulist) {
      Map<String, List<CMenu>> menuMap = new HashMap<>();
      //组装map key:fatherCode value:旗下的子菜单
      //可以省略一次循环
      for (CMenu cMenu : menulist) {
          List<CMenu> menus = menuMap.get(cMenu.getFathercode());
          if (menus == null) {
              menus = new ArrayList<>();
          }
          menus.add(cMenu);
          menuMap.put(cMenu.getFathercode(), menus);
      }
      return menuMap;
  }

  //递归方法
  private List<CMenu> menus(String fatherCode, Map<String, List<CMenu>> menuMap) {
      List<CMenu> childlist = menuMap.get(fatherCode);
      //退出递归的方法，不然就会死循环啦
      if (childlist == null || childlist.size() == 0) {
          return null;
      }
      //根据父类id查询旗下子类
      for (CMenu cMenu : childlist) {
          //如果是叶子节点 则退出循环
          if (cMenu == null || cMenu.getIsleaf() == null || cMenu.getIsleaf() == 1) {
              continue;
          }
          //如果不是 继续递归
          cMenu.setMenus(menus(cMenu.getCode(), menuMap));
      }
      return childlist;
  }
{% endcodeblock %}
    
生成菜单树这是第一步，下一篇是完成权限登陆下的菜单初始化，最终我们要完成一个完成的登录功能~
    
    
    
    
    
    
    