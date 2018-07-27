title: Springboot 定时器用法
author: MurasakiSeiFu.
date: 2018-04-20 10:29:09
tags:
categories: Springboot
---
## 简介
顾名思义，定时器的意思就是每隔固定时间就执行一次的方法。
## 使用
### 1.开关
我们都知道 springboot 有一个自己的入口，就是@SpringBootApplication，我们的第一步，当然就是在启动类上打开定时器开关：@EnableScheduling，我们点开这个注解就会看到，在注释里，就已经告诉我该如何使用啦。因为太长，就不铺在这里了。
### 2.创建一个定时器类
``` java
@Component
public class DemoScheduler {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());
    
    @Scheduled(cron = "0/10 * * * * ?")
    public void createtable() {
        logger.info("10秒一次定时器！");
    }
}
```
这个方法的意思就是每10秒打印一次日志。
### 3.注解详解
#### @Component
@Component注解，我们都不陌生，就是将这个类作为组件扫描到spring容器中，这样我们就可以通过类路径扫描自动发现这个组件了。
#### @Scheduled
@Scheduled 就是定时器的核心注解。
源码注释上是这么说的，简单翻译一下：
一个用来标记要安排的方法的注释(翻译的不是太好..嘿嘿就是一个注释，用来标记要安排定时的方法)。 必须指定 cron(),fixedDelay()或fixedRate()属性中的一个。

注释的方法不能有任何参数。这是方法通常会有一个void返回类型; 如果不是，则通过调度程序调用时，返回的值将被忽略。也就是说被安排的方法是不期望有返回值的，即使有也会被忽略。

此注解可以用作元注解以创建具有属性覆盖的自定义组合批注。
我们看一下这个注解的源码。
``` Java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Repeatable(Schedules.class)  
public @interface Scheduled {

    String cron() default "";  
  
    String zone() default "";  
  
    long fixedDelay() default -1L;  
  
    String fixedDelayString() default "";  
  
    long fixedRate() default -1L;  
  
    String fixedRateString() default "";  
  
    long initialDelay() default -1L;  
  
    String initialDelayString() default "";  
} 
```
这些属性我们一个个说。
##### ** 1.cron **
这个是比较常用的属性。一种类似于cron的表达方式。可以作为一个触发器，时间包括时分秒月日年星期等，格式为"0*...MON-FRI"。
“星号” 表示为占位符，从左至右：秒分时日月年，最后一位则表示星期。
##### ** 2.zone **
将被cron表达式解析的时区。 默认情况下，此属性是空字符串（即使用服务器的本地时区将）。
##### ** 3.fixedDelay **
以毫秒为时间间隔。为Long型
##### ** 4.fixedDelayString **
以毫秒为时间间隔。为String类型
##### ** 5.fixedRate **
在调用之间以毫秒为单位,执行带有固定时间段的注释方法。为Long型
##### ** 6.fixedRateString **
在调用之间以毫秒为单位,执行带有固定时间段的注释方法。为String类型
##### ** 7.initialDelay **
第一次执行fixedRate()或fixedDelay()任务之前要延迟的毫秒数。为Long型
##### ** 8.initialDelayString **
第一次执行fixedRate()或fixedDelay()任务之前要延迟的毫秒数。为String类型

一般我们常用的就是 cron 属性。但是cron表达式我们一般也不是很了解 -。-、
在这里我给大家推荐一个网站 可以在线生成cron表达式，很方便~
网址：[在线cron表达式生成器](http://cron.qqe2.com/)









