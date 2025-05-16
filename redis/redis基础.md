# 五种基本数据类型



## 常用通用命令

1. keys  名字表达式   *表示任意多个字符  ？表示一个字符
2. del  名字  删除一个指定的key 可以删除多个key
3. exists 判断某个key是否存在
4. expire  



## 字符串String类型

value根据类型不同可以分成三类

- string: 普通字符串
- int: 整数类型,可以做自增自减操作
- float: 浮点类型. 也可以做自增自减

string常见命令:

1. set 添加或修改已经存在的一个String类型的键值对
2. get: 根据key获取String类型的value
3. mset : 批量添加多个String类型键值对
4. mget : 根据key 获取多个value值
5. incr : 根据让一个整形key 自增1  注意(浮点数不可以!!!)
6. incrby : 让一个整数key自增并指定步长    可以为负数
7. incrbyfloat: 让一个浮点数key自增并指定步长
8. setnx 设置一个String键值对 前提是这个key不存在
9. setex 设置一个String键值对,并指定有效期

## key的层级

项目名:业务名:类型名:字段名

例如 s-pay-mall:login: id:1

## hash类型

常用命令:

- hset key filed value   添加或修改hash类型key的filed的值
- hget key filed 根据key 和filed获取值
- hmget 批量获取一个hash中多个filed对应的value
- hgetall 获取一个hash中所有filed value键值对
- hkeys 获取一个hash中所有filed
- hvals 获取一个hash中所有values
- hincrby key filed increment 让一个hash中的filed自增并指定步长
- hsetnx 添加一个hash类型key的filed值,前提是这个filed不存在

## list类型

特点

1. 有序
2. 元素可以重复
3. 插入和删除快
4. 查询速度一般

常用命令:

1. lpush key value  向key列表左侧插入一个或多个元素
2. lpop key 移除并返回key列表左侧的元素 可以后面跟数字指定移除元素个数 默认是1 ,如果列表没有元素返回nil
3. rpush …… 同理
4. rpop 同理
5. lrange key start end 指定key 和开始结束的角标,按顺序返回元素,从左边开始

## set类型

特征：

1. 无序
2. 不可重复
3. 查找快

常用命令：

1. sadd key value 向set中添加一个或多个元素
2. srem key value 移除set中的指定元素可以多个
3. scard key 返回set中元素个数
4. sismember key value 判断set中是否有指定元素
5. smembers key 获取set中所有元素
6. sinter key1 key2 获取两个set集合的交集
7. sdiff key1 key2 获取差集
8. sunion 获取并集

## sortedset类型

特征：

1. 可排序
2. 元素不重复
3. 查询速度快

![image-20250428130644274](image-20250428130644274.png)

# Java客户端

## jedis



1. 引包

   ```xml
   <dependency>
       <groupId>redis.clients</groupId>
       <artifactId>jedis</artifactId>
       <version>3.9.0</version>
   </dependency>
   ```

2. 连接数据库

   ```java
   jedis = new Jedis("47.94.214.84", 6379);
   jedis.auth("123456");
   jedis.select(0);
   ```

3. 运行

   ```java
   jedis.set("name4","ywz");
   String name = jedis.get("name3");
   System.out.println(name);
   ```





# Redis中级



## 缓存穿透

问题：有人恶意制造大量请求redis缓存和mysql数据库中都没有的数据 这时因为MySQL中没有这样的数据所以也不会缓存到redis中 ，请求都会达到数据库中 造成数据库拥堵，性能降低

解决方案：对于请求redis和MySQL都没有的数据，在第一次到达时为该数据设置一个null值，并设置一个比较短的过期时间

并加入判断逻辑，使第二次及以后的请求不会达到数据库中

## 缓存雪崩

问题：由于redis缓存中大量key突然集体失效或redis宕机导致请求大量到达数据库，造成拥堵，

解决方案：

1. 为redis缓存设置随机的过期时间防止集体大量失效
2. redis集群 //TODO

## 缓存击穿

问题：一个被时刻大量访问的资源在redis中的缓存失效，查询数据的操作一般是先查redis中 如果没有再查数据库然后将数据写入缓存。由于写入缓存和查数据库这一步耗时比查redis慢许多，导致一瞬间有许多请求到达redis时发现没有该缓存

然后都去数据库中查找，导致数据库堵塞

解决方案：

1. 对查询数据库这一步操作设置一个锁，确保同一时间只能有一个请求到达数据库更新缓存，
   好处是能解决缓存击穿问题，坏处是其他请求都在等待

2. 逻辑过期 在缓存数据的同时不显示设置过期时间，在缓存中加一个ttl代表逻辑过期时间，当时间超过逻辑过期时间时，
   依然会争抢锁 然后只有一个请求操作数据库更新缓存，但与第一种不同的是，所有请求不会等待而是直接返回旧数据

   包括抢到锁的请求它会再开一个线程去操作数据库，它本身会直接返回旧数据
   好处是 不会存在请求都在等待的情况提升用户体验，坏处是 丧失了一致性







# 用redis实现分布式锁 version 1.0

通过setnx命令  只有在key不存在时才能设置

分布式锁解决多节点 多线程并发请求服务器时导致同一用户购买单一商品多个



当操作大致为一个 “查询——判断——修改” 流程时，

在多线程环境下，往往要考虑 是否会有线程 在判断 和修改之间插入 造成bug



在查询库存，判断库存，修改库存处加锁

在查询用户id和订单id是否同步出现以及后续判断加锁



redisson实现可重入锁机制

![image-20250515123628134](image-20250515123628134.png)