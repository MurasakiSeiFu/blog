title: 'Util:Springboot类beans获取'
author: MurasakiSeiFu.
date: 2018-04-19 15:02:50
tags:
categories:
  - Utils
---
在项目中看到这个工具类，感觉很神奇，决定记录下来，并好好分析一下是如何实现的。
{% codeblock lang:Java %}
@Component
public class ApplicationContextUtil implements ApplicationContextAware {

    /**
     * 上下文对象实例
     */
    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        ApplicationContextUtil.applicationContext = applicationContext;
    }
    
    /**
     * 通过name获取Bean.
     * @param name
     * @return
     */
    @SuppressWarnings("unchecked")
    public static <T> T getBean(String name) {
        checkApplicationContext();
        return (T) applicationContext.getBean(name);
    }

    /**
     * 通过class获取Bean.
     * @param name
     * @return
     */	
    public static <T> T getBean(Class<T> clazz) {
        checkApplicationContext();
        return (T) applicationContext.getBean(clazz);
    }

    public static void cleanApplicationContext() {
        applicationContext = null;
    }

    private static void checkApplicationContext() {
        if (applicationContext == null) {
            throw new IllegalStateException("applicaitonContext未注入");
        }
    }
}
{% endcodeblock %}
##### **ApplicationContextAware**
这里最引人注意的就要数这个接口了，我们点开去看看它的源码和注释。
大致翻译一下：任何希望接受到运行的 ApplicationContext 通知的对象都可以实现这个接口。比如，当一个对象需要访问一组 bean 时，实现这个接口是有意义的。
在这里它有一个小提示，就是如果你实现该接口的目的是引用bean并对其进行配置，这个目的是可取，比仅为查找某个 bean 要好得多~
在这个接口里就只有一个方法，我们看一下。
{% codeblock lang:Java %}
void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
{% endcodeblock %}
这个方法就是用来设置ApplicationContext。通常，这个调用将用于初始化对象。
说白了，实现这个接口就是为了去获取 ApplicationContext 。
当我们获取到了ApplicationContext，就可以使用里面的方法。打上断点我们就会知道，这个工具类在springboot启动初始化加载后，就会把ApplicationContext对象注入到我们的工具类中。因为我们的实例和方法都是static修饰的，所以在外部依然可以获取到指定Bean的实例。

在单元测试的时候，这个工具类尤其好用，或者在我们想要初始化某些自定义的数据的时候。
{% codeblock lang:Java %}
CMenuService cMenuService = ApplicationContextUtil.getBean(CMenuServiceImpl.class);
CMenu cMenu = cMenuService.findAll();
{% endcodeblock %}
像这样，我们就可以在外部使用这个功能模块的接口了。