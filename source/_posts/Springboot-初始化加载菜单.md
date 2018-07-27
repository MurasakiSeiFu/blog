title: Springboot 初始化加载菜单
author: MurasakiSeiFu.
tags: []
categories: []
date: 2018-04-20 15:05:00
---
实际应用中，我们会有在项目服务启动的时候就去加载一些数据或做一些事情这样的需求。比如我们一直在做的这个菜单，就在项目启动的时候把数据提前加载到静态变量里。
## MenuUtils
依然是准备工作，我们先完成初始加载时需要的数据。基本思路也就是在这个工具类里把获取到的数据库数据存放到静态变量里，在加载的时候执行它就可以了~
``` Java
public class MenuUtils {

    protected Logger logger = LoggerFactory.getLogger(this.getClass());
    // 存放菜单的静态变量:ROLE_MENU
    public static Map<String, CMenu> ROLE_MENU = new HashMap<>();
    
    public static MenuUtils instance;
    // 实例化 MenuUtils
    public static MenuUtils getInstance() {
        if (instance == null) {
            instance = new MenuUtils();
        }
        return instance;
    }
    // 初始化
    public void init() {
        logger.info("MenuUtils init");
        loadRoleMenu();
    }

    private void loadRoleMenu() {
        CMenuService cMenuService = ApplicationContextUtil.getBean(CMenuServiceImpl.class);
        CRoleService cRoleService = ApplicationContextUtil.getBean(CRoleServiceImpl.class);
        // 查询所有角色
        List<CRole> roles = cRoleService.findAll();
        for (CRole cRole : roles) {
            // 根据不同角色生成菜单树
            CMenu cMenu = cMenuService.menuTree(cRole.getRoleCode());
            ROLE_MENU.put(cRole.getRoleCode(), cMenu);
        }
    }
}
```
ApplicationContextUtil 是我们自己封装的一个工具类，用来获取Bean的实例。就在这里:[ApplicationContextUtil](/2018/04/19/Util-Springboot类beans获取/)
cMenuService.menuTree 是我们根据角色获取菜单的接口，在这里:[cMenuService.menuTree](/2018/04/20/springboot-根据角色权限生成菜单树/)
## 如何启动加载数据
Springboot为我们提供了一个接口：CommandLineRunner
我们点进去看看源码和注释。
简单翻译一下：用来表示一个bean被包含在一个SpringApplication中时应该运行的接口。 多个CommandLineRunner bean可以在同一个应用程序上下文中定义，并且可以使用Ordered接口或@Order注解进行排序。
我们只需要重写这个run方法，就可以使我们这个bean在程序启动时就能加载了，并且如果有多个实现CommandLineRunner的实现类，可以用@Order注解进行排序，数字越小优先级越高。
``` Java
public interface CommandLineRunner {

	/**
	 * Callback used to run the bean.
	 * @param args incoming main method arguments
	 * @throws Exception on error
	 */
	void run(String... args) throws Exception;

}
```
## 完成我们的初始化工具类
``` Java
@Component
public class StoredDataInit implements CommandLineRunner {

    protected Logger logger = LoggerFactory.getLogger(this.getClass());

    @Override
    public void run(String... args) throws Exception {
        logger.info("MenuUtils run...");
        try {
            MenuUtils.getInstance().init();
        } catch (Exception e) {
            logger.error("MenuUtils run...error!!!!");
        }
    }
}
```

打上断点 ，我们就会发现，在启动完成前，就会加载我们的StoredDataInit了。
我们也会好奇是在什么时候加载的~
``` Java
@SpringBootApplication
@MapperScan("com.datas.manager.core.mapper")
public class ManagerBootApplication {

    private static final Logger LOGGER = LoggerFactory.getLogger(ManagerBootApplication.class);

    public static void main(String[] args) throws Exception {
        SpringApplication app = new SpringApplication(ManagerBootApplication.class);
        app.setBannerMode(Banner.Mode.OFF);
        // 就是在这步之后哦！
        ConfigurableApplicationContext context = app.run(args);

        ....
    }
}
```

我们现在完成了所有关于菜单以及角色权限菜单的工作，接着就要开始完成我们的登录了!








