# 线程的状态

## 操作系统层面的线程五种状态



### **1. 新建状态（New）**

- **定义**：线程被创建但尚未启动（未分配系统资源）。
- **触发条件**：程序调用 `new Thread()` 或类似创建线程的API。
- **转换**：只能通过调用 `start()` 进入 **就绪状态**。

------

### **2. 就绪状态（Ready）**

- **定义**：线程已准备好运行，等待操作系统调度器分配CPU时间片。
- **触发条件**：
  - 新建线程调用 `start()`。
  - 运行状态的线程时间片用完或被抢占（如更高优先级线程出现）。
  - 阻塞状态的线程等待的资源已就绪（如I/O完成、锁释放）。
- **特点**：线程在就绪队列中排队，等待被调度。

------

### **3. 运行状态（Running）**

- **定义**：线程正在CPU上执行代码。
- **触发条件**：调度器从就绪队列中选择该线程分配CPU时间片。
- **转换**：
  - ➔ **就绪状态**：时间片用完或被更高优先级线程抢占。
  - ➔ **阻塞状态**：主动等待资源（如发起I/O请求、尝试获取锁失败）。

------

### **4. 阻塞状态（Blocked/Waiting）**

- **定义**：线程因等待外部事件（如I/O、锁、信号量）而暂停运行，**主动释放CPU**。
- **触发条件**：
  - 发起阻塞式系统调用（如读取磁盘、网络数据）。
  - 请求获取已被其他线程持有的锁（如`synchronized`锁）。
  - 调用 `wait()`、`join()` 等方法主动让出CPU。
- **转换**：
  - ➔ **就绪状态**：等待的事件完成（如I/O返回、锁被释放、收到通知）。

------

### **5. 终止状态（Terminated）**

- **定义**：线程执行完毕或异常退出，释放所有资源（如内存、文件句柄）。
- **触发条件**：
  - 线程正常执行完 `run()` 方法。
  - 线程被强制终止（如调用 `stop()`，但此方法已废弃，不推荐使用）。
  - 未捕获的异常导致线程意外终止。

```plaintext
新建（New） 
  ↓ start()
就绪（Ready） ←——→ 运行（Running）
  ↑    |              ↓ 等待资源
  |    ← 时间片用完 —— |
  |                   ↓
  —— 资源就绪 ——→ 阻塞（Blocked）
                    ↓ 资源就绪
终止（Terminated）
```



## java层面的线程六种状态

### **1. `NEW`（新建状态）**

- **定义**：线程对象已创建（通过 `new Thread()`），但尚未调用 `start()` 方法。
- **特点**：未分配系统资源（如未关联操作系统线程）。
- **转换**：调用 `start()` 后进入 `RUNNABLE` 状态。

------

### **2. `RUNNABLE`（可运行状态）**

- **定义**：线程已启动，可能正在执行或等待 CPU 时间片。
- **对应操作系统状态**：合并了操作系统的 **就绪（Ready）** 和 **运行（Running）** 状态。
- **触发条件**：
  - `start()` 方法调用后。
  - 线程从 `BLOCKED`、`WAITING` 或 `TIMED_WAITING` 状态被唤醒。
- **关键点**：线程可能正在运行，也可能在就绪队列中等待调度。

------

### **3. `BLOCKED`（阻塞状态）**

- **定义**：线程因等待获取 **监视器锁（Monitor Lock）** 而被阻塞。
- **触发场景**：
  - 尝试进入 `synchronized` 代码块或方法，但锁已被其他线程持有。
- **转换**：当锁被释放时，线程会竞争锁，成功则进入 `RUNNABLE` 状态。

------

### **4. `WAITING`（无限期等待状态）**

- **定义**：线程主动进入等待状态，**不占用 CPU 时间片**，直到被其他线程显式唤醒。
- **触发方法**：
  - `Object.wait()`（需先持有锁）。
  - `Thread.join()`（等待目标线程终止）。
  - `LockSupport.park()`（通过并发工具类）。
- **转换**：需要其他线程调用 `notify()`、`notifyAll()` 或 `LockSupport.unpark()` 唤醒。

------

### **5. `TIMED_WAITING`（限期等待状态）**

- **定义**：线程主动进入等待状态，但会在指定时间后自动恢复。
- **触发方法**：
  - `Thread.sleep(long millis)`。
  - `Object.wait(long timeout)`。
  - `Thread.join(long millis)`。
  - `LockSupport.parkNanos()`/`parkUntil()`。
- **转换**：超时时间到或被其他线程唤醒。

------

### **6. `TERMINATED`（终止状态）**

- **定义**：线程执行完成（`run()` 方法结束）或异常终止。
- **特点**：释放所有资源，不可重启。
- **触发条件**：
  - `run()` 正常结束。
  - 未捕获异常导致线程终止。

```
NEW 
  ↓ start()
RUNNABLE 
  |  ↖ 锁可用 / 唤醒 / 超时
  | 锁被占用 → BLOCKED
  | 调用 wait()/join() → WAITING
  | 调用 sleep(timeout)/wait(timeout) → TIMED_WAITING
  ↓ run() 结束
TERMINATED
```

### **关键区别与注意事项**

1. **`BLOCKED` vs `WAITING`**：
   - `BLOCKED`：被动等待锁（如 `synchronized`）。
   - `WAITING`/`TIMED_WAITING`：主动释放锁并等待（如 `wait()`、`join()`）。
2. **`RUNNABLE` 的特殊性**：
   - JVM 不区分“就绪”和“运行”，统一标记为 `RUNNABLE`。
   - 是否实际占用 CPU 由操作系统调度决定。
3. **监控与调试**：
   - 使用 `jstack` 或 `Thread.getState()` 可查看线程状态。
   - 死锁分析时重点关注 `BLOCKED` 状态。

------

### **示例代码**

```java
public class ThreadStateDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            synchronized (ThreadStateDemo.class) {
                try {
                    Thread.sleep(1000); // 进入 TIMED_WAITING
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        System.out.println(thread.getState()); // NEW

        thread.start();
        System.out.println(thread.getState()); // RUNNABLE

        synchronized (ThreadStateDemo.class) {
            thread.join(500); // 主线程等待，thread 在竞争锁时进入 BLOCKED
            System.out.println(thread.getState()); // BLOCKED
        }

        Thread.sleep(1500);
        System.out.println(thread.getState()); // TERMINATED
    }
}
```



## 两者之间的联系

---

### **操作系统与 Java 线程状态的映射关系**
Java 线程状态是 JVM 对操作系统线程状态的一层抽象封装，两者既有联系又有差异。以下是它们的核心对应关系及实际场景分析：

---

#### **1. 新建状态（New）**
- **操作系统**：线程对象已创建，但未分配资源（如未关联内核线程）。
- **Java**：`NEW` 状态，通过 `new Thread()` 创建但未调用 `start()`。
- **联系**：完全一致，均表示线程未启动。

---

#### **2. 就绪（Ready） + 运行（Running） → `RUNNABLE`**
- **操作系统**：分为就绪（等待CPU）和运行（正在执行）。
- **Java**：统一为 `RUNNABLE`，无论线程是否正在占用 CPU。
- **差异**：JVM 不区分是否实际运行，因此 `RUNNABLE` 可能对应操作系统的 **就绪** 或 **运行** 状态。
  
  ```java
  // 示例：Java 中线程启动后始终显示为 RUNNABLE，无论是否获得 CPU
  Thread thread = new Thread(() -> {
      while (true); // 占用 CPU
  });
  thread.start();
  System.out.println(thread.getState()); // 输出 RUNNABLE
  ```

---

#### **3. 阻塞（Blocked） → `BLOCKED` 或 `WAITING`/`TIMED_WAITING`**
操作系统中的 **阻塞** 状态对应 Java 中的多种细分状态，具体取决于阻塞原因：

| **阻塞原因**                | **操作系统状态** | **Java 状态**           | **触发场景**                     |
| --------------------------- | ---------------- | ----------------------- | -------------------------------- |
| 等待锁（如 `synchronized`） | 阻塞             | `BLOCKED`               | 竞争已被其他线程持有的锁         |
| 等待条件（如 `wait()`）     | 阻塞             | `WAITING`               | 调用 `Object.wait()` 或 `join()` |
| 限时等待（如 `sleep()`）    | 阻塞             | `TIMED_WAITING`         | 调用 `Thread.sleep(1000)`        |
| I/O 操作                    | 阻塞             | 仍为 `RUNNABLE`（特殊） | 执行阻塞式系统调用（如读文件）   |

- **关键差异**：Java 明确区分锁竞争（`BLOCKED`）和主动等待（`WAITING`/`TIMED_WAITING`），而操作系统统一视为阻塞。
  ```java
  // 示例：BLOCKED 状态（竞争锁）
  Object lock = new Object();
  new Thread(() -> {
      synchronized (lock) {
          try { Thread.sleep(1000); } catch (InterruptedException e) {}
      }
  }).start();
  
  Thread t2 = new Thread(() -> {
      synchronized (lock) { // 若锁被 t1 持有，t2 进入 BLOCKED
          System.out.println("t2 获取锁");
      }
  });
  t2.start();
  System.out.println(t2.getState()); // 输出 BLOCKED
  ```

---

#### **4. 终止（Terminated）**
- **操作系统**：线程执行完毕，释放所有资源。
- **Java**：`TERMINATED`，线程的 `run()` 方法结束或异常终止。
- **联系**：完全一致，均表示线程生命周期结束。

---

### **状态转换的核心差异**
#### **1. `RUNNABLE` 的广泛性**
- **操作系统**：明确区分就绪（等待调度）和运行（占用 CPU）。
- **Java**：统一为 `RUNNABLE`，无法通过 `getState()` 区分线程是否正在执行。
  ```java
  // 示例：所有就绪和运行的线程在 Java 中均为 RUNNABLE
  Thread t1 = new Thread(() -> {
      // 占用 CPU 运行
  });
  Thread t2 = new Thread(() -> {
      // 在就绪队列等待
  });
  t1.start();
  t2.start();
  System.out.println(t1.getState()); // RUNNABLE
  System.out.println(t2.getState()); // RUNNABLE
  ```

#### **2. I/O 操作的线程状态**
- **操作系统**：线程因等待 I/O 进入阻塞状态。
- **Java**：仍显示为 `RUNNABLE`，因为 JVM 无法感知底层 I/O 阻塞。
  ```java
  // 示例：执行阻塞式 I/O 时，Java 线程状态仍为 RUNNABLE
  Thread ioThread = new Thread(() -> {
      try {
          FileInputStream fis = new FileInputStream("largefile.txt");
          fis.read(); // 阻塞式读取，操作系统线程进入阻塞状态
      } catch (IOException e) {}
  });
  ioThread.start();
  System.out.println(ioThread.getState()); // 输出 RUNNABLE
  ```

---

### **实际应用场景**
#### **1. 死锁检测**
- **操作系统**：通过工具（如 `gdb`）查看线程阻塞原因（如锁竞争）。
- **Java**：通过 `jstack` 分析线程状态：
  - 若多个线程处于 `BLOCKED`，可能发生锁竞争死锁。
  - 若线程处于 `WAITING`，需检查 `wait()`/`notify()` 逻辑。

#### **2. 性能优化**
- **高 `BLOCKED` 状态线程**：优化锁粒度或使用无锁数据结构。
- **大量 `TIMED_WAITING`**：检查超时设置是否合理，避免无效等待。

---

### **总结**
- **操作系统状态**：是底层线程调度的真实体现。
- **Java 线程状态**：是 JVM 对操作系统状态的抽象封装，更贴近开发者视角。
- **核心联系**：`BLOCKED`、`WAITING`、`TIMED_WAITING` 最终映射到操作系统的阻塞状态，但 Java 通过细分状态帮助开发者更精准定位问题。



# synchronized



## 锁加在静态方法和普通方法的区别

在 Java 中，`synchronized` 修饰普通方法和静态方法的锁机制有本质区别，主要体现在 **锁的作用域** 和 **并发控制范围** 上。以下是详细对比：

---

### **1. 锁普通方法（实例方法）**
- **锁对象**：当前实例对象（`this`）。
- **并发范围**：同一实例的同步方法互斥，不同实例的同步方法互不影响。
- **适用场景**：保护实例变量（非静态成员变量）的线程安全。

#### **示例代码**：
```java
public class Example {
    private int count = 0;

    // 锁的是当前实例（this）
    public synchronized void increment() {
        count++;
    }
}
```

#### **执行效果**：
- 线程 A 调用 `obj1.increment()`，线程 B 调用 `obj1.increment()`：**串行执行**（竞争同一把锁）。
- 线程 A 调用 `obj1.increment()`，线程 B 调用 `obj2.increment()`：**并行执行**（不同实例的锁互不干扰）。

---

### **2. 锁静态方法**
- **锁对象**：类的 `Class` 对象（如 `Example.class`）。
- **并发范围**：所有实例的静态同步方法互斥，且与非静态同步方法无关。
- **适用场景**：保护静态变量（类成员变量）的线程安全。

#### **示例代码**：
```java
public class Example {
    private static int staticCount = 0;

    // 锁的是 Example.class
    public static synchronized void staticIncrement() {
        staticCount++;
    }
}
```

#### **执行效果**：
- 线程 A 调用 `Example.staticIncrement()`，线程 B 调用 `Example.staticIncrement()`：**串行执行**（共享同一把类锁）。
- 线程 A 调用 `obj1.staticIncrement()`，线程 B 调用 `obj2.staticIncrement()`：**串行执行**（所有实例共享类锁）。
- 线程 A 调用 `obj1.increment()`（普通同步方法），线程 B 调用 `Example.staticIncrement()`：**并行执行**（锁不同，互不影响）。

---

### **3. 核心区别总结**
| **特性**       | **普通同步方法**           | **静态同步方法**           |
| -------------- | -------------------------- | -------------------------- |
| **锁对象**     | 当前实例（`this`）         | 类的 `Class` 对象          |
| **保护数据**   | 实例变量（非静态成员变量） | 静态变量（类成员变量）     |
| **并发粒度**   | 实例级别                   | 类级别（全局）             |
| **跨实例影响** | 不同实例的同步方法不互斥   | 所有实例的静态同步方法互斥 |

---

### **4. 关键注意事项**
#### **(1) 锁对象隔离性**
- 普通方法锁和静态方法锁是两把不同的锁，**不会互相阻塞**：
  ```java
  public class Example {
      public synchronized void methodA() {}  // 锁 this
      public static synchronized void methodB() {} // 锁 Example.class
  }
  
  // 线程1调用 obj.methodA()，线程2调用 Example.methodB()：并行执行。
  ```

#### **(2) 混合使用风险**
若同时操作实例变量和静态变量，需显式指定锁对象：
```java
public class Example {
    private static int staticCount;
    private int instanceCount;

    // 错误：普通同步方法只能保护 instanceCount，无法保护 staticCount
    public synchronized void unsafeMethod() {
        staticCount++; // 仍需类锁保护
    }

    // 正确：显式使用类锁保护静态变量
    public void safeMethod() {
        synchronized (Example.class) {
            staticCount++;
        }
    }
}
```

#### **(3) 性能影响**
- **普通方法锁**：粒度较细，适用于高并发场景（不同实例可并行）。
- **静态方法锁**：粒度较粗，容易成为性能瓶颈（全局串行）。

---

### **5. 扩展：代码块锁**
`synchronized` 还可显式指定锁对象，灵活控制同步范围：
```java
public class Example {
    private final Object lock = new Object();

    public void method() {
        synchronized (lock) {  // 显式指定锁对象
            // ...
        }
    }
}
```

---

### **总结**
- **普通同步方法**：实例级别的锁，保护实例变量。
- **静态同步方法**：类级别的锁，保护静态变量。
- **关键区别**：锁对象不同，导致并发控制的范围不同。







# 想到的一个小问题：spring boot应用中是否要在进行增删改操作时加锁

在Spring Boot程序中调用Mapper层进行增删改操作时，是否需要外部加锁取决于具体业务场景和数据一致性需求，以下是详细分析：

### **1. 不需要外部加锁的典型场景**
#### **（1）单行数据的原子操作**
- **数据库行锁保证**：大多数数据库（如MySQL InnoDB）在执行写操作（如`UPDATE`、`DELETE`）时会自动对目标行加**行级锁**，防止其他事务同时修改同一行。
- **示例**：
  ```sql
  UPDATE account SET balance = balance - 100 WHERE id = 1;
  ```
  无需应用层锁，数据库自动处理并发。

#### **（2）幂等性设计**
- **天然防重**：若操作本身幂等（如根据唯一键覆盖写入），重复执行不会影响结果。
- **示例**：
  ```java
  // 插入或更新（存在则覆盖）
  userMapper.upsertUser(user); 
  ```

#### **（3）事务隔离级别控制**
- **依赖数据库隔离级别**：通过Spring的`@Transactional`设置合适的事务隔离级别（如`READ_COMMITTED`），避免脏读等问题。

---

### **2. 需要外部加锁的场景**
#### **（1）跨多步骤的非原子操作**
- **竞态条件风险**：若业务逻辑需先查询再更新，可能因并发导致数据不一致。
- **示例**：
  ```java
  // 问题代码：检查-更新存在竞态条件
  public void unsafeUpdate(Long id) {
      Integer current = userMapper.selectValue(id);
      if (current > 0) {
          userMapper.updateValue(id, current - 1); // 可能被其他线程覆盖
      }
  }
  ```
  **解决方案**：
  - **悲观锁**：使用`SELECT ... FOR UPDATE`锁定记录。
    ```java
    @Transactional
    public void safeUpdateWithPessimisticLock(Long id) {
        User user = userMapper.selectForUpdate(id); // SELECT ... FOR UPDATE
        if (user.getValue() > 0) {
            userMapper.updateValue(id, user.getValue() - 1);
        }
    }
    ```
  - **乐观锁**：通过版本号或时间戳控制。
    ```java
    // 实体类添加版本字段
    public class User {
        private Long id;
        private Integer value;
        private Integer version; // 乐观锁版本
    }
    
    // Mapper XML
    <update id="updateWithVersion">
        UPDATE user 
        SET value = #{value}, version = version + 1 
        WHERE id = #{id} AND version = #{version}
    </update>
    ```

#### **（2）分布式环境下的全局资源竞争**
- **单机锁失效**：应用部署多实例时，Java内置锁（如`synchronized`）仅限单JVM有效。
- **解决方案**：使用分布式锁（如Redis RedLock、ZooKeeper）。
  ```java
  public void distributedSafeUpdate(Long id) {
      String lockKey = "user_lock:" + id;
      try {
          // 尝试获取分布式锁（伪代码）
          boolean locked = redisLock.acquire(lockKey, 10, TimeUnit.SECONDS);
          if (locked) {
              User user = userMapper.selectById(id);
              userMapper.update(user);
          }
      } finally {
          redisLock.release(lockKey);
      }
  }
  ```

#### **（3）高并发秒杀场景**
- **超卖问题**：大量请求同时扣减库存，需强一致性保证。
- **解决方案**：
  - **数据库悲观锁**：`SELECT ... FOR UPDATE`。
  - **Redis原子操作**：利用`DECR`或Lua脚本保证原子性。
  - **消息队列削峰**：通过异步处理缓冲并发请求。

---

### **3. 最佳实践总结**
| **场景**                     | **是否需要锁** | **推荐方案**                        |
| ---------------------------- | -------------- | ----------------------------------- |
| 单行原子操作（如直接UPDATE） | 否             | 依赖数据库行锁                      |
| 先查后改的非原子操作         | 是             | 悲观锁（SELECT FOR UPDATE）或乐观锁 |
| 分布式环境资源竞争           | 是             | 分布式锁（Redis/ZooKeeper）         |
| 幂等性操作（如覆盖写入）     | 否             | 依赖唯一索引或幂等设计              |

---

### **4. 示例代码对比**
#### **不安全实现（无锁）**
```java
public void unsafeDeduct(Long userId, int amount) {
    User user = userMapper.selectById(userId);
    if (user.getBalance() >= amount) {
        user.setBalance(user.getBalance() - amount);
        userMapper.updateById(user);
    }
}
```
**风险**：并发时多个线程可能同时读到相同余额，导致超额扣减。

#### **安全实现（数据库悲观锁）**
```java
@Transactional
public void safeDeductWithPessimisticLock(Long userId, int amount) {
    User user = userMapper.selectForUpdate(userId); // 加行锁
    if (user.getBalance() >= amount) {
        user.setBalance(user.getBalance() - amount);
        userMapper.updateById(user);
    }
}
```

#### **安全实现（乐观锁）**
```java
public void safeDeductWithOptimisticLock(Long userId, int amount) {
    User user = userMapper.selectById(userId);
    int newBalance = user.getBalance() - amount;
    if (newBalance >= 0) {
        int rows = userMapper.updateBalance(userId, newBalance, user.getVersion());
        if (rows == 0) {
            throw new RetryException("并发冲突，请重试");
        }
    }
}
```

---

### **结论**
- **优先依赖数据库锁**：在单行原子操作中，数据库的行级锁足够高效且安全。
- **复杂逻辑需显式控制**：对于多步骤操作或分布式环境，合理选择悲观锁、乐观锁或分布式锁。
- **避免过度加锁**：锁会降低并发性能，仅在必要时使用，并尽量缩小锁范围。