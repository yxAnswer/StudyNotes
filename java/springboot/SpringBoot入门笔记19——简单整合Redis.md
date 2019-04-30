# SpringBoot整合redis

[TOC]

## 1、redis 简介及安装

官方说明：

Redis是一个开源（BSD许可），内存数据结构存储，用作数据库，缓存和消息代理。它支持数据结构，如字符串，散列，列表，集合，带有范围查询的排序集，位图，超级日志，具有半径查询和流的地理空间索引。Redis具有内置复制，Lua脚本，LRU驱逐，事务和不同级别的磁盘持久性，并通过Redis Sentinel提供高可用性并使用Redis Cluster自动分区。

官网地址：

[https://redis.io/](https://redis.io/)

具体的安装，使用这里不讲，redis官方文档都有说明。稍微在这贴一下安装命令

```
//第一步下载redis
$ wget http://download.redis.io/releases/redis-5.0.4.tar.gz
//解压redis
$ tar xzf redis-5.0.4.tar.gz
//进入redis安装目录
$ cd redis-5.0.4
//初始化
$ make
//启动服务
$ src/redis-server
//使用内置客户端交互测试
$ src/redis-cli
		//存入key-value
redis> set foo bar
OK
		//通过key 取出value
redis> get foo
"bar"
```

其他的比如开守护线程啊，修改配置啊，集群配置啊，这里先不讲。

### 1.1 redis 修改配置文件

装完redis，我们启动redis-server 这时候，命令行停在了服务，不能做其他事情了，所以我们需要做的是让redis以守护线程的方式启动。并且可以修改端口、ip地址。

**第一步：备份配置文件**

将安装目录下的 `redis.conf` 配置文件复制一份，放到我们的`/etc/redis` 下面，当然这个文件夹大家可以自己随便建。 

`cp  /usr/local/redis/redis.conf   /etc/redis`

**第二步： 修改配置文件**

By default Redis does not run as a daemon. Use 'yes' if you need it.# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程。

- 改为守护进程方式启动，将`daemonize  no` 改为`daemonize  yes`
- 注释掉` #bind 127.0.0.1`  或者配置自己的ip地址
- 修改保护模式，`protected-mode yes`为`protected-mode no`

### 1.2 修改防火墙，开放端口

默认端口为6379 ，当然也可以在上面配置文件中改。

（1）查看端口

`vim  /etc/sysconfig/iptables`

(2) 添加端口

`-A INPUT -P tcp -m tcp --dport 6379 -j ACCEPT`

(3) 重启防火墙

`service iptables restart `   当然我的环境是centos 6 的虚拟机

拓展性其他命令:

```
service iptables status 查看防火墙状态

1 关闭防火墙-----service iptables stop 
2 启动防火墙-----service iptables start 
3 重启防火墙-----service iptables restart 
4 查看防火墙状态--service iptables status 
5 永久关闭防火墙--chkconfig iptables off 
6 永久关闭后启用--chkconfig iptables on

```

（4） 本机测试是否已经开放端口

在我们的windows的命令行 `telnet  ip地址  端口号`   

## 2、SpringBoot集成Redis

看官网:[https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-redis](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-redis)

官网表示SpringBoot 其实是使用[Spring Data Redis](https://github.com/spring-projects/spring-data-redis) 来操作redis 的。具体SpringBoot是如何做的呢。这就不得不说一下 SpringBoot 的一堆 `stater`了。SpringBoot将常用的工具都集成，统一管理，用到什么直接导入。

[https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#using-boot-starter](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#using-boot-starter)

在官网我们看到，我们开发web 用的 `spring-boot-starter-web`等等等等，我们集成redis用到的是`spring-boot-starter-data-redis`

### 2.1集成starter 以及修改配置文件

第一步：pom.xml文件添加依赖

```xml
<dependency>
	 <groupId>org.springframework.boot</groupId>
	 <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

第二步：修改配置文件

常用如下：不同版本可能不一致，下面试老版本，最好看官网文档

```properties

#=========redis基础配置=========
spring.redis.database=0
spring.redis.host=192.168.13.129
spring.redis.port=6379
# 连接超时时间 单位 ms（毫秒）
spring.redis.timeout=3000ms

#=========redis线程池设置=========
# 连接池中的最大空闲连接，默认值也是8。
spring.redis.jedis.pool.max-idle=200


#连接池中的最小空闲连接，默认值也是0。
spring.redis.jedis.pool.min-idle=200
# 如果赋值为-1，则表示不限制；pool已经分配了maxActive个jedis实例，则此时pool的状态为exhausted(耗尽)。
spring.redis.jedis.pool.max-active=2000

# 等待可用连接的最大时间，单位毫秒，默认值为-1，表示永不超时
spring.redis.jedis.pool.max-wait=1000ms
```



看下SpringBoot官方文档的配置文件有哪些配置，2.1.4 版本的官方文档：

```properties
# REDIS (RedisProperties)
spring.redis.cluster.max-redirects= # 跨集群执行命令时要遵循的最大重定向数。
spring.redis.cluster.nodes= # 以逗号分隔的“host:port”对列表，用于引导。
spring.redis.database=0 # 连接工厂使用的数据库索引（应该有16个吧好像）
spring.redis.url= # 连接URL。覆盖主机、端口和密码。用户将被忽略。例如: redis://user:password@example.com:6379
spring.redis.host=localhost # Redis 服务端口号.
spring.redis.jedis.pool.max-active=8 #在给定时间，连接池可以分配的最大连接数。使用负数表示没有限制。
spring.redis.jedis.pool.max-idle=8 # 连接池中“空闲”连接的最大数量。使用负值表示无限数量的空闲连接。
spring.redis.jedis.pool.max-wait=-1ms #连接池用完且在引发异常之前阻塞等待的最大时间。使用负值无限期地阻塞。
spring.redis.jedis.pool.min-idle=0 # 连接池中最小的空闲连接。此设置只有在为正数时才有效。
spring.redis.lettuce.shutdown-timeout=100ms #关闭超时
spring.redis.password= # redis 服务器登录密码
spring.redis.port=6379 # Redis 服务端口号
spring.redis.sentinel.master= # redis 服务器名称
spring.redis.sentinel.nodes= # 以逗号分隔的“host:port”对列表。
spring.redis.ssl=false # 是否启用SSL支持
spring.redis.timeout= # 连接超时

```



### 2.2 代码测试redis

写个测试的controller 代码如下：

```java
package com.example.redisdemo.controller;

import com.example.redisdemo.domain.JsonData;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * redis 测试controller
 */
@RestController
@RequestMapping("/api/v1/redis")
public class RdisTestController {

    @Autowired
    private StringRedisTemplate redisTpl; //redis 的模板类，帮助我们操作redis

    @GetMapping(value = "add")
    public Object add() {

        //opsForValue : Returns the operations performed on simple values (or Strings in Redis terminology).

        redisTpl.opsForValue().set("name", "帅哥");//存值

        return JsonData.buildSuccess();

    }

    @GetMapping(value = "get")
    public Object get() {

        String value = redisTpl.opsForValue().get("name");//取值
        return JsonData.buildSuccess(value);

    }

}

```

我们写了一个存，一个区的两个接口，开启服务，测试下接口吧

调用下接口`http://localhost:8080/api/v1/redis/add`

然后再调用下 `http://localhost:8080/api/v1/redis/get`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190416235526969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

### 2.3 redis 工具类封装

```java
package com.example.redisdemo.utils;

import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

/**
 * 作者    ANSWER
 * 时间    2019/4/17 0:00
 * 文件    RedisUtils
 * 描述   Redis 工具类
 */
@Component
public class RedisUtils {


    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // ---------------------------------common------------------------------

    /**
     * 指定缓存失效时间
     *
     * @param key  键
     * @param time 时间(秒)
     * @return
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     *
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }

    /**
     * 判断key是否存在
     *
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除缓存
     *
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }
    // ---------------------------------String------------------------------

    /**
     * 普通缓存获取
     *
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     *
     * @param key   键
     * @param value 值
     * @return true成功 false失败
     */
    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 普通缓存放入并设置时间
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 递增
     *
     * @param key   键
     * @param delta 要增加几(大于0)
     * @return
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }

    /**
     * 递减
     *
     * @param key   键
     * @param delta 要减少几(小于0)
     * @return
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }
    // ---------------------------------Map------------------------------

    /**
     * HashGet
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return 值
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     *
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     *
     * @param key 键
     * @param map 对应多个键值
     * @return true 成功 false 失败
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * HashSet 并设置时间
     *
     * @param key  键
     * @param map  对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除hash表中的值
     *
     * @param key  键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }

    /**
     * 判断hash表中是否有该项的值
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }

    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key  键
     * @param item 项
     * @param by   要增加几(大于0)
     * @return
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }

    /**
     * hash递减
     *
     * @param key  键
     * @param item 项
     * @param by   要减少记(小于0)
     * @return
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }
    // ---------------------------------------set------------------------------------

    /**
     * 根据key获取Set中的所有值
     *
     * @param key 键
     * @return
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将数据放入set缓存
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 将set数据放入缓存
     *
     * @param key    键
     * @param time   时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     *
     * @param key 键
     * @return
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 移除值为value的
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 移除的个数
     */
    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
    // ------------------------------------list----------------------------------

    /**
     * 获取list缓存的内容
     *
     * @param key   键
     * @param start 开始
     * @param end   结束 0 到 -1代表所有值
     * @return
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 获取list缓存的长度
     *
     * @param key 键
     * @return
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 通过索引 获取list中的值
     *
     * @param key   键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     *
     * @param key   键
     * @param index 索引
     * @param value 值
     * @return
     */
    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 移除N个值为value
     *
     * @param key   键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */
    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
}

```

