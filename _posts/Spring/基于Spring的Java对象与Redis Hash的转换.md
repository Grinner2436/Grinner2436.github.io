---
layout: post
title: 基于Spring的Java对象与Redis Hash的转换
categories:
  - Spring
tags:
  - 《Spring Data》
note: 基于Spring的Java对象与Redis Hash的转换
---

# 基于Spring的Java对象与Redis Hash的转换
## Spring Data Redis把Java对象转为Redis Hash的三种方式
*   Using `HashMapper` and `HashOperations`

*   Using `Redis Repositories`
* 
*   Direct mapping, by using `HashOperations` and a `serializer`

## 一、使用`HashMapper` 和`HashOperations`
1. HashMapper的作用是将对象转换为Map<K, V>以及转换回来。每个类转换的方式不一样，转出的结果也不一样。以下是HashMapper接口的三个实现：

*   `org.springframework.data.redis.hash.ObjectHashMapper`

*  `Jackson2HashMapper`

*   `BeanUtilsHashMapper`

有些HashMapper在将对象映射为Hash的时候，会将class信息，单独存储为一个键值对，以便识别对象类型。所以这些HashMapper()可以没有泛型。不同Mapper使用的键不同。  
```
ObjectMapper：`_class -> com.ck.redis.Person`， 
Jackson2HashMapper：`@class -> com.ck.redis.Person`，
```
2. HashOperations是RedisTemplate.opsForHash()的返回值，用于和Redis通信，使用配置的序列化处理器来将任意类型转换成byte[],存储到Redis里就全是byte[]。这个类会进行以下转换过程：
```
@Override 
public void putAll(K key, Map<? extends HK, ? extends HV> m) {

   if (m.isEmpty()) {
      return;
   }

   byte[] rawKey = rawKey(key);

   Map<byte[], byte[]> hashes = new LinkedHashMap<>(m.size());

   for (Map.Entry<? extends HK, ? extends HV> entry : m.entrySet()) {
      hashes.put(rawHashKey(entry.getKey()), rawHashValue(entry.getValue()));
   }

   execute(connection -> {
      connection.hMSet(rawKey, hashes);
      return null;
   }, true);
}
```
关键点：
（1）使用rawKey方法，将Redis键转化为byte[],这个值是通过RedisTemplate.setKeySerializer()方法配置的。

（2）使用rawHashKey方法，将Hash的键转化为byte[],这个值是通过RedisTemplate.setHashKeySerializer()方法配置的。

（3）使用rawHashValue方法，将Hash的值转化为byte[],这个值是通过RedisTemplate.setHashValueSerializer()方法配置的。


3. 二者结合，可以转 + 存。HashMapper的作用是对象转Map，所以实际上决定的是Map长什么样。而存到Redis里长什么样，是由序列化决定的。


### ObjectHashMapper
1. 类源码注释：HashMapper based on MappingRedisConverter. Supports nested properties and simple types like String.
2. ObjectHashMapper implements HashMapper<Object, byte[], byte[]>，转换出来是一个Map<byte[], byte[]>

```
HashOperations<String, byte[], byte[]> hashOperations = redisTemplate.opsForHash();
Person person = new Person("first","last");
HashMapper<Object, byte[], byte[]> mapper = new ObjectHashMapper();
Map<byte[], byte[]> mappedHash = mapper.toHash(person);
hashOperations.putAll("person", mappedHash);
```

### Jackson2HashMapper
1. 类源码注释：ObjectMapper based HashMapper implementation that allows flattening. 
2. Jackson2HashMapper implements HashMapper<Object, String, Object>，转换出来是一个Map<String,Object>
3. 使用Jackson2HashMapper需要单独引入依赖，因为spring-boot-starter-data-redis没有带相关的依赖。[FasterXML Jackson](https://github.com/FasterXML/jackson)

4. 它可以将嵌套类映射为json或者扁平化的key，这是一个构造函数的入参。
5. 它可以使用指定的com.fasterxml.jackson.databind.ObjectMapper来映射值,是一个构造函数的入参。

### BeanUtilsHashMapper
1. 类源码注释：HashMapper based on Apache Commons BeanUtils project. Does NOT supports nested properties.
2.  BeanUtilsHashMapper<T> implements HashMapper<T, String, String>，转换出来是一个Map<String,String>，转换出来没有Class信息，因为构造函数需要一个Class入参。
3. 使用BeanUtilsHashMapper需要单独引入依赖，因为spring-boot-starter-data-redis没有带相关的依赖。org.apache.commons.beanutils.BeanUtils。

## 二、使用`Redis Repositories`
1. Redis Repositories require at least Redis Server version 2.8.0 and do not work with transactions. 

2. 使用@RedisHash("people")标注model以后，可以像定义@Entity一样存取Redis实体。
3. public  interface PersonRepository extends CrudRepository<Person, String> { }
```
@RedisHash("people")  
public  class Person { 
  @Id String id; 
  String firstname; 
  String lastname; 
  Address address; 
}

public  interface PersonRepository extends CrudRepository<Person, String> {

}

@Configuration  
@EnableRedisRepositories  
public  class ApplicationConfig {
}

```

## 三、使用`HashOperations` and a `serializer`
*   `JdkSerializationRedisSerializer`, which is used by default for `RedisCache` and `RedisTemplate`.

*   the `StringRedisSerializer`.

* `OxmSerializer` for Object/XML mapping through Spring `OXM` support .
* `Jackson2JsonRedisSerializer` for   `JSON`
* `GenericJackson2JsonRedisSerializer` for  `JSON`
### 
1. 对象会被转成JSON，{“asdf”:“sdaf”}
2. String会被转成String，“psers”



