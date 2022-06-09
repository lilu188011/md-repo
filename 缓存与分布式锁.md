## SpringCache使用

### 一、简介

- Spring从3.1开始定义了org.springframework.cache.Cache和org.springframework.cache.CacheManager接口来统一不同的缓存技术；并支持使用JCache（JSR-107）注解简化我们开发；
- Cache接口为缓存的组件规范定义，包含缓存的各种操作集合；Cache接口下Spring提供了各种xxxCache的实现；如RedisCache，EhCacheCache,ConcurrentMapCache等；
- 每次调用需要缓存功能的方法时，Spring会检查检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。
- 使用Spring缓存抽象时我们需要关注以下两点；
  - 确定方法需要被缓存以及他们的缓存策略
  - 从缓存中读取之前缓存存储的数据

### 二、基础概念

![img](https://raw.githubusercontent.com/lilu188011/img-repo/master/70124d3756944de794bdd1975d24979d.jpeg)





#### 1) 引入依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

```

#### 2) 添加配置

自动配置了：

- CacheAutoConfiguration 会导入RedisCacheConfiguration;
- 会自动装配缓存管理器 RedisCacheManager;

手动配置：

```properties
#配置使用redis作为缓存
spring.cache.type=redis

#spring.cache.cache-names=qq,
#毫秒为单位
spring.cache.redis.time-to-live=3600000

#如果指定了前缀就用我们指定的前缀，如果没有就默认使用缓存的名字作为前缀
#spring.cache.redis.key-prefix=CACHE_
spring.cache.redis.use-key-prefix=true

#是否缓存空值，防止缓存穿透
spring.cache.redis.cache-null-values=true

```

3） 常用注解

- @Cacheable ：触发将数据保存到缓存的操作；
- @CacheEvict : 触发将数据从缓存删除的操作；失效模式

```java
//指定删除某个分区下的所有缓存
//存储同一类型的数据，都可以指定成同一个分区，分区名默认就是存储的前缀（配置文件不指定的话）
@CacheEvict(value="category",allEntries = true)
    @Transactional
    @Override
    public void updateCascade(CategoryEntity category) {
        this.updateById(category);
        categoryBrandRelationService.updateCategory(category.getCatId(), category.getName());
    }

```

```java
   //指定分区下哪个key的缓存
@CacheEvict(value = "category",key = "'getLevel1Categorys'")
    @Transactional
    @Override
    public void updateCascade(CategoryEntity category) {
        this.updateById(category);
        categoryBrandRelationService.updateCategory(category.getCatId(), category.getName());
    }

```

- @CachePut ：不影响方法执行更新缓存；双写模式
- @Cacheing：组合以上多个操作；

```java
   @Caching(evict = { @CacheEvict(value = "category",key = "'getLevel1Categorys'"),
            @CacheEvict(value = "category",key = "'getCatelogJson'")})
    @Transactional
    @Override
    public void updateCascade(CategoryEntity category) {
        this.updateById(category);
        categoryBrandRelationService.updateCategory(category.getCatId(), category.getName());
    }

```

- @CacheConfig：在类级别共享缓存的相同配置；

### 三、业务实现

- 开启缓存功能 @EnableCaching

- 只需要使用注解就能完成缓存操作

- 每一个需要缓存的数据我们都来指定要放到哪个名字的缓存【缓存的分区（按照业务类型分】 **@Cacheable({“category”})**

- **@Cacheable** 代表当前方法的结果需要缓存，如果缓存中有，方法都不用调用，如果缓存中没有，会调用方法。最后将方法的结果放入缓存

- 默认行为

  - 如果缓存中有，方法不再调用

  - key是默认生成的:缓存的名字::SimpleKey::

  - 缓存的value值，默认使用jdk序列化机制，将序列化后的数据存到redis中

  - 默认ttl时间是 -1：

  - 自定义操作：key的生成

    - 指定生成缓存的key：key属性指定，接收一个 SpEl

      - @Cacheable(value = {“category”},**key = “‘level1Categorys’”**)或者@Cacheable(value = {“category”},**key = “#root.method.name”**)
      - spel的详细语法见：https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#cache-spel-context

    - 指定缓存的数据的存活时间:配置文档中修改存活时间 ttl，毫秒为单位

      - **spring.cache.redis.time-to-live=60000spring.cache.redis.time-to-live=60000**

    - 将数据保存为json格式: 自定义配置类MyCacheManager

    - ```java
      package com.atguigu.gulimall.product.config;/**
       * @author chenfl
       * @create 2022-03-01-15:48
       */
      
      import org.springframework.boot.autoconfigure.cache.CacheProperties;
      import org.springframework.boot.context.properties.EnableConfigurationProperties;
      import org.springframework.cache.annotation.EnableCaching;
      import org.springframework.context.annotation.Bean;
      import org.springframework.context.annotation.Configuration;
      import org.springframework.data.redis.cache.RedisCacheConfiguration;
      import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
      import org.springframework.data.redis.serializer.RedisSerializationContext;
      import org.springframework.data.redis.serializer.StringRedisSerializer;
      
      /**
       * @author chenfl
       * @description 缓存配置
       * @date 2022/3/1 15:48
       */
      @EnableConfigurationProperties(CacheProperties.class)
      @Configuration
      @EnableCaching
      public class MyCacheConfig {
      
          /**
           * 配置文件的配置没有用上
           * 1. 原来和配置文件绑定的配置类为：@ConfigurationProperties(prefix = "spring.cache")
           * public class CacheProperties
           * <p>
           * 2. 要让他生效，要加上 @EnableConfigurationProperties(CacheProperties.class)
           */
          @Bean
          public RedisCacheConfiguration redisCacheConfiguration(CacheProperties cacheProperties) {
              RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
              // config = config.entryTtl();
              config = config.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()));
              config = config.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
      
              CacheProperties.Redis redisProperties = cacheProperties.getRedis();
              //将配置文件中所有的配置都生效
              if (redisProperties.getTimeToLive() != null) {
                  config = config.entryTtl(redisProperties.getTimeToLive());
              }
              if (redisProperties.getKeyPrefix() != null) {
                  config = config.prefixKeysWith(redisProperties.getKeyPrefix());
              }
              if (!redisProperties.isCacheNullValues()) {
                  config = config.disableCachingNullValues();
              }
              if (!redisProperties.isUseKeyPrefix()) {
                  config = config.disableKeyPrefix();
              }
              return config;
          }
      }
      
      
      ```

    - 原理：CacheAutoConfiguration -> RedisCacheConfiguration-> RedisCacheManager -> 初始化所有的缓存 -> 每个缓存决定使用什么配置-> 如果redisCacheConfiguration有就用自己的，没有就用默认配置 ->想改缓存的配置，只需要给容器中放一个RedisCacheConfiguration即可->就会应用到当前RedisCacheManager管理的所有缓存分区中

    - Spring-Cache的不足之处：

      - 读模式
        - 缓存穿透：查询一个null数据。解决方案：缓存空数据
        - 缓存击穿：大量并发进来同时查询一个正好过期的数据。解决方案：加锁 ? 默认是无加锁的;使用@Cacheable(**sync = true**)来解决击穿问题
        - 缓存雪崩：大量的key同时过期。解决：加随机时间。加上过期时间（spring.cache.redis.time-to-live=3600000）
      - 写模式：（缓存与数据库一致）
        - 读写加锁。
        - 引入Canal,感知到MySQL的更新去更新Redis
        - 读多写多，直接去数据库查询就行

    - 总结

      - 常规数据（读多写少，即时性，一致性要求不高的数据，完全可以使用Spring-Cache）：写模式(只要缓存的数据有过期时间就足够了)

