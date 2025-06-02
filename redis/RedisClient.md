# RedissonClient 常用 API 详解

Redisson 是一个强大的 Java Redis 客户端，提供了丰富的分布式对象和服务。下面我将详细解释 RedissonClient 的常用 API：

## 一、基础对象操作

### 1. Bucket（通用对象桶）
```java
// 获取或创建 Bucket
RBucket<String> bucket = redissonClient.getBucket("myBucket");

// 设置值（带过期时间）
bucket.set("value", 10, TimeUnit.SECONDS);

// 获取值
String value = bucket.get();

// 获取并设置新值
String prevValue = bucket.getAndSet("newValue");

// 比较并设置（CAS）
boolean updated = bucket.compareAndSet("expected", "newValue");

// 删除
bucket.delete();
```

### 2. BinaryStream（二进制流）
```java
RBinaryStream stream = redissonClient.getBinaryStream("myStream");

// 写数据
OutputStream os = stream.getOutputStream();
os.write("Hello".getBytes());
os.close();

// 读数据
InputStream is = stream.getInputStream();
byte[] data = new byte[is.available()];
is.read(data);
is.close();
```

## 二、分布式集合

### 1. Map（分布式映射）
```java
RMap<String, String> map = redissonClient.getMap("myMap");

// 基本操作
map.put("key", "value");
String val = map.get("key");
map.remove("key");

// 原子操作
map.fastPut("key", "value"); // 非阻塞
map.fastRemove("key");       // 非阻塞

// 锁保护操作
map.compute("key", (k, v) -> v + "-modified");

// 加载映射（支持 LRU 缓存）
RMapCache<String, String> cacheMap = redissonClient.getMapCache("cacheMap");
cacheMap.put("key", "value", 10, TimeUnit.MINUTES); // 带过期时间
```

### 2. Set（分布式集合）
```java
RSet<String> set = redissonClient.getSet("mySet");

// 基本操作
set.add("item");
set.remove("item");
boolean contains = set.contains("item");

// 高级操作
Set<String> allItems = set.readAll();

// 分布式锁保护操作
set.removeIf(item -> item.startsWith("A"));
```

### 3. List（分布式列表）
```java
RList<String> list = redissonClient.getList("myList");

// 基本操作
list.add("item");
list.get(0);
list.remove(0);

// 阻塞操作（分布式队列）
RBlockingQueue<String> queue = redissonClient.getBlockingQueue("myQueue");
queue.offer("msg"); // 入队
String msg = queue.take(); // 阻塞出队
```

### 4. SortedSet（分布式有序集合）
```java
RSortedSet<String> sortedSet = redissonClient.getSortedSet("mySortedSet");

// 添加元素（带分数）
sortedSet.add("item", 100);

// 获取范围
Collection<String> topItems = sortedSet.valueRange(0, 9); // 前10名
```

## 三、分布式锁与同步器

### 1. Lock（可重入锁）
```java
RLock lock = redissonClient.getLock("resourceLock");

// 基本加锁
lock.lock();
try {
    // 临界区代码
} finally {
    lock.unlock();
}

// 尝试加锁（带超时）
boolean acquired = lock.tryLock(10, 30, TimeUnit.SECONDS);
if (acquired) {
    try {
        // 临界区代码
    } finally {
        lock.unlock();
    }
}
```

### 2. ReadWriteLock（读写锁）
```java
RReadWriteLock rwLock = redissonClient.getReadWriteLock("docLock");

// 写锁
RWriteLock writeLock = rwLock.writeLock();
writeLock.lock();
try {
    // 写操作
} finally {
    writeLock.unlock();
}

// 读锁
RReadLock readLock = rwLock.readLock();
readLock.lock();
try {
    // 读操作
} finally {
    readLock.unlock();
}
```

### 3. Semaphore（信号量）
```java
RSemaphore semaphore = redissonClient.getSemaphore("resourceSemaphore");

// 初始化许可数
semaphore.trySetPermits(5);

// 获取许可
semaphore.acquire(); // 阻塞
boolean acquired = semaphore.tryAcquire(3, TimeUnit.SECONDS); // 尝试获取

// 释放许可
semaphore.release();
```

### 4. CountDownLatch（闭锁）
```java
RCountDownLatch latch = redissonClient.getCountDownLatch("startLatch");

// 初始化计数
latch.trySetCount(3);

// 等待计数归零
latch.await();

// 在另一个线程中减少计数
latch.countDown();
```

## 四、原子变量

### 1. AtomicLong（原子长整型）
```java
RAtomicLong counter = redissonClient.getAtomicLong("myCounter");

// 基本操作
counter.incrementAndGet();
counter.addAndGet(10);

// 比较并设置
counter.compareAndSet(100, 0);
```

### 2. AtomicDouble（原子双精度）
```java
RAtomicDouble doubleCounter = redissonClient.getAtomicDouble("myDouble");

// 基本操作
doubleCounter.addAndGet(0.5);
```

## 五、发布订阅

### 1. Topic（主题）
```java
RTopic topic = redissonClient.getTopic("newsChannel");

// 添加监听器
int listenerId = topic.addListener(String.class, (channel, msg) -> {
    System.out.println("收到消息: " + msg);
});

// 发布消息
topic.publish("重要通知!");

// 移除监听器
topic.removeListener(listenerId);
```

### 2. PatternTopic（模式主题）
```java
RPatternTopic patternTopic = redissonClient.getPatternTopic("news.*");

// 添加监听器（支持通配符）
patternTopic.addListener(String.class, (pattern, channel, msg) -> {
    System.out.printf("模式 %s 频道 %s: %s%n", pattern, channel, msg);
});
```

## 六、高级数据结构

### 1. Bloom Filter（布隆过滤器）
```java
RBloomFilter<String> bloomFilter = redissonClient.getBloomFilter("userFilter");

// 初始化
bloomFilter.tryInit(100000L, 0.03); // 10万元素，3%误判率

// 添加元素
bloomFilter.add("user@example.com");

// 检查存在
boolean exists = bloomFilter.contains("user@example.com");
```

### 2. HyperLogLog（基数统计）
```java
RHyperLogLog<String> hyperLogLog = redissonClient.getHyperLogLog("uniqueUsers");

// 添加元素
hyperLogLog.add("user1");
hyperLogLog.addAll(Arrays.asList("user2", "user3"));

// 获取基数估计
long count = hyperLogLog.count();
```

### 3. Geospatial（地理空间）
```java
RGeo<String> geo = redissonClient.getGeo("cities");

// 添加位置
geo.add(new GeoEntry(13.361389, 38.115556, "Palermo"),
        new GeoEntry(15.087269, 37.502669, "Catania"));

// 计算距离
Double distance = geo.dist("Palermo", "Catania", GeoUnit.KILOMETERS);

// 半径搜索
List<String> cities = geo.radius(15, 37, 200, GeoUnit.KILOMETERS);
```

## 七、服务相关

### 1. Scheduler（分布式调度器）
```java
RScheduledExecutorService executor = redissonClient.getExecutorService("myScheduler");

// 延迟执行
executor.schedule(() -> System.out.println("Delayed task"), 10, TimeUnit.SECONDS);

// 周期性执行
executor.scheduleAtFixedRate(() -> System.out.println("Periodic task"), 5, 1, TimeUnit.MINUTES);
```

### 2. Remote Service（远程服务）
```java
// 服务端
RRemoteService remoteService = redissonClient.getRemoteService();
remoteService.register(MyServiceInterface.class, new MyServiceImpl());

// 客户端
MyServiceInterface service = remoteService.get(MyServiceInterface.class);
String result = service.doSomething("param");
```

## 八、事务与批量操作

### 1. Transaction（事务）
```java
RTransaction transaction = redissonClient.createTransaction(TransactionOptions.defaults());

try {
    RBucket<String> bucket = transaction.getBucket("txBucket");
    bucket.set("transactionValue");
    
    RMap<String, String> map = transaction.getMap("txMap");
    map.put("key", "value");
    
    transaction.commit();
} catch (Exception e) {
    transaction.rollback();
}
```

### 2. Batch（批处理）
```java
RBatch batch = redissonClient.createBatch(BatchOptions.defaults());

batch.getBucket("batchBucket").setAsync("value");
batch.getMap("batchMap").putAsync("key", "value");
batch.getAtomicLong("batchCounter").incrementAndGetAsync();

BatchResult<?> result = batch.execute();
```

## 九、RedissonClient 管理 API

### 1. 键操作
```java
// 检查键是否存在
boolean exists = redissonClient.getKeys().countExists("key1", "key2") > 0;

// 查找匹配的键
Iterable<String> keys = redissonClient.getKeys().getKeysByPattern("user:*");

// 删除键
long deleted = redissonClient.getKeys().delete("key1", "key2");
```

### 2. 脚本执行
```java
redissonClient.getScript().eval(RScript.Mode.READ_ONLY, 
    "return redis.call('get', KEYS[1])", 
    RScript.ReturnType.VALUE, 
    Collections.singletonList("myKey"));
```

### 3. 关闭客户端
```java
redissonClient.shutdown();
```

## 十、配置与状态

### 1. 配置管理
```java
Config config = redissonClient.getConfig();
System.out.println("Redis server: " + config.useSingleServer().getAddress());
```

### 2. 状态检查
```java
// 检查连接状态
boolean isConnected = !redissonClient.isShutdown() && !redissonClient.isShuttingDown();

// 获取服务器信息
RedisNodes masterNodes = redissonClient.getRedisNodes(RedisNodes.MASTER);
for (RedisMaster node : masterNodes.getMasters()) {
    System.out.println("Master node: " + node.getAddress());
}
```

## 总结

RedissonClient 提供了全面的分布式对象和服务，主要特点包括：

1. **分布式对象**：实现了 Java 常用数据结构（Map、Set、List 等）的分布式版本
2. **分布式服务**：提供分布式锁、信号量、闭锁等同步工具
3. **高级特性**：支持布隆过滤器、基数统计、地理空间等高级数据结构
4. **分布式编程**：支持分布式调度、远程服务调用等高级功能
5. **事务支持**：提供 ACID 事务和批处理操作

与 StringRedisTemplate 相比，RedissonClient 更专注于分布式场景，提供了更丰富的分布式原语和更高层次的抽象，适合构建复杂的分布式系统。而 StringRedisTemplate 更适合基本的键值操作和简单的 Redis 功能使用。

# Redisson 事务保证机制详解

在 Redisson 中保证事务主要通过以下三种机制实现，每种机制都有其特定的使用场景和保证级别：

## 一、Redisson 事务 (RTransaction)

### 基本用法
```java
// 1. 创建事务
RTransaction transaction = redissonClient.createTransaction(
    TransactionOptions.defaults()
        .timeout(30, TimeUnit.SECONDS)  // 事务超时时间
        .responseTimeout(3, TimeUnit.SECONDS)  // 操作响应超时
);

try {
    // 2. 从事务中获取对象
    RBucket<String> bucket = transaction.getBucket("txBucket");
    RMap<String, String> map = transaction.getMap("txMap");
    
    // 3. 事务操作
    bucket.set("transactionValue");
    map.put("key", "value");
    
    // 4. 提交事务
    transaction.commit();
} catch (Exception e) {
    // 5. 回滚事务
    transaction.rollback();
    throw new RuntimeException("Transaction failed", e);
}
```

### 核心特性
1. **ACID 保证**：
   - 原子性：所有操作要么全部成功，要么全部失败
   - 一致性：操作前后数据状态保持一致
   - 隔离性：基于 Redis 的 WATCH/MULTI/EXEC 机制
   - 持久性：取决于 Redis 配置（AOF/RDB）

2. **锁机制**：
   - 自动对涉及的所有键加锁
   - 防止其他客户端修改事务中的数据
   - 锁在事务提交/回滚后自动释放

3. **回滚支持**：
   - 自动撤销所有未提交的操作
   - 内置操作日志记录

## 二、分布式锁 + 批处理

### 适用场景
当需要跨多个非事务对象操作时：

```java
// 1. 获取分布式锁
RLock lock = redissonClient.getLock("resourceLock");
lock.lock(30, TimeUnit.SECONDS);  // 带超时防止死锁

try {
    // 2. 创建批处理
    RBatch batch = redissonClient.createBatch();
    
    // 3. 添加批操作
    batch.getBucket("bucket1").setAsync("value1");
    batch.getMap("map1").putAsync("key", "value");
    batch.getAtomicLong("counter").incrementAndGetAsync();
    
    // 4. 原子执行所有操作
    BatchResult<?> result = batch.execute();
    
    // 5. 验证结果
    if (!result.getResponses().isEmpty()) {
        // 处理结果...
    }
} finally {
    // 6. 释放锁
    lock.unlock();
}
```

### 优势
1. 比事务更灵活（可操作不同类型对象）
2. 减少网络往返次数（一次发送所有命令）
3. 支持异步操作

## 三、Lua 脚本（最高级别的原子性）

### 实现原理
Redis 单线程执行 Lua 脚本，保证原子性：

```java
String script = "local bucketValue = redis.call('get', KEYS[1]); " +
               "if bucketValue == ARGV[1] then " +
               "   redis.call('set', KEYS[2], ARGV[2]); " +
               "   redis.call('incr', KEYS[3]); " +
               "   return 1; " +
               "else " +
               "   return 0; " +
               "end";

// 执行脚本
Long result = redissonClient.getScript().eval(
    RScript.Mode.READ_WRITE,
    script,
    RScript.ReturnType.INTEGER,
    Arrays.asList("bucket1", "map1", "counter"),
    "expectedValue", "newValue"
);

if (result == 1) {
    System.out.println("操作成功");
} else {
    System.out.println("条件不满足，操作未执行");
}
```

### 适用场景
1. 需要复杂条件判断的事务
2. 需要最高性能的原子操作
3. 需要减少网络开销的操作

## 四、事务模式对比

| 特性           | RTransaction | 分布式锁+批处理 | Lua脚本 |
| -------------- | ------------ | --------------- | ------- |
| **原子性**     | ✅            | ✅               | ✅       |
| **隔离性**     | ✅            | ✅               | ✅       |
| **跨对象操作** | 有限支持     | ✅               | ✅       |
| **复杂条件**   | ❌            | ❌               | ✅       |
| **性能**       | 中等         | 高              | 最高    |
| **锁自动管理** | ✅            | ❌               | ✅       |
| **回滚支持**   | ✅            | ❌               | ❌       |
| **学习曲线**   | 低           | 中              | 高      |
| **网络开销**   | 高           | 低              | 最低    |

## 五、最佳实践建议

### 1. 选择合适的事务机制
- 简单操作 → `RTransaction`
- 跨多个键的操作 → **分布式锁+批处理**
- 高性能复杂操作 → **Lua脚本**

### 2. 事务超时设置
```java
TransactionOptions options = TransactionOptions.defaults()
    .timeout(10, TimeUnit.SECONDS)  // 整个事务超时
    .responseTimeout(2, TimeUnit.SECONDS); // 单操作超时
```

### 3. 避免长事务
- 事务执行时间不要超过1秒
- 减少事务中操作的数量
- 复杂事务拆分为多个小事务

### 4. 错误处理策略
```java
try {
    transaction.commit();
} catch (TransactionException e) {
    if (e.getCause() instanceof TransactionTimeoutException) {
        // 处理超时
    } else if (e.getCause() instanceof TransactionRollbackException) {
        // 处理冲突回滚
    }
    transaction.rollback();
}
```

### 5. 监控与重试
```java
int maxRetries = 3;
int attempt = 0;
boolean success = false;

while (attempt < maxRetries && !success) {
    try {
        // 尝试事务操作
        success = true;
    } catch (TransactionConflictException e) {
        attempt++;
        Thread.sleep(50 * attempt); // 指数退避
    }
}
```

## 六、高级事务模式

### 1. XA 分布式事务（跨系统）
```java
// 需要配置JTA事务管理器
RedissonXAResource xaResource = redissonClient.getXAResource();
UserTransaction utx = (UserTransaction)new InitialContext().lookup("...");

utx.begin();
xaResource.start(xid, XAResource.TMNOFLAGS);
// 执行操作...
xaResource.end(xid, XAResource.TMSUCCESS);
xaResource.prepare(xid);
utx.commit();
```

### 2. 反应式事务
```java
RTransactionReactive transaction = redissonReactiveClient.createTransaction(TransactionOptions.defaults());

transaction.getBucket("reactiveBucket").set("value")
    .then(transaction.getMap("reactiveMap").put("key", "value"))
    .then(transaction.commit())
    .onErrorResume(e -> transaction.rollback().then(Mono.error(e)))
    .subscribe();
```

## 七、常见问题解决方案

### 问题1：事务冲突频繁
**解决方案**：
- 减小事务范围
- 使用更细粒度的键
- 实现乐观锁机制
```java
RMap<String, String> map = transaction.getMap("map");
map.putIfAbsent("key", "value", 10, TimeUnit.SECONDS);
```

### 问题2：事务超时
**解决方案**：
```java
TransactionOptions options = TransactionOptions.defaults()
    .timeout(2, TimeUnit.SECONDS)  // 缩短超时
    .retryAttempts(0);             // 禁用重试
```

### 问题3：内存溢出
**解决方案**：
- 限制事务操作数量
- 分批处理大型事务
- 使用流式处理

> **关键提示**：在分布式系统中，事务保证总是存在权衡。Redisson 提供了多种机制，选择哪种取决于您的具体需求：
> - 强一致性需求 → 使用 `RTransaction` 或 Lua 脚本
> - 最终一致性可接受 → 使用分布式锁+批处理
> - 超高吞吐需求 → Lua 脚本