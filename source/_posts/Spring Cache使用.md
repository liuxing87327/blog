title: "Spring Cache使用"
date: 2015-06-18 00:27:00
category: Java
tags: [Spring,Cache]

---

*记录下自己项目在用的Spring Cache的使用方式。*
*Spring的抽象已经做得够好了，适合于大多数场景，非常复杂的就需要自己AOP实现了。*
*Spring官网的文档挺不错的，但是对Cache这块的介绍不是很详细，结合网上大牛的博文，汇总下文。*

## 缓存概念

{% blockquote 缓存简介 http://jinnianshilongnian.iteye.com/blog/2001040 开涛的博客 %}
### 缓存简介
缓存，我的理解是：让数据更接近于使用者；工作机制是：先从缓存中读取数据，如果没有再从慢速设备上读取实际数据（数据也会存入缓存）；缓存什么：那些经常读取且不经常修改的数据/那些昂贵（CPU/IO）的且对于相同的请求有相同的计算结果的数据。如CPU--L1/L2--内存--磁盘就是一个典型的例子，CPU需要数据时先从L1/L2中读取，如果没有到内存中找，如果还没有会到磁盘上找。还有如用过Maven的朋友都应该知道，我们找依赖的时候，先从本机仓库找，再从本地服务器仓库找，最后到远程仓库服务器找；还有如京东的物流为什么那么快？他们在各个地都有分仓库，如果该仓库有货物那么送货的速度是非常快的。
 
### 缓存命中率
即从缓存中读取数据的次数 与 总读取次数的比率，命中率越高越好：
命中率 = 从缓存中读取次数 / (总读取次数[从缓存中读取次数 + 从慢速设备上读取的次数])
Miss率 = 没有从缓存中读取的次数 / (总读取次数[从缓存中读取次数 + 从慢速设备上读取的次数])
 
这是一个非常重要的监控指标，如果做缓存一定要健康这个指标来看缓存是否工作良好；
 
### 缓存策略
#### Eviction policy
移除策略，即如果缓存满了，从缓存中移除数据的策略；常见的有LFU、LRU、FIFO：
- FIFO（First In First Out）：先进先出算法，即先放入缓存的先被移除；
- LRU（Least Recently Used）：最久未使用算法，使用时间距离现在最久的那个被移除；
- LFU（Least Frequently Used）：最近最少使用算法，一定时间段内使用次数（频率）最少的那个被移除；
 
#### TTL（Time To Live ）
存活期，即从缓存中创建时间点开始直到它到期的一个时间段（不管在这个时间段内有没有访问都将过期）
 
#### TTI（Time To Idle）
空闲期，即一个数据多久没被访问将从缓存中移除的时间。
 
 
到此，基本了解了缓存的知识，在Java中，我们一般对调用方法进行缓存控制，比如我调用"findUserById(Long id)"，那么我应该在调用这个方法之前先从缓存中查找有没有，如果没有再掉该方法如从数据库加载用户，然后添加到缓存中，下次调用时将会从缓存中获取到数据。
 
自Spring 3.1起，提供了类似于@Transactional注解事务的注解Cache支持，且提供了Cache抽象；在此之前一般通过AOP实现；使用Spring Cache的好处：
- 提供基本的Cache抽象，方便切换各种底层Cache；
- 通过注解Cache可以实现类似于事务一样，缓存逻辑透明的应用到我们的业务代码上，且只需要更少的代码就可以完成；
- 提供事务回滚时也自动回滚缓存；
- 支持比较复杂的缓存逻辑；
 
对于Spring Cache抽象，主要从以下几个方面学习：
- Cache API及默认提供的实现
- Cache注解
- 实现复杂的Cache逻辑

{% endblockquote %}


## Spring Cache简介
{% blockquote Spring Cache 介绍 http://www.cnblogs.com/rollenholt/p/4202631.html Spring Cache 介绍 - Rollen Holt - 博客园 %}
Spring3.1开始引入了激动人心的基于注释（annotation）的缓存（cache）技术，它本质上不是一个具体的缓存实现方案（例如EHCache 或者 OSCache），而是一个对缓存使用的抽象，通过在既有代码中添加少量它定义的各种 annotation，即能够达到缓存方法的返回对象的效果。

Spring的缓存技术还具备相当的灵活性，不仅能够使用 SpEL（Spring Expression Language）来定义缓存的key和各种condition，还提供开箱即用的缓存临时存储方案，也支持和主流的专业缓存例如EHCache、memcached集成。

其特点总结如下：

- 通过少量的配置 annotation 注释即可使得既有代码支持缓存
- 支持开箱即用 Out-Of-The-Box，即不用安装和部署额外第三方组件即可使用缓存
- 支持 Spring Express Language，能使用对象的任何属性或者方法来定义缓存的 key 和 condition
- 支持 AspectJ，并通过其实现任何方法的缓存支持
- 支持自定义 key 和自定义缓存管理者，具有相当的灵活性和扩展性
{% endblockquote %}


## API介绍
### Cache接口

`理解这个接口有助于我们实现自己的缓存管理器`

```java
package org.springframework.cache;

public interface Cache {

	/**
	 * 缓存的名字
	 */
	String getName();

	/**
	 * 得到底层使用的缓存
	 */
	Object getNativeCache();

	/**
	 * 根据key得到一个ValueWrapper，然后调用其get方法获取值 
	 */
	ValueWrapper get(Object key);

	/**
	 * 根据key，和value的类型直接获取value  
	 */
	<T> T get(Object key, Class<T> type);

	/**
	 * 存数据
	 */
	void put(Object key, Object value);

	/**
	 * 如果值不存在，则添加，用来替代如下代码
	 * Object existingValue = cache.get(key);
	 * if (existingValue == null) {
	 *     cache.put(key, value);
	 *     return null;
	 * } else {
	 *     return existingValue;
	 * }
	 */
	ValueWrapper putIfAbsent(Object key, Object value);

	/**
	 * 根据key删数据
	 */
	void evict(Object key);

	/**
	 * 清空数据
	 */
	void clear();

	/**
	 * 缓存值的Wrapper  
	 */
	interface ValueWrapper {
		/**
		 * 得到value
		 */
		Object get();
	}
}
```

#### 默认实现
默认已经实现了几个常用的cache
位于spring-context-x.RELEASE.jar和spring-context-support-x.RELEASE.jar的cache目录下
- ConcurrentMapCache：基于java.util.concurrent.ConcurrentHashMap
- GuavaCache：基于Google的Guava工具
- EhCacheCache：基于Ehcache
- JCacheCache：基于javax.cache.Cache（不常用）

### CacheManager
`用来管理多个cache`

```java
package org.springframework.cache;

import java.util.Collection;

public interface CacheManager {

	/**
	 * 根据cache名获取cache
	 */
	Cache getCache(String name);

	/**
	 * 得到所有cache的名字
	 */
	Collection<String> getCacheNames();

}
```

#### 默认实现
对应Cache接口的默认实现

- ConcurrentMapCacheManager / ConcurrentMapCacheFactoryBean
- GuavaCacheManager
- EhCacheCacheManager / EhCacheManagerFactoryBean
- JCacheCacheManager / JCacheManagerFactoryBean


### CompositeCacheManager
用于组合CacheManager，可以从多个CacheManager中轮询得到相应的Cache

```xml
<bean id="cacheManager" class="org.springframework.cache.support.CompositeCacheManager">
    <property name="cacheManagers">
        <list>
            <ref bean="concurrentMapCacheManager"/>
            <ref bean="guavaCacheManager"/>
        </list>
    </property>
    <!-- 都找不到时，不返回null，而是返回NOP的Cache -->
    <property name="fallbackToNoOpCache" value="true"/>
</bean>
```

### 事务
除GuavaCacheManager外，其他Cache都支持Spring事务，如果注解方法出现事务回滚，对应缓存操作也会回滚

### 缓存策略
都是Cache自行维护，Spring只提供对外抽象API

## Cache注解
每个注解都有多个参数，这里不一一列出，建议进入源码查看注释

### 启用注解
```xml
<cache:annotation-driven cache-manager="cacheManager"/>
```

### @CachePut
写数据

```java
@CachePut(value = "addPotentialNoticeCache", key = "targetClass + '.' + #userCode")
public List<PublicAutoAddPotentialJob.AutoAddPotentialNotice> put(int userCode, List<PublicAutoAddPotentialJob.AutoAddPotentialNotice> noticeList) {
    LOGGER.info("缓存（{}）的公客自动添加潜在客的通知", userCode);
    return noticeList;
}
```

### @CacheEvict
失效数据

```java
@CacheEvict(value = "addPotentialNoticeCache", key = "targetClass + '.' + #userCode")
public void remove(int userCode) {
    LOGGER.info("清除（{}）的公客自动添加潜在客的通知", userCode);
}

```

### @Cacheable
这个用的比较多
用在查询方法上，先从缓存中读取，如果没有再调用方法获取数据，然后把数据添加到缓存中

```java
@Cacheable(value = "kyAreaCache", key="targetClass + '.' + methodName + '.' + #areaId")
public KyArea findById(String areaId) {
    // 业务代码省略
}
```

### 运行流程

1.  首先执行@CacheEvict（如果beforeInvocation=true且condition 通过），如果allEntries=true，则清空所有
2.  接着收集@Cacheable（如果condition 通过，且key对应的数据不在缓存），放入cachePutRequests（也就是说如果cachePutRequests为空，则数据在缓存中）
3.  如果cachePutRequests为空且没有@CachePut操作，那么将查找@Cacheable的缓存，否则result=缓存数据（也就是说只要当没有cache put请求时才会查找缓存）
4.  如果没有找到缓存，那么调用实际的API，把结果放入result
5.  如果有@CachePut操作(如果condition 通过)，那么放入cachePutRequests
6.  执行cachePutRequests，将数据写入缓存（unless为空或者unless解析结果为false）；
7.  执行@CacheEvict（如果beforeInvocation=false 且 condition 通过），如果allEntries=true，则清空所有

### SpEL上下文数据

在使用时，#root.methodName 等同于 methodName

|   名称   |     位置    |   描述   |     示例     |
| :---------- | :--------| :-------- | :------ |
|   methodName  |   root对象   |  当前被调用的方法名    |   #root.methodName   |
|   method  |   root对象   |   当前被调用的方法   |   #root.method.name   |
|   target  |   root对象   |   当前被调用的目标对象   |   #root.target   |
|   targetClass  |   root对象   |   当前被调用的目标对象类   |  #root.targetClass    |
|   args  |   root对象   |   当前被调用的方法的参数列表   |   #root.args[0]   |
|   caches  |  root对象    |   当前方法调用使用的缓存列表（如@Cacheable(value={"cache1", "cache2"})），则有两个cache   |   #root.caches[0].name   |
|   argument name  |   执行上下文   |   当前被调用的方法的参数，如findById(Long id)，我们可以通过#id拿到参数   |   #user.id   |
|   result  |   执行上下文   |   方法执行后的返回值（仅当方法执行之后的判断有效，如‘unless’，'cache evict'的beforeInvocation=false）   |   #result   |

### 条件缓存
主要是在注解内用condition和unless的表达式分别对参数和返回结果进行筛选后缓存

### @Caching
多个缓存注解组合使用

```java
@Caching(
        put = {
                @CachePut(value = "user", key = "#user.id"),
                @CachePut(value = "user", key = "#user.username"),
                @CachePut(value = "user", key = "#user.email")
        }
)
public User save(User user) {

}
```

### 自定义缓存注解
把一些特殊场景的注解包装到一个独立的注解中，比如@Caching组合使用的注解

```java
@Caching(
        put = {
                @CachePut(value = "user", key = "#user.id"),
                @CachePut(value = "user", key = "#user.username"),
                @CachePut(value = "user", key = "#user.email")
        }
)
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface UserSaveCache {

}
```

```java
@UserSaveCache
public User save(User user) {

}
```

## 示例
### 基于ConcurrentMapCache
#### 自定义CacheManager
我需要使用有容量限制和缓存失效时间策略的Cache，默认的ConcurrentMapCacheManager没法满足
通过实现CacheManager接口定制出自己的CacheManager。
还是拷贝ConcurrentMapCacheManager，使用Guava的Cache做底层容器，因为Guava的Cache容器可以设置缓存策略


`新增了exp、maximumSize两个策略变量`
`修改底层Cache容器的创建`

下面只列出自定义的代码，其他的都是Spring的ConcurrentMapCacheManager的代码

```java
import com.google.common.cache.CacheBuilder;
import org.springframework.cache.Cache;
import org.springframework.cache.CacheManager;
import org.springframework.cache.concurrent.ConcurrentMapCache;

import java.util.Arrays;
import java.util.Collection;
import java.util.Collections;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.TimeUnit;

/**
 * 功能说明：自定义的ConcurrentMapCacheManager，新增超时时间和最大存储限制
 * 作者：liuxing(2015-04-13 18:44)
 */
public class ConcurrentMapCacheManager implements CacheManager {

    /**
     * 过期时间，秒（自定义）
     */
    private long exp = 1800;
    /**
     * 最大存储数量 （自定义）
     */
    private long maximumSize = 1000;

    public void setExp(long exp) {
        this.exp = exp;
    }

    public void setMaximumSize(long maximumSize) {
        this.maximumSize = maximumSize;
    }

    /**
     * 创建一个缓存容器，这个方法改写为使用Guava的Cache
     * @param name
     * @return
     */
    protected Cache createConcurrentMapCache(String name) {
        return new ConcurrentMapCache(name, CacheBuilder.newBuilder().expireAfterWrite(this.exp, TimeUnit.SECONDS)
                                                                     .maximumSize(this.maximumSize)
                                                                     .build()
                                                                     .asMap(), isAllowNullValues());
    }
}
```

#### 初始化
xml风格

```xml
<!-- 启用缓存注解功能，这个是必须的，否则注解不会生效，指定一个默认的Manager，否则需要在注解使用时指定Manager -->
<cache:annotation-driven cache-manager="memoryCacheManager"/>

<!-- 本地内存缓存 -->
<bean id="memoryCacheManager" class="com.dooioo.ky.cache.ConcurrentMapCacheManager" p:maximumSize="2000" p:exp="1800">
    <property name="cacheNames">
        <list>
            <value>kyMemoryCache</value>
        </list>
    </property>
</bean>
```

#### 使用

```java
@Cacheable(value = "kyMemoryCache", key="targetClass + '.' + methodName")
public Map<String, String> queryMobiles(){
    // 业务代码省略
}
```

### 使用Memcached

一般常用的缓存当属memcached了，这个就需要自己实现CacheManager和Cache
注意我实现的Cache里面有做一些定制化操作，比如对key的处理

#### 创建MemcachedCache

```java
import com.dooioo.common.jstl.DyFunctions;
import com.dooioo.commons.Strings;
import com.google.common.base.Joiner;
import net.rubyeye.xmemcached.MemcachedClient;
import net.rubyeye.xmemcached.exception.MemcachedException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cache.Cache;
import org.springframework.cache.support.SimpleValueWrapper;

import java.util.concurrent.TimeoutException;

/**
 * 功能说明：自定义spring的cache的实现，参考cache包实现
 * 作者：liuxing(2015-04-12 13:57)
 */
public class MemcachedCache implements Cache {

    private static final Logger LOGGER = LoggerFactory.getLogger(MemcachedCache.class);

    /**
     * 缓存的别名
     */
    private String name;
    /**
     * memcached客户端
     */
    private MemcachedClient client;
    /**
     * 缓存过期时间，默认是1小时
     * 自定义的属性
     */
    private int exp = 3600;
    /**
     * 是否对key进行base64加密
     */
    private boolean base64Key = false;
    /**
     * 前缀名
     */
    private String prefix;

    @Override
    public String getName() {
        return name;
    }

    @Override
    public Object getNativeCache() {
        return this.client;
    }

    @Override
    public ValueWrapper get(Object key) {
        Object object = null;
        try {
            object = this.client.get(handleKey(objectToString(key)));
        } catch (TimeoutException e) {
            LOGGER.error(e.getMessage(), e);
        } catch (InterruptedException e) {
            LOGGER.error(e.getMessage(), e);
        } catch (MemcachedException e) {
            LOGGER.error(e.getMessage(), e);
        }

        return (object != null ? new SimpleValueWrapper(object) : null);
    }

    @Override
    public <T> T get(Object key, Class<T> type) {
        try {
            Object object = this.client.get(handleKey(objectToString(key)));
            return (T) object;
        } catch (TimeoutException e) {
            LOGGER.error(e.getMessage(), e);
        } catch (InterruptedException e) {
            LOGGER.error(e.getMessage(), e);
        } catch (MemcachedException e) {
            LOGGER.error(e.getMessage(), e);
        }

        return null;
    }

    @Override
    public void put(Object key, Object value) {
        if (value == null) {
//            this.evict(key);
            return;
        }

        try {
            this.client.set(handleKey(objectToString(key)), exp, value);
        } catch (TimeoutException e) {
            LOGGER.error(e.getMessage(), e);
        } catch (InterruptedException e) {
            LOGGER.error(e.getMessage(), e);
        } catch (MemcachedException e) {
            LOGGER.error(e.getMessage(), e);
        }
    }

    @Override
    public ValueWrapper putIfAbsent(Object key, Object value) {
        this.put(key, value);
        return this.get(key);
    }

    @Override
    public void evict(Object key) {
        try {
            this.client.delete(handleKey(objectToString(key)));
        } catch (TimeoutException e) {
            LOGGER.error(e.getMessage(), e);
        } catch (InterruptedException e) {
            LOGGER.error(e.getMessage(), e);
        } catch (MemcachedException e) {
            LOGGER.error(e.getMessage(), e);
        }
    }

    @Override
    public void clear() {
        try {
            this.client.flushAll();
        } catch (TimeoutException e) {
            LOGGER.error(e.getMessage(), e);
        } catch (InterruptedException e) {
            LOGGER.error(e.getMessage(), e);
        } catch (MemcachedException e) {
            LOGGER.error(e.getMessage(), e);
        }
    }

    public void setName(String name) {
        this.name = name;
    }

    public MemcachedClient getClient() {
        return client;
    }

    public void setClient(MemcachedClient client) {
        this.client = client;
    }

    public void setExp(int exp) {
        this.exp = exp;
    }

    public void setBase64Key(boolean base64Key) {
        this.base64Key = base64Key;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    /**
     * 处理key
     * @param key
     * @return
     */
    private String handleKey(final String key) {
        if (base64Key) {
            return Joiner.on(EMPTY_SEPARATOR).skipNulls().join(this.prefix, DyFunctions.base64Encode(key));
        }

        return Joiner.on(EMPTY_SEPARATOR).skipNulls().join(this.prefix, key);
    }

    /**
     * 转换key，去掉空格
     * @param object
     * @return
     */
    private String objectToString(Object object) {
        if (object == null) {
            return null;
        } else if (object instanceof String) {
            return Strings.replace((String) object, " ", "_");
        } else {
            return object.toString();
        }
    }

    private static final String EMPTY_SEPARATOR = "";

}
```

#### 创建MemcachedCacheManager

继承AbstractCacheManager

```java
import org.springframework.cache.Cache;
import org.springframework.cache.support.AbstractCacheManager;

import java.util.Collection;

/**
 * 功能说明：memcachedCacheManager
 * 作者：liuxing(2015-04-12 15:13)
 */
public class MemcachedCacheManager extends AbstractCacheManager {

    private Collection<Cache> caches;

    @Override
    protected Collection<? extends Cache> loadCaches() {
        return this.caches;
    }

    public void setCaches(Collection<Cache> caches) {
        this.caches = caches;
    }

    public Cache getCache(String name) {
        return super.getCache(name);
    }

}
```

#### 初始化
```xml
<!-- 启用缓存注解功能，这个是必须的，否则注解不会生效，指定一个默认的Manager，否则需要在注解使用时指定Manager -->
<cache:annotation-driven cache-manager="cacheManager"/>

<!-- memcached缓存管理器 -->
<bean id="cacheManager" class="com.dooioo.ky.cache.MemcachedCacheManager">
    <property name="caches">
        <set>
            <bean class="com.dooioo.ky.cache.MemcachedCache" p:client-ref="ky.memcachedClient" p:name="kyAreaCache" p:exp="86400"/>
            <bean class="com.dooioo.ky.cache.MemcachedCache" p:client-ref="ky.memcachedClient" p:name="kyOrganizationCache" p:exp="3600"/>
        </set>
    </property>
</bean>
```


#### 使用
```java
@Cacheable(value = "kyAreaCache", key="targetClass + '.' + methodName + '.' + #areaId")
public KyArea findById(String areaId) {
    // 业务代码省略
}
```


## 更多

更多复杂的使用场景和注解语法请自行谷歌！

**参考**
http://docs.spring.io/spring/docs/4.1.x/spring-framework-reference/html/cache.html

http://www.cnblogs.com/rollenholt/p/4202631.html

http://jinnianshilongnian.iteye.com/blog/2001040