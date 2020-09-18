[TOC]

# springboot 定时任务和异步任务

##  1、java 中常用的定时任务

> 1、常见定时任务 Java自带的java.util.Timer类
> ​			timer:配置比较麻烦，时间延后问题
> ​			timertask:不推荐
> 2、Quartz框架
> ​		配置更简单
> ​		xml或者注解
> 3、SpringBoot使用注解方式开启定时任务
> ​		1）启动类里面 @EnableScheduling开启定时任务，自动扫描
> ​		2）定时任务业务类 加注解 @Component被容器扫描
> ​		3）定时执行的方法加上注解 @Scheduled(fixedRate=2000) 定期执行一次



其中Quartz可以用来做复杂的定时任务。SpringBoot 内部自带的定时任务使用更加简单方便。

## 2、最简单的定时任务快速体验

> 在写这篇文章的时候，新建了一个项目studyf4用来存储接下来学习过程中写的demo，以便复习。
>
> 体验定时任务就三步：
>
> 1）启动类里面 @EnableScheduling开启定时任务，自动扫描
> ​		2）定时任务业务类 加注解 @Component被容器扫描
> ​		3）定时执行的方法加上注解 @Scheduled(fixedRate=2000) 定期执行一次

看代码：

启动类：

```java
package com.study;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

/**
 * 文件名: MainApplication.java
 * 描述：启动类
 * Create by Google on 2018/10/15 22:43
 */
@SpringBootApplication
@EnableScheduling //允许开启定时任务
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }
}

```

定时任务测试类：

TestTask:

```java
package com.study.task;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * 文件名: TestTask.java
 * 描述：定时任务类业务类
 * Create by Google on 2018/10/15 22:43
 */
@Component//次注解可以被springboot扫描到
public class TestTask {

    private int i=0;
    //每一秒执行一次
    @Scheduled(fixedDelay = 1000)
    public void sum(){
        i++;
        SimpleDateFormat sf=new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        System.out.println("定时任务执行了"+i+"次"+sf.format(new Date()));
    }
}

```

看输出结果：

```
定时任务执行了294次2018-10-15 10:53:36
定时任务执行了295次2018-10-15 10:53:37
定时任务执行了296次2018-10-15 10:53:38
定时任务执行了297次2018-10-15 10:53:39
定时任务执行了298次2018-10-15 10:53:40
定时任务执行了299次2018-10-15 10:53:41
定时任务执行了300次2018-10-15 10:53:42
定时任务执行了301次2018-10-15 10:53:43
定时任务执行了302次2018-10-15 10:53:44
定时任务执行了303次2018-10-15 10:53:45
太多了不展示了，太简单了
```



## 3、定时任务稍微高级点的玩法

### 3.1 使用定时任务表达式

> 		1、cron 定时任务表达式 @Scheduled(cron="*/1 * * * * *") 表示每秒
> ​			1）crontab 工具  https://tool.lu/crontab/
> ​		2、fixedRate: 定时多久执行一次（上一次开始执行时间点后xx秒再次执行；）
> ​		3、fixedDelay: 上一次执行结束时间点后xx秒再次执行
> ​		4、fixedDelayString:  字符串形式，可以通过配置文件指定

上面这几种使用方式在Scheduled源码中都有说明

```java
/*
 * Copyright 2002-2018 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.scheduling.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * An annotation that marks a method to be scheduled. Exactly one of
 * the {@link #cron()}, {@link #fixedDelay()}, or {@link #fixedRate()}
 * attributes must be specified.
 *
 * <p>The annotated method must expect no arguments. It will typically have
 * a {@code void} return type; if not, the returned value will be ignored
 * when called through the scheduler.
 *
 * <p>Processing of {@code @Scheduled} annotations is performed by
 * registering a {@link ScheduledAnnotationBeanPostProcessor}. This can be
 * done manually or, more conveniently, through the {@code <task:annotation-driven/>}
 * element or @{@link EnableScheduling} annotation.
 *
 * <p>This annotation may be used as a <em>meta-annotation</em> to create custom
 * <em>composed annotations</em> with attribute overrides.
 *
 * @author Mark Fisher
 * @author Dave Syer
 * @author Chris Beams
 * @since 3.0
 * @see EnableScheduling
 * @see ScheduledAnnotationBeanPostProcessor
 * @see Schedules
 */
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(Schedules.class)
public @interface Scheduled {

	/**
	 * A cron-like expression, extending the usual UN*X definition to include
	 * triggers on the second as well as minute, hour, day of month, month
	 * and day of week.  e.g. {@code "0 * * * * MON-FRI"} means once per minute on
	 * weekdays (at the top of the minute - the 0th second).
	 * @return an expression that can be parsed to a cron schedule
	 * @see org.springframework.scheduling.support.CronSequenceGenerator
	 */
	String cron() default "";

	/**
	 * A time zone for which the cron expression will be resolved. By default, this
	 * attribute is the empty String (i.e. the server's local time zone will be used).
	 * @return a zone id accepted by {@link java.util.TimeZone#getTimeZone(String)},
	 * or an empty String to indicate the server's default time zone
	 * @since 4.0
	 * @see org.springframework.scheduling.support.CronTrigger#CronTrigger(String, java.util.TimeZone)
	 * @see java.util.TimeZone
	 */
	String zone() default "";

	/**
	 * Execute the annotated method with a fixed period in milliseconds between the
	 * end of the last invocation and the start of the next.
	 * @return the delay in milliseconds
	 */
	long fixedDelay() default -1;

	/**
	 * Execute the annotated method with a fixed period in milliseconds between the
	 * end of the last invocation and the start of the next.
	 * @return the delay in milliseconds as a String value, e.g. a placeholder
	 * or a {@link java.time.Duration#parse java.time.Duration} compliant value
	 * @since 3.2.2
	 */
	String fixedDelayString() default "";

	/**
	 * Execute the annotated method with a fixed period in milliseconds between
	 * invocations.
	 * @return the period in milliseconds
	 */
	long fixedRate() default -1;

	/**
	 * Execute the annotated method with a fixed period in milliseconds between
	 * invocations.
	 * @return the period in milliseconds as a String value, e.g. a placeholder
	 * or a {@link java.time.Duration#parse java.time.Duration} compliant value
	 * @since 3.2.2
	 */
	String fixedRateString() default "";

	/**
	 * Number of milliseconds to delay before the first execution of a
	 * {@link #fixedRate()} or {@link #fixedDelay()} task.
	 * @return the initial delay in milliseconds
	 * @since 3.2
	 */
	long initialDelay() default -1;

	/**
	 * Number of milliseconds to delay before the first execution of a
	 * {@link #fixedRate()} or {@link #fixedDelay()} task.
	 * @return the initial delay in milliseconds as a String value, e.g. a placeholder
	 * or a {@link java.time.Duration#parse java.time.Duration} compliant value
	 * @since 3.2.2
	 */
	String initialDelayString() default "";

}

```

###  3.2 cron（）的使用

网上的资料：

和linux 的cron差不多，比如[https://tool.lu/crontab/](https://tool.lu/crontab/) cron表达式在线工具

```
Linux
*    *    *    *    *    
-    -    -    -    -    
|    |    |    |    |    
|    |    |    |    |    
|    |    |    |    +----- day of week (0 - 7) (Sunday=0 or 7)
|    |    |    +---------- month (1 - 12)
|    |    +--------------- day of month (1 - 31)
|    +-------------------- hour (0 - 23)
+------------------------- min (0 - 59)

Java(Spring)
*    *    *    *    *    *
-    -    -    -    -    -
|    |    |    |    |    |
|    |    |    |    |    +----- day of week (0 - 7) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
|    |    |    |    +---------- month (1 - 12) OR jan,feb,mar,apr ...
|    |    |    +--------------- day of month (1 - 31)
|    |    +-------------------- hour (0 - 23)
|    +------------------------- min (0 - 59)
+------------------------------ second (0 - 59)
```



Cron表达式是一个字符串，字符串以5或6个空格隔开，分开工6或7个域，每一个域代表一个含义,Cron有如下两种语法 

```
Seconds Minutes Hours DayofMonth Month DayofWeek Year 或 
Seconds Minutes Hours DayofMonth Month DayofWeek 
```

执行顺序

> 按顺序依次为：
>
> 秒（0~59）
> 分钟（0~59）
> 小时（0~23）
> 天（0~31）
> 月（0~11）
> 星期（1~7 1=SUN 或 SUN，MON，TUE，WED，THU，FRI，SAT）
> 年份(1970-2099)
>
> Seconds:可出现,-  *  / 四个字符，有效范围为0-59的整数   
> Minutes:可出现,-  *  / 四个字符，有效范围为0-59的整数   
> Hours:可出现,-  *  / 四个字符，有效范围为0-23的整数   
> DayofMonth:可出现,-  *  / ? L W C八个字符，有效范围为0-31的整数    
> Month:可出现,-  *  / 四个字符，有效范围为1-12的整数或JAN-DEC   
> DayofWeek:可出现,-  *  / ? L C #四个字符，有效范围为1-7的整数或SUN-SAT两个范围。1表示星期天，2表示星期一， 依次类推   
> Year:可出现,-  *  / 四个字符，有效范围为1970-2099年​ ​ 

```
(1)*：表示匹配该域的任意值，假如在Minutes域使用*,即表示每分钟都会触发事件。   
  
(2)?:只能用在DayofMonth和DayofWeek两个域。它也匹配域的任意值，但实际不会。因为DayofMonth和DayofWeek会相互影响。例如想在每月的20日触发调度，不管20日到底是星期几，则只能使用如下写法： 13  13 15 20 * ?,其中最后一位只能用？，而不能使用*，如果使用*表示不管星期几都会触发，实际上并不是这样。
"?" 与{日期}互斥，即意味着若明确指定{日期}触发，则表示{星期}无意义，以免引起冲突和混乱
  
(3)-:表示范围，例如在Minutes域使用5-20，表示从5分到20分钟每分钟触发一次   
  
(4)/：表示起始时间开始触发，然后每隔固定时间触发一次，例如在Minutes域使用5/20,则意味着5分钟触发一次，而25，45等分别触发一次.   
  
(5),:表示列出枚举值值。例如：在Minutes域使用5,20，则意味着在5和20分每分钟触发一次。   
  
(6)L:表示最后，只能出现在DayofWeek和DayofMonth域，如果在DayofWeek域使用5L,意味着在最后的一个星期四触发。   
  
(7)W:表示有效工作日(周一到周五),只能出现在DayofMonth域，系统将在离指定日期的最近的有效工作日触发事件。例如：在 DayofMonth使用5W，如果5日是星期六，则将在最近的工作日：星期五，即4日触发。如果5日是星期天，则在6日触发；如果5日在星期一到星期五中的一天，则就在5日触发。另外一点，W的最近寻找不会跨过月份   
  
(8)LW:这两个字符可以连用，表示在某个月最后一个工作日，即最后一个星期五。   
  
(9)#:用于确定每个月第几个星期几，只能出现在DayofMonth域。例如在4#2，表示某月的第二个星期三。
```

例子：

比如`@Scheduled(cron = "*/30 * 7-22 * * * *")`

表示的是每天7点到22点之间，每隔30秒执行一次。

另外fixedRate和fixedDelay、 fixedDelayString  使用方式是一样的

`@Scheduled（fixedRate=2000）`

`@Scheduled（fixedDelay=2000）`

这两个的区别注意一下，fixedRate 是从上次开始执行时间算起，执行两秒后，再执行第二次

而fixedDelay是从上次结束执行时算起，执行两秒后，再执行第二次

fixedDelayString:  字符串形式，可以通过配置文件指定，比如：

`@Scheduled(fixedDelayString="2000")` 我们可以再配置文件中指定定时的时间，然后再这里用字符串的方式引用。

### 3.3 但是要注意的是：

**spring的schedule值支持6个域的表达式，也就是不能设定年，如果超过六个则会报错 ** 看源码：

```java
/**
	 * Parse the given pattern expression.
	 */
	private void parse(String expression) throws IllegalArgumentException {
		String[] fields = StringUtils.tokenizeToStringArray(expression, " ");
		if (!areValidCronFields(fields)) {
			throw new IllegalArgumentException(String.format(
					"Cron expression must consist of 6 fields (found %d in \"%s\")", fields.length, expression));
		}
		setNumberHits(this.seconds, fields[0], 0, 60);
		setNumberHits(this.minutes, fields[1], 0, 60);
		setNumberHits(this.hours, fields[2], 0, 24);
		setDaysOfMonth(this.daysOfMonth, fields[3]);
		setMonths(this.months, fields[4]);
		setDays(this.daysOfWeek, replaceOrdinals(fields[5], "SUN,MON,TUE,WED,THU,FRI,SAT"), 8);
		if (this.daysOfWeek.get(7)) {
			// Sunday can be represented as 0 or 7
			this.daysOfWeek.set(0);
			this.daysOfWeek.clear(7);
		}
	}
 
	private static boolean areValidCronFields(String[] fields) {
		return (fields != null && fields.length == 6);
	}

```

### 3.4 各域支持的数据类型

    秒：可出现", - * /"四个字符，有效范围为0-59的整数  
    
    分：可出现", - * /"四个字符，有效范围为0-59的整数  
    
    时：可出现", - * /"四个字符，有效范围为0-23的整数  
    
    每月第几天：可出现", - * / ? L W C"八个字符，有效范围为0-31的整数  
    
    月：可出现", - * /"四个字符，有效范围为1-12的整数或JAN-DEc  
    
    星期：可出现", - * / ? L C #"四个字符，有效范围为1-7的整数或SUN-SAT两个范围。1表示星期天，2表示星期一， 依次类推
```
  * : 表示匹配该域的任意值，比如在秒*, 就表示每秒都会触发事件。；

    ? : 只能用在每月第几天和星期两个域。表示不指定值，当2个子表达式其中之一被指定了值以后，为了避免冲突，需要将另一个子表达式的值设为“?”；

    - : 表示范围，例如在分域使用5-20，表示从5分到20分钟每分钟触发一次  

    / : 表示起始时间开始触发，然后每隔固定时间触发一次，例如在分域使用5/20,则意味着5分，25分，45分，分别触发一次.  

    , : 表示列出枚举值。例如：在分域使用5,20，则意味着在5和20分时触发一次。  

    L : 表示最后，只能出现在星期和每月第几天域，如果在星期域使用1L,意味着在最后的一个星期日触发。  

    W : 表示有效工作日(周一到周五),只能出现在每月第几日域，系统将在离指定日期的最近的有效工作日触发事件。注意一点，W的最近寻找不会跨过月份  

    LW : 这两个字符可以连用，表示在某个月最后一个工作日，即最后一个星期五。  

    # : 用于确定每个月第几个星期几，只能出现在每月第几天域。例如在1#3，表示某月的第三个星期日。

```



## 4、异步任务

### 4.1 简单的体验异步任务

1、首先在启动类里面添加注解`@EnableAsync //开启异步任务`

2、在需要作为异步执行的类加上`@Component`标记组件被容器扫描

3、异步类或者方法上添加 `@Async`注解，表示这个类或者方法是一个异步的

4、在其他地方调用这些方法，所有的方法会异步执行。

例如： 当我们有一些耗时操作，但是还必须返回，就可以通过并行的方式，比如同一个接口有3个操作比较耗时，如果是并行处理，所用时间就是他们3个任务耗时和，如果是并行处理，取决于最耗时那个任务，可以提高效率。

```java
@Component
@Async
public class AsyncTask {


    public void  task1(){
        try {
            Thread.sleep(5000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("task 1");
    }

    public Future<String> task2(){
        try {
            Thread.sleep(5000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("task 2");
        return new AsyncResult<String>("task 2执行结束");
    }

    public Future<String> task3(){
        try {
            Thread.sleep(5000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("task 3");
        return new AsyncResult<String>("task 3执行结束");
    }
}

```

```java
  	@Autowired
    AsyncTask asyncTask;

    @GetMapping("testAsync")
    public ApiResult testAsyncTask(){
        long start = System.currentTimeMillis();
        Future<String> stringFuture2 = asyncTask.task2();
        Future<String> stringFuture3 = asyncTask.task3();
        String s1="",s2="";
        for (;;){
            if (stringFuture2.isDone()&&stringFuture3.isDone()){
                try {
                     s1 = stringFuture2.get();
                     s2 = stringFuture3.get();
                    System.out.println(stringFuture2.get());
                    System.out.println(stringFuture3.get());

                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (ExecutionException e) {
                    e.printStackTrace();
                }finally {
                    break;
                }
            }
        }
        long end = System.currentTimeMillis();
        System.out.println(end-start);
        return ApiResult.success(s1+s2);
    }
```



> 注意点：
> 1）要把异步任务封装到类里面，不能直接写到Controller
> 2）增加Future<String> 返回结果 AsyncResult<String>("task执行完成");  
> 3）如果需要拿到结果 需要判断全部的 task.isDone()	

如果要返回结果，比如

```java
  @Async
  public Future<String> task3() throws InterruptedException {
        long begin = System.currentTimeMillis();
        Thread.sleep(5000);
        long end = System.currentTimeMillis();
        System.out.println("任务3耗时---"+(end-begin));
        return new AsyncResult<>("任务3");
    }
```

Future<V>  返回异步的结果

AsyncResult<>

```java
Future<String> stringFuture = asyncTask.task3();
stringFuture.isDone();//表示是否执行完
stringFuture.cancel();//是否取消
```



