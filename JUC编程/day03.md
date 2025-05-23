从97集到

# 保护性暂停模式

保护性暂停设计模式（Guarded Suspension）是多线程编程中用于协调线程执行顺序的一种模式，其核心思想是：**当某个条件未满足时，线程主动暂停并等待，直到条件被其他线程修改后唤醒**。这种模式通过避免忙等待（Busy-Waiting）来提高资源利用率，并确保线程间的安全协作。

---

### **核心概念**
1. **条件检查（Guard Condition）**  
   线程在执行关键操作前检查某个条件是否满足（如资源是否可用、数据是否就绪）。
2. **等待与唤醒机制**  
   若条件不满足，线程进入等待状态；当其他线程修改条件后，通过通知机制唤醒等待的线程。

---

### **适用场景**
- **生产者-消费者模型**：消费者等待生产者提供数据。
- **任务队列处理**：工作线程等待新任务到达。
- **资源池管理**：线程等待资源释放后再获取。

---

### **实现步骤（以Java为例）**
#### **1. 使用`synchronized`和`wait()/notify()`**
```java
public class GuardedObject {
    private Object lock = new Object();
    private boolean condition = false;

    public void waitForCondition() throws InterruptedException {
        synchronized (lock) {
            while (!condition) {  // 循环检查条件，防止虚假唤醒
                lock.wait();       // 释放锁并等待
            }
            // 条件满足后执行操作
        }
    }

    public void triggerCondition() {
        synchronized (lock) {
            condition = true;
            lock.notifyAll();      // 唤醒所有等待线程
        }
    }
}
```

#### **2. 使用`ReentrantLock`和`Condition`（更灵活）**
```java
import java.util.concurrent.locks.*;

public class GuardedObjectWithLock {
    private final Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();
    private boolean isReady = false;

    public void await() throws InterruptedException {
        lock.lock();
        try {
            while (!isReady) {     // 循环检查条件
                condition.await(); // 等待
            }
            // 条件满足后执行操作
        } finally {
            lock.unlock();
        }
    }

    public void signal() {
        lock.lock();
        try {
            isReady = true;
            condition.signalAll(); // 唤醒所有等待线程
        } finally {
            lock.unlock();
        }
    }
}
```

---

### **关键要点**
1. **循环检查条件**  
   使用`while`而非`if`检查条件，防止虚假唤醒（Spurious Wakeup）。
   ```java
   while (!condition) {
       wait();
   }
   ```

2. **锁与条件变量的正确使用**  
   - 调用`wait()`或`await()`前必须持有锁。
   - 修改条件后，通过`notify()`/`signal()`通知等待线程。

3. **性能优化**  
   - 使用`Condition`支持多个等待队列（如不同条件）。
   - 优先使用`notifyAll()`或`signalAll()`避免线程饥饿。

---

### **典型应用示例**
#### **生产者-消费者模型**
```java
public class MessageQueue {
    private final Queue<String> queue = new LinkedList<>();
    private final int maxSize;

    public MessageQueue(int maxSize) {
        this.maxSize = maxSize;
    }

    public synchronized void produce(String message) throws InterruptedException {
        while (queue.size() == maxSize) {
            wait(); // 队列满，生产者等待
        }
        queue.add(message);
        notifyAll(); // 唤醒消费者
    }

    public synchronized String consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait(); // 队列空，消费者等待
        }
        String message = queue.poll();
        notifyAll(); // 唤醒生产者
        return message;
    }
}
```

---

### **总结**
| **特性**     | **说明**                                               |
| ------------ | ------------------------------------------------------ |
| **目标**     | 协调多线程执行顺序，避免资源浪费                       |
| **实现方式** | `synchronized`+`wait()/notify()` 或 `Lock`+`Condition` |
| **核心优势** | 减少CPU空转，提升线程协作效率                          |
| **注意事项** | 防止虚假唤醒、正确管理锁、避免死锁                     |

保护性暂停模式通过条件等待与通知机制，实现了线程间的高效协作，是构建健壮多线程应用的基石之一。

## 问题：保护性暂停模式是需要产生结果和消费结果的线程一一对应么

保护性暂停模式（Guarded Suspension）**不要求产生结果和消费结果的线程严格一一对应**，其核心在于通过条件变量的等待与通知机制协调线程间的协作，具体实现方式决定了生产者和消费者之间的匹配关系。以下是详细分析：

---

### **1. 模式的灵活性**
保护性暂停模式的核心是**条件管理**，而非线程绑定。根据设计需求，它可以支持以下场景：
- **一对一**：一个生产者线程对应一个消费者线程。
- **多对一**：多个生产者线程为单个消费者线程提供结果。
- **一对多**：单个生产者线程为多个消费者线程提供结果。
- **多对多**：多个生产者线程与多个消费者线程通过唯一标识符（如请求ID）匹配结果。

---

### **2. 典型实现方式**
#### **(1) 简单的一对一场景**
```java
public class GuardedObject {
    private Object result;
    private final Object lock = new Object();

    // 消费者等待结果
    public Object get() throws InterruptedException {
        synchronized (lock) {
            while (result == null) {
                lock.wait();
            }
            return result;
        }
    }

    // 生产者设置结果并通知
    public void complete(Object value) {
        synchronized (lock) {
            result = value;
            lock.notifyAll();
        }
    }
}
```
- **局限性**：仅支持单次结果的传递，若多次调用`complete()`或`get()`，可能引发混乱。

#### **(2) 多对多场景（基于唯一标识符）**
通过引入唯一标识符（如请求ID），允许生产者和消费者动态匹配：
```java
public class GuardedObjectMultiplex {
    private static final ConcurrentHashMap<String, GuardedObject> map = new ConcurrentHashMap<>();

    // 消费者创建唯一请求并等待
    public static GuardedObject create(String id) {
        GuardedObject go = new GuardedObject();
        map.put(id, go);
        return go;
    }

    // 生产者根据ID填充结果
    public static void fireEvent(String id, Object result) {
        GuardedObject go = map.remove(id);
        if (go != null) {
            go.complete(result);
        }
    }

    // 内部类（与简单实现类似）
    private static class GuardedObject {
        private Object result;
        private final Object lock = new Object();

        public Object get() throws InterruptedException {
            synchronized (lock) {
                while (result == null) {
                    lock.wait();
                }
                return result;
            }
        }

        public void complete(Object value) {
            synchronized (lock) {
                result = value;
                lock.notifyAll();
            }
        }
    }
}
```
- **使用示例**：
  ```java
  // 消费者线程
  String requestId = UUID.randomUUID().toString();
  GuardedObjectMultiplex.create(requestId);
  Object result = GuardedObjectMultiplex.get(requestId);
  
  // 生产者线程
  GuardedObjectMultiplex.fireEvent(requestId, "Result Data");
  ```
- **优势**：支持任意数量的生产者和消费者，通过唯一ID解耦。

---

### **3. 适用场景对比**
| **场景**     | **实现方式**                 | **特点**                              |
| ------------ | ---------------------------- | ------------------------------------- |
| **一对一**   | 简单`GuardedObject`          | 简单但扩展性差，仅适用于单次交互      |
| **多对多**   | 基于唯一ID的映射管理         | 灵活，支持高并发和动态匹配            |
| **广播通知** | 单结果多消费者（如配置更新） | 所有消费者等待同一结果，`notifyAll()` |

---

### **4. 关键设计原则**
1. **解耦生产与消费**  
   - 生产者与消费者通过唯一标识符或共享条件关联，而非直接绑定线程。
2. **线程安全的数据结构**  
   - 使用`ConcurrentHashMap`或同步块管理多个`GuardedObject`实例。
3. **防止资源泄漏**  
   - 设置超时机制（如`wait(long timeout)`），避免消费者无限等待未产生的结果。
4. **支持异步通知**  
   - 结合消息队列或事件总线，实现跨线程、跨服务的结果传递。

---

### **5. 总结**
- **无需一一对应**：保护性暂停模式通过条件变量和唯一标识符设计，天然支持多对多线程协作。
- **扩展性关键**：通过唯一标识符（如请求ID）解耦生产者和消费者，是构建高并发系统的常用手段。
- **适用性广泛**：适用于RPC调用、异步任务回调、分布式锁等场景，是线程间协调的通用模式。



## 问题：可以一对多么

---

是的，**保护性暂停模式可以支持一对多（一个生产者对应多个消费者，或多个生产者对应一个消费者）的场景**。通过合理设计条件管理和线程协作机制，可以实现灵活的线程交互。以下是具体实现方式和示例：

---

### **1. 一对多场景的实现**
#### **场景描述**
- **一个生产者线程**：生成一个结果，供多个消费者线程使用。
- **多个消费者线程**：等待同一结果就绪后，各自继续执行。

#### **实现方案**
通过共享的`GuardedObject`对象，所有消费者线程等待同一条件，生产者线程在结果就绪后唤醒所有等待的消费者。

```java
public class SharedGuardedObject {
    private Object result;
    private final Object lock = new Object();

    // 消费者等待结果
    public Object getResult() throws InterruptedException {
        synchronized (lock) {
            while (result == null) {
                lock.wait();
            }
            return result;
        }
    }

    // 生产者设置结果并唤醒所有消费者
    public void setResult(Object value) {
        synchronized (lock) {
            result = value;
            lock.notifyAll(); // 唤醒所有等待线程
        }
    }
}
```

#### **使用示例**
```java
public class OneProducerToManyConsumers {
    public static void main(String[] args) {
        SharedGuardedObject sharedObj = new SharedGuardedObject();

        // 生产者线程（1个）
        new Thread(() -> {
            try {
                Thread.sleep(2000); // 模拟耗时操作
                sharedObj.setResult("Result Data");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        // 消费者线程（多个）
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                try {
                    Object result = sharedObj.getResult();
                    System.out.println(Thread.currentThread().getName() + " received: " + result);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "Consumer-" + i).start();
        }
    }
}
```

#### **输出结果**
```
Consumer-0 received: Result Data
Consumer-1 received: Result Data
Consumer-2 received: Result Data
Consumer-3 received: Result Data
Consumer-4 received: Result Data
```

---

### **2. 多对一场景的实现**
#### **场景描述**
- **多个生产者线程**：每个生产者生成独立的结果。
- **一个消费者线程**：等待任意一个生产者完成任务后获取结果。

#### **实现方案**
通过`BlockingQueue`或`CompletableFuture`等工具实现更灵活的协作，但若使用保护性暂停模式，需引入唯一标识符（如请求ID）匹配生产者和消费者。

```java
import java.util.concurrent.ConcurrentHashMap;

public class MultiplexGuardedObject {
    private static final ConcurrentHashMap<String, Object> results = new ConcurrentHashMap<>();
    private static final Object lock = new Object();

    // 消费者注册请求并等待结果
    public static Object waitForResult(String requestId) throws InterruptedException {
        synchronized (lock) {
            while (!results.containsKey(requestId)) {
                lock.wait();
            }
            return results.remove(requestId);
        }
    }

    // 生产者填充结果并通知
    public static void putResult(String requestId, Object result) {
        synchronized (lock) {
            results.put(requestId, result);
            lock.notifyAll(); // 唤醒所有等待线程
        }
    }
}
```

#### **使用示例**
```java
public class ManyProducersToOneConsumer {
    public static void main(String[] args) {
        // 消费者线程（1个）
        new Thread(() -> {
            try {
                String requestId = "REQ-123";
                Object result = MultiplexGuardedObject.waitForResult(requestId);
                System.out.println("Consumer received: " + result);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        // 生产者线程（多个）
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                String requestId = "REQ-123";
                MultiplexGuardedObject.putResult(requestId, "Result from " + Thread.currentThread().getName());
            }, "Producer-" + i).start();
        }
    }
}
```

#### **潜在问题**
- 多个生产者填充同一请求ID的结果时，消费者可能接收到任意一个结果，需根据业务需求处理竞争。

---

### **3. 关键设计原则**
| **场景**   | **实现要点**                              | **注意事项**                     |
| ---------- | ----------------------------------------- | -------------------------------- |
| **一对多** | 共享同一个`GuardedObject`，唤醒所有消费者 | 使用`notifyAll()`而非`notify()`  |
| **多对一** | 基于唯一标识符匹配结果                    | 避免请求ID冲突，管理结果生命周期 |
| **多对多** | 结合线程池和唯一标识符                    | 防止资源泄漏（如未处理的结果）   |

---

### **4. 总结**
- **支持一对多/多对一**：保护性暂停模式通过共享条件变量或唯一标识符机制，灵活支持不同线程数量的协作。
- **核心优势**：解耦生产者和消费者，减少线程间直接依赖。
- **适用场景**：异步任务回调、分布式锁、批量任务处理等。





# 问题：子类调用父类的被synchronized修饰的方法会发生什么

在Java中，当子类调用父类被`synchronized`修饰的方法时，**锁对象是子类实例本身**（即调用方法的对象），而非父类实例或父类的Class对象。以下是具体分析及示例：

---

### **1. 核心规则**
- **实例方法的`synchronized`锁**：  
  `synchronized`修饰的实例方法，锁的是当前对象实例（即`this`）。  
  无论方法定义在父类还是子类中，实际调用时，锁的总是**调用该方法的对象**（即子类实例）。

- **继承与锁的关系**：  
  子类继承父类的`synchronized`方法时，锁行为由实际调用的对象决定，与类层次无关。

---

### **2. 示例验证**
#### **(1) 子类直接调用父类的`synchronized`方法**
```java
class Parent {
    public synchronized void syncMethod() {
        System.out.println("Parent syncMethod: " + Thread.currentThread().getName());
        try {
            Thread.sleep(2000); // 模拟耗时操作
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Child extends Parent {
    // 未覆盖父类方法，直接继承syncMethod
}

public class Test {
    public static void main(String[] args) {
        Child child = new Child();
        
        // 线程1调用父类syncMethod
        new Thread(child::syncMethod, "Thread-1").start();
        
        // 线程2调用父类syncMethod
        new Thread(child::syncMethod, "Thread-2").start();
    }
}
```

**输出结果**：  
```
Parent syncMethod: Thread-1
（等待2秒）
Parent syncMethod: Thread-2
```

**说明**：  
- 线程1和线程2竞争的是同一个锁（`child`对象），因此同步执行。  
- 锁对象是子类实例`child`，而非父类或父类方法。

---

#### **(2) 子类覆盖父类的`synchronized`方法**
```java
class Parent {
    public synchronized void syncMethod() {
        System.out.println("Parent syncMethod: " + Thread.currentThread().getName());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Child extends Parent {
    @Override
    public synchronized void syncMethod() {
        System.out.println("Child syncMethod: " + Thread.currentThread().getName());
        super.syncMethod(); // 调用父类方法
    }
}

public class Test {
    public static void main(String[] args) {
        Child child = new Child();
        
        new Thread(child::syncMethod, "Thread-1").start();
        new Thread(child::syncMethod, "Thread-2").start();
    }
}
```

**输出结果**：  
```
Child syncMethod: Thread-1
Parent syncMethod: Thread-1
（等待2秒）
Child syncMethod: Thread-2
Parent syncMethod: Thread-2
```

**说明**：  
- 子类覆盖方法后，`synchronized`锁的仍是子类实例`child`。  
- 线程1和线程2依然同步执行，因为共享同一个锁（`child`对象）。

---

#### **(3) 不同子类实例调用父类方法**
```java
public class Test {
    public static void main(String[] args) {
        Child child1 = new Child();
        Child child2 = new Child();
        
        new Thread(child1::syncMethod, "Thread-1").start();
        new Thread(child2::syncMethod, "Thread-2").start();
    }
}
```

**输出结果**：  
```
Parent syncMethod: Thread-1
Parent syncMethod: Thread-2
（无等待，并行执行）
```

**说明**：  
- 不同子类实例（`child1`和`child2`）调用父类方法时，锁对象不同（各自实例），因此并行执行。

---

### **3. 关键结论**
| **场景**                             | **锁对象**         | **同步效果**             |
| ------------------------------------ | ------------------ | ------------------------ |
| 子类调用父类`synchronized`方法       | 子类实例（`this`） | 同一子类实例的调用会同步 |
| 子类覆盖父类方法并保持`synchronized` | 子类实例（`this`） | 同一子类实例的调用会同步 |
| 不同子类实例调用父类方法             | 各自实例           | 不同实例的调用不会同步   |

---

### **4. 注意事项**
- **静态方法的锁**：  
  若父类方法为`static synchronized`，锁的是父类的Class对象（`Parent.class`），与子类无关。
  
- **锁的粒度**：  
  若子类需要更细粒度的锁控制，可覆盖父类方法并使用不同的锁策略（如显式锁`ReentrantLock`）。

---

### **总结**
子类调用父类的`synchronized`方法时，锁的始终是子类实例本身（`this`）。  
- **同步生效条件**：同一实例的多次调用会同步。  
- **同步失效条件**：不同实例的调用互不影响。  
设计时需明确锁的粒度，避免因继承导致的意外同步行为。

# 问题：wait方法需要配合对象的monitor锁使用，如果对象是轻量级锁就自动升级为重量级锁么？

在Java中，当调用`wait()`方法时，如果当前对象处于**轻量级锁（Lightweight Lock）**状态，JVM会强制将该锁升级为**重量级锁（Heavyweight Lock）**，以确保`wait()`和`notify()`机制的正确运行。以下是详细分析：

---

### **1. `wait()`方法的依赖条件**
`wait()`方法必须配合对象的**监视器锁（Monitor Lock）**使用，其核心逻辑包括：
- **释放锁**：调用`wait()`的线程会释放当前持有的锁。
- **等待与唤醒**：线程进入等待队列，直到其他线程调用`notify()`/`notifyAll()`或发生超时/中断。

---

### **2. 轻量级锁的局限性**
轻量级锁的设计目标是**优化无竞争或低竞争场景**，通过CAS操作和线程栈的锁记录（Lock Record）实现锁的获取与释放。然而：
- **不支持等待队列管理**：轻量级锁无法管理因`wait()`而挂起的线程。
- **无法处理复杂同步**：`wait()`需要依赖监视器的等待/唤醒机制，而轻量级锁缺乏这些功能。

---

### **3. 锁升级的触发机制**
当调用`wait()`时，JVM会执行以下步骤：
1. **检查锁状态**：若当前锁是轻量级锁（锁标志位`00`），触发锁膨胀（Lock Inflation）。
2. **创建重量级锁（Monitor）**：
   - 分配一个`ObjectMonitor`对象（HotSpot实现），包含等待队列（Wait Set）、入口队列（Entry Set）等结构。
   - 更新对象头中的锁标志位为`10`，指向`ObjectMonitor`。
3. **挂起线程**：将当前线程加入`ObjectMonitor`的等待队列，并通过操作系统原语（如`pthread_cond_wait`）挂起线程。

---

### **4. 验证示例**
以下代码演示调用`wait()`时锁的升级过程：
```java
public class WaitLockUpgradeExample {
    private static final Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        // 线程1：获取轻量级锁后调用wait()
        new Thread(() -> {
            synchronized (lock) {  // 初始为偏向锁或无锁，首次竞争变为轻量级锁
                try {
                    lock.wait();   // 触发锁膨胀为重量级锁
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        Thread.sleep(1000); // 确保线程1已进入等待

        // 线程2：尝试获取锁，验证锁状态
        new Thread(() -> {
            synchronized (lock) {  // 此时锁已升级为重量级锁
                System.out.println("Thread2 acquired the lock");
                lock.notify();
            }
        }).start();
    }
}
```

**输出**：
```
Thread2 acquired the lock
```

**锁状态变化**：
1. 初始时，`lock`对象可能处于无锁或偏向锁状态。
2. 线程1首次获取锁时，升级为轻量级锁。
3. 线程1调用`wait()`后，锁膨胀为重量级锁。
4. 线程2通过重量级锁的机制获取锁并唤醒线程1。

---

### **5. 锁升级的意义**
- **功能完整性**：重量级锁的`ObjectMonitor`提供了等待队列管理、线程阻塞/唤醒等机制，是`wait()`/`notify()`的底层基础。
- **性能权衡**：虽然重量级锁的竞争开销更大，但`wait()`场景通常意味着线程需要挂起，此时轻量级锁的优化不再适用。

---

### **6. 其他锁状态的兼容性**
| **锁状态** | **是否支持`wait()`** | **说明**                                         |
| ---------- | -------------------- | ------------------------------------------------ |
| 无锁       | ❌                    | 调用`wait()`会抛出`IllegalMonitorStateException` |
| 偏向锁     | ❌                    | 需先撤销偏向锁，升级为重量级锁                   |
| 轻量级锁   | ❌                    | 触发锁膨胀为重量级锁                             |
| 重量级锁   | ✅                    | 直接使用`ObjectMonitor`的等待队列机制            |

---

### **总结**
- **必然升级**：调用`wait()`时，若对象处于轻量级锁状态，JVM会强制升级为重量级锁。
- **原因**：轻量级锁无法实现`wait()`所需的等待队列和线程挂起/唤醒机制。
- **设计目标**：牺牲部分性能（锁膨胀开销），确保同步机制的正确性和功能性。

# Park&UnPark

在Java中，`LockSupport`类提供的**`park()`**和**`unpark()`**方法是用于线程阻塞（挂起）和唤醒的低级操作。它们比传统的`wait()`/`notify()`更灵活，且不依赖监视器锁（`synchronized`块），常用于实现高性能并发工具（如`ReentrantLock`）。以下是其核心机制、使用场景及示例：

---

### **1. 核心机制**
#### **(1) 许可证（Permit）模型**
- **`unpark(Thread t)`**：为线程`t`发放一个许可证（最多持有1个）。
- **`park()`**：尝试消耗许可证：
  - 若有许可证，立即返回（不阻塞）。
  - 若无许可证，挂起当前线程，直到满足以下条件之一：
    - 其他线程调用`unpark(t)`发放许可证。
    - 线程被中断（`Thread.interrupt()`）。
    - 虚假唤醒（Spurious Wakeup）。

#### **(2) 关键特性**
- **无锁依赖**：无需进入`synchronized`块，直接操作线程。
- **精准唤醒**：可指定唤醒某个线程（`unpark(t)`）。
- **时序灵活性**：`unpark()`可在`park()`之前调用，确保后续`park()`不阻塞。

---

### **2. 基本用法**
#### **(1) 阻塞与唤醒线程**
```java
Thread worker = new Thread(() -> {
    System.out.println("Worker线程启动，即将挂起...");
    LockSupport.park(); // 挂起线程，等待许可证
    System.out.println("Worker线程被唤醒！");
});

worker.start();
Thread.sleep(2000); // 主线程等待2秒
LockSupport.unpark(worker); // 发放许可证，唤醒Worker线程
```

**输出**：

```
Worker线程启动，即将挂起...
（等待2秒）
Worker线程被唤醒！
```

#### **(2) 先`unpark()`后`park()`**
```java
Thread worker = new Thread(() -> {
    System.out.println("Worker线程启动");
    Thread.sleep(1000); // 模拟耗时操作
    LockSupport.park(); // 此时许可证已存在，直接继续执行
    System.out.println("Worker线程继续执行");
});

worker.start();
LockSupport.unpark(worker); // 提前发放许可证
```

**输出**：
```
Worker线程启动
Worker线程继续执行
```

---

### **3. 对比`wait()/notify()`**
| **特性**       | **`park()/unpark()`**            | **`wait()/notify()`**                                 |
| -------------- | -------------------------------- | ----------------------------------------------------- |
| **锁依赖**     | 无需锁                           | 需在`synchronized`块中使用                            |
| **精准唤醒**   | 支持（指定线程）                 | 仅能随机唤醒（`notify()`）或全部唤醒（`notifyAll()`） |
| **时序灵活性** | `unpark()`可先于`park()`调用     | 必须先`wait()`后`notify()`                            |
| **中断响应**   | `park()`会立即返回并设置中断标志 | `wait()`需显式处理`InterruptedException`              |

---

### **4. 使用场景**
#### **(1) 自定义锁实现**
```java
class SimpleLock {
    private final AtomicReference<Thread> owner = new AtomicReference<>();

    public void lock() {
        Thread current = Thread.currentThread();
        while (!owner.compareAndSet(null, current)) {
            LockSupport.park(); // 竞争失败，挂起
        }
    }

    public void unlock() {
        Thread current = Thread.currentThread();
        if (owner.compareAndSet(current, null)) {
            LockSupport.unpark(owner.get()); // 唤醒下一个线程
        }
    }
}
```

#### **(2) 线程池任务调度**
```java
ExecutorService pool = Executors.newFixedThreadPool(2);
Runnable task = () -> {
    LockSupport.parkNanos(1_000_000_000); // 挂起1秒
    System.out.println("任务完成");
};
pool.submit(task);
```

#### **(3) 中断处理**
```java
Thread worker = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        LockSupport.park(); // 阻塞线程
        System.out.println("唤醒后检查中断状态");
    }
});
worker.start();

// 中断线程
worker.interrupt(); // 导致park()返回，并设置中断标志
```

---

### **5. 注意事项**
1. **许可证不累积**：多次调用`unpark()`仅保证一个许可证。
2. **虚假唤醒**：需在循环中检查条件（类似`wait()`）。
3. **资源管理**：确保及时唤醒线程，避免永久阻塞。
4. **平台依赖性**：底层实现依赖操作系统，性能可能因环境而异。

---

### **总结**
`LockSupport.park()`和`unpark()`为线程阻塞与唤醒提供了更底层的控制，适用于实现高性能并发工具。其核心优势在于：
- **灵活性**：无需锁、精准唤醒、时序无关。
- **高效性**：直接操作线程状态，减少上下文切换开销。

合理使用这两个方法，可以优化多线程协作逻辑，但需注意线程安全和资源管理！



# 线程状态转换

Java线程的生命周期由六种状态构成，这些状态在`java.lang.Thread.State`枚举中定义。以下是各状态的详细说明及其转换关系：

---

### **1. NEW（新建）**
- **描述**：线程被创建但尚未启动（未调用`start()`方法）。
- **转换条件**：
  - **→ RUNNABLE**：调用`start()`方法启动线程。

---

### **2. RUNNABLE（可运行）**
- **描述**：线程正在JVM中执行或等待操作系统资源（如CPU时间片）。包含两种子状态：
  - **Ready**：线程已就绪，等待被调度。
  - **Running**：线程正在执行。
- **转换条件**：
  - **→ BLOCKED**：尝试进入同步代码块但未获取锁。
  - **→ WAITING**：调用`Object.wait()`、`Thread.join()`或`LockSupport.park()`。
  - **→ TIMED_WAITING**：调用带超时的方法（如`Thread.sleep()`、`Object.wait(timeout)`）。
  - **→ TERMINATED**：`run()`方法执行结束。

---

### **3. BLOCKED（阻塞）**
- **描述**：线程因等待获取**内置锁（synchronized锁）**而阻塞。
- **转换条件**：
  - **→ RUNNABLE**：成功获取锁并进入同步代码块。

---

### **4. WAITING（无限期等待）**
- **描述**：线程主动进入等待状态，直到被其他线程显式唤醒。
- **触发方法**：
  - `Object.wait()`（需在同步块中调用）。
  - `Thread.join()`（等待其他线程终止）。
  - `LockSupport.park()`。
- **转换条件**：
  - **→ RUNNABLE**：被唤醒（`Object.notify()`/`notifyAll()`、`LockSupport.unpark()`或目标线程终止）。

---

### **5. TIMED_WAITING（限期等待）**
- **描述**：线程主动进入有时间限制的等待状态。
- **触发方法**：
  - `Thread.sleep(timeout)`。
  - `Object.wait(timeout)`。
  - `Thread.join(timeout)`。
  - `LockSupport.parkNanos(timeout)`。
- **转换条件**：
  - **→ RUNNABLE**：超时结束或被唤醒。
  - **→ TERMINATED**：等待过程中线程被终止（极少见）。

---

### **6. TERMINATED（终止）**
- **描述**：线程已执行完成（`run()`方法结束）或因异常退出。
- **转换条件**：无（不可逆状态）。

---

### **状态转换示意图**
```plaintext
NEW
  ↓ (start())
RUNNABLE ↔ (尝试获取synchronized锁失败) → BLOCKED
  ↓ (调用等待方法)
WAITING
  ↓ (被唤醒或中断)
RUNNABLE
  ↓ (调用带超时的等待方法)
TIMED_WAITING
  ↓ (超时或唤醒)
RUNNABLE
  ↓ (run()结束)
TERMINATED
```

---

### **关键转换场景**
| **触发操作**                | **状态转换**                     |
| --------------------------- | -------------------------------- |
| `start()`                   | NEW → RUNNABLE                   |
| 未获取`synchronized`锁      | RUNNABLE → BLOCKED               |
| `wait()`/`join()`/`park()`  | RUNNABLE → WAITING               |
| `sleep()`/`wait(timeout)`等 | RUNNABLE → TIMED_WAITING         |
| `notify()`/`unpark()`/超时  | WAITING/TIMED_WAITING → RUNNABLE |
| `run()`结束或异常终止       | RUNNABLE → TERMINATED            |

---

### **注意事项**
1. **中断机制**：  
   - 在`WAITING`/`TIMED_WAITING`状态下调用`interrupt()`会触发`InterruptedException`，线程回到`RUNNABLE`。
2. **Lock与Condition**：  
   - 使用`Lock`和`Condition`的等待方法（如`await()`）会将线程置于`WAITING`或`TIMED_WAITING`状态，而非`BLOCKED`。
3. **不可逆状态**：  
   - 线程一旦进入`TERMINATED`状态，无法再通过`start()`重启。

---

### **总结**
理解线程状态的转换机制是编写高效、健壮多线程程序的基础。通过合理使用同步、等待和唤醒机制，可以避免死锁、活锁及资源竞争等问题。



# 活锁现象

活锁（Livelock）是多线程编程中的一种问题，表现为线程虽然没有被阻塞（如等待锁或I/O），但无法继续执行，因为它们不断改变状态以响应其他线程的动作，导致系统无法向前推进。以下是活锁的详细解释、示例及解决方法：

---

#### **1. 活锁的定义与特点**
- **定义**：线程持续改变状态以避免冲突，但最终无法完成任务的并发问题。
- **特点**：
  - 线程处于运行状态（RUNNABLE），但无实际进展。
  - 通常由不恰当的重试策略或协调逻辑引发。
  - 与死锁不同，线程未被阻塞，而是“忙等待”。

---

#### **2. 活锁的典型场景**
##### **示例1：礼貌让行**
两个线程（如两个人）互相礼让，导致无限循环：
```java
public class LivelockExample {
    static class Person {
        private String name;
        private boolean isPassing;

        public Person(String name) {
            this.name = name;
            this.isPassing = false;
        }

        public void passDoor(Person other) {
            while (!isPassing) {
                if (!other.isPassing) {
                    System.out.println(name + ": 您先请！");
                    other.isPassing = true;
                } else {
                    System.out.println(name + ": 谢谢，您先请！");
                    isPassing = true;
                }
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(name + " 通过了门。");
        }
    }

    public static void main(String[] args) {
        final Person alice = new Person("Alice");
        final Person bob = new Person("Bob");

        new Thread(() -> alice.passDoor(bob)).start();
        new Thread(() -> bob.passDoor(alice)).start();
    }
}
```
**输出**：

```
Alice: 您先请！
Bob: 您先请！
Alice: 谢谢，您先请！
Bob: 谢谢，您先请！
（无限循环）
```

##### **示例2：冲突检测与重试**
两个线程尝试修改共享资源，但因冲突检测机制不当导致无限回退：
```java
public class LivelockExample2 {
    private static volatile boolean isAdjusting = false;

    public static void main(String[] args) {
        new Thread(() -> {
            while (true) {
                if (!isAdjusting) {
                    isAdjusting = true;
                    System.out.println("Worker1 开始调整...");
                    try { Thread.sleep(1000); } catch (Exception e) {}
                    System.out.println("Worker1 调整完成。");
                    isAdjusting = false;
                } else {
                    System.out.println("Worker1 检测到冲突，等待...");
                    try { Thread.sleep(500); } catch (Exception e) {}
                }
            }
        }).start();

        new Thread(() -> {
            while (true) {
                if (!isAdjusting) {
                    isAdjusting = true;
                    System.out.println("Worker2 开始调整...");
                    try { Thread.sleep(1000); } catch (Exception e) {}
                    System.out.println("Worker2 调整完成。");
                    isAdjusting = false;
                } else {
                    System.out.println("Worker2 检测到冲突，等待...");
                    try { Thread.sleep(500); } catch (Exception e) {}
                }
            }
        }).start();
    }
}
```
**输出**：
```
Worker1 检测到冲突，等待...
Worker2 检测到冲突，等待...
（无限循环）
```

---

#### **3. 活锁的解决方法**
##### **方法1：引入随机性**
在重试时加入随机延迟，减少线程同步冲突的概率：
```java
public class LivelockSolutionExample {
    static class Person {
        // ... 省略其他代码

        public void passDoor(Person other) {
            while (!isPassing) {
                if (!other.isPassing) {
                    System.out.println(name + ": 您先请！");
                    other.isPassing = true;
                } else {
                    System.out.println(name + ": 谢谢，您先请！");
                    isPassing = true;
                }
                try {
                    // 添加随机延迟（0-1000毫秒）
                    Thread.sleep((long) (Math.random() * 1000));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(name + " 通过了门。");
        }
    }

    public static void main(String[] args) {
        // ... 启动线程
    }
}
```

##### **方法2：设置最大重试次数**
限制线程的重试次数，避免无限循环：
```java
public class LivelockSolutionExample2 {
    static class Person {
        private int retryCount = 0;
        private static final int MAX_RETRY = 3;

        public void passDoor(Person other) {
            while (!isPassing && retryCount < MAX_RETRY) {
                // ... 原有逻辑
                retryCount++;
            }
            if (retryCount >= MAX_RETRY) {
                System.out.println(name + " 放弃让行，直接通过！");
                isPassing = true;
            }
        }
    }
}
```

##### **方法3：使用超时机制**
通过超时强制打破活锁状态：
```java
public class LivelockSolutionExample3 {
    static class Person {
        public void passDoor(Person other) {
            long startTime = System.currentTimeMillis();
            while (!isPassing) {
                if (System.currentTimeMillis() - startTime > 5000) {
                    System.out.println(name + " 超时，强制通过！");
                    isPassing = true;
                    break;
                }
                // ... 原有逻辑
            }
        }
    }
}
```

---

#### **4. 活锁与死锁的区别**
| **特性**     | **活锁（Livelock）**            | **死锁（Deadlock）**                |
| ------------ | ------------------------------- | ----------------------------------- |
| **线程状态** | 线程处于运行状态（RUNNABLE）    | 线程处于阻塞状态（BLOCKED/WAITING） |
| **表现形式** | 线程持续重试但无进展            | 线程无限等待资源                    |
| **触发原因** | 不恰当的重试策略或协调逻辑      | 资源竞争与持有-等待循环             |
| **解决难度** | 需调整重试逻辑或引入随机性/超时 | 需打破资源循环等待                  |

---

#### **5. 总结**
- **活锁的本质**：线程因不当协调逻辑陷入无限重试循环。
- **解决方案**：引入随机性、限制重试次数、设置超时。
- **应用场景**：高并发系统中的任务调度、资源分配等场景。

通过合理设计线程交互逻辑，结合上述解决方法，可以有效避免活锁问题，提升系统的稳定性和性能。

# 饥饿问题

### 多线程的饥饿问题

**饥饿（Starvation）** 是多线程编程中的一种现象，指某个线程因无法获取所需资源（如CPU时间片、锁、I/O等）而长期无法执行任务。与死锁和活锁不同，饥饿的线程可能一直处于可运行状态（`RUNNABLE`），但被其他线程“排挤”，导致无法推进。

---

#### **1. 饥饿的常见原因**
1. **线程优先级设置不合理**  
   高优先级线程持续占用CPU资源，低优先级线程长期得不到执行（尤其在依赖操作系统调度时）。
   
2. **资源分配策略不公平**  
   - **非公平锁**：某些线程频繁抢占锁，导致其他线程无法获得锁。  
   - **不均衡的任务分配**：任务队列中某些线程的任务被其他线程快速消费。

3. **线程持有资源时间过长**  
   某个线程长期占用共享资源（如数据库连接、文件句柄），导致其他线程等待超时或失败。

4. **算法设计缺陷**  
   例如，线程调度算法始终优先处理新请求，导致旧请求被“饿死”。

---

#### **2. 饥饿的示例**
##### **示例1：非公平锁导致的饥饿**
```java
public class StarvationExample {
    private static final Lock lock = new ReentrantLock(); // 默认非公平锁

    public static void main(String[] args) {
        // 线程1：频繁获取锁
        new Thread(() -> {
            while (true) {
                lock.lock();
                try {
                    System.out.println("Thread1 执行任务");
                    Thread.sleep(100); // 模拟短时占用
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        }).start();

        // 线程2：因无法获取锁而饥饿
        new Thread(() -> {
            while (true) {
                lock.lock();
                try {
                    System.out.println("Thread2 执行任务");
                } finally {
                    lock.unlock();
                }
            }
        }).start();
    }
}
```
**输出**：  
`Thread1` 持续输出，`Thread2` 几乎无法执行。

##### **示例2：线程优先级导致的饥饿**
```java
public class PriorityStarvationExample {
    public static void main(String[] args) {
        // 高优先级线程
        Thread highPriorityThread = new Thread(() -> {
            while (true) {
                System.out.println("HighPriorityThread 运行");
            }
        });
        highPriorityThread.setPriority(Thread.MAX_PRIORITY);

        // 低优先级线程
        Thread lowPriorityThread = new Thread(() -> {
            while (true) {
                System.out.println("LowPriorityThread 运行");
            }
        });
        lowPriorityThread.setPriority(Thread.MIN_PRIORITY);

        highPriorityThread.start();
        lowPriorityThread.start();
    }
}
```
**结果**：  
在某些操作系统中，`LowPriorityThread` 可能几乎得不到执行。

---

#### **3. 解决饥饿的方法**
##### **方法1：使用公平锁（Fair Lock）**
- **原理**：按线程等待顺序分配锁，避免插队。  
- **实现（Java）**：  
  ```java
  Lock fairLock = new ReentrantLock(true); // 公平锁
  ```

##### **方法2：合理设置线程优先级**
- **注意事项**：  
  Java线程优先级（1-10）依赖操作系统实现，建议避免过度依赖优先级，而是通过设计公平的任务调度。

##### **方法3：引入超时机制**
- **原理**：线程在等待资源时设置超时，避免无限期等待。  
- **实现（Java）**：  
  ```java
  if (lock.tryLock(1, TimeUnit.SECONDS)) { // 尝试获取锁，最多等待1秒
      try {
          // 执行任务
      } finally {
          lock.unlock();
      }
  } else {
      // 处理超时逻辑
  }
  ```

##### **方法4：均衡任务分配**
- **示例**：使用轮询调度（Round-Robin）或加权队列，确保所有线程的任务被公平处理。  
  ```java
  ExecutorService pool = Executors.newFixedThreadPool(4);
  ```

##### **方法5：限制资源占用时间**
- **原理**：强制线程释放资源，避免长期独占。  
- **实现**：在代码逻辑中设置最大执行时间，或使用资源池（如数据库连接池）。

---

#### **4. 饥饿与死锁、活锁的区别**
| **现象** | **描述**                                                     | **线程状态**            |
| -------- | ------------------------------------------------------------ | ----------------------- |
| **饥饿** | 线程因无法获取资源而长期无法执行。                           | `RUNNABLE` 或 `BLOCKED` |
| **死锁** | 多个线程互相等待对方释放资源，导致所有线程阻塞。             | `BLOCKED` 或 `WAITING`  |
| **活锁** | 线程不断改变状态以响应其他线程，但无法推进任务（如“礼让”导致循环）。 | `RUNNABLE`              |

---

#### **5. 实际应用建议**
1. **优先使用公平锁**：在资源竞争激烈时，公平锁可减少饥饿风险（但可能降低吞吐量）。  
2. **监控线程状态**：通过工具（如VisualVM、Arthas）检测长时间等待的线程。  
3. **设计资源超时**：对锁、I/O操作等设置超时，避免无限期等待。  
4. **避免过度优化**：不要为了性能牺牲公平性，尤其在需要稳定性的系统中。

---

### **总结**
饥饿问题源于资源分配的不公平性，通过合理设计锁策略、任务调度和超时机制，可以有效缓解。关键在于平衡性能与公平性，确保系统长期稳定运行。

# **ReentrantLock** 

---

**ReentrantLock** 是 Java 并发包（`java.util.concurrent.locks`）中提供的一个可重入互斥锁，相比传统的 `synchronized` 关键字，它提供了更灵活的锁控制、更高的性能优化以及更丰富的功能（如公平锁、可中断锁、超时锁等）。以下是其核心特性和使用详解：

---

### **1. ReentrantLock 的核心特性**
| **特性**       | **说明**                                                     |
| -------------- | ------------------------------------------------------------ |
| **可重入性**   | 同一线程可重复获取锁（避免死锁）。                           |
| **公平性选择** | 支持公平锁（按等待顺序获取锁）和非公平锁（允许插队，默认）。 |
| **锁中断**     | 线程等待锁时可被中断（`lockInterruptibly()`）。              |
| **超时机制**   | 尝试获取锁时设置超时时间（`tryLock(long timeout, TimeUnit unit)`）。 |
| **条件变量**   | 通过 `Condition` 实现多条件等待和唤醒（类似 `wait()`/`notify()` 的增强版）。 |

---

### **2. ReentrantLock 的基本使用**
#### **基本模板**
```java
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockExample {
    private final ReentrantLock lock = new ReentrantLock();

    public void doWork() {
        lock.lock();  // 获取锁
        try {
            // 受保护的代码块
            System.out.println(Thread.currentThread().getName() + " 正在执行任务");
        } finally {
            lock.unlock();  // 必须在 finally 中释放锁
        }
    }

    public static void main(String[] args) {
        ReentrantLockExample example = new ReentrantLockExample();
        new Thread(example::doWork).start();
        new Thread(example::doWork).start();
    }
}
```

---

### **3. 核心方法详解**
#### **(1) `lock()` 与 `unlock()`**
- **`lock()`**：获取锁，若锁被其他线程持有，则当前线程阻塞等待。
- **`unlock()`**：释放锁，必须显式调用（通常在 `finally` 块中确保释放）。

#### **(2) `tryLock()`**
尝试非阻塞获取锁，若成功返回 `true`，否则返回 `false`：
```java
if (lock.tryLock()) {
    try {
        // 获取锁成功，执行任务
    } finally {
        lock.unlock();
    }
} else {
    // 获取锁失败，执行其他逻辑
}
```

#### **(3) `tryLock(long timeout, TimeUnit unit)`**
在指定时间内尝试获取锁，支持中断：
```java
try {
    if (lock.tryLock(1, TimeUnit.SECONDS)) {
        try {
            // 获取锁成功
        } finally {
            lock.unlock();
        }
    } else {
        // 超时未获取锁
    }
} catch (InterruptedException e) {
    // 处理中断异常
}
```

#### **(4) `lockInterruptibly()`**
可中断地获取锁，若线程在等待锁时被中断，会抛出 `InterruptedException`：
```java
try {
    lock.lockInterruptibly();
    try {
        // 执行任务
    } finally {
        lock.unlock();
    }
} catch (InterruptedException e) {
    // 处理中断
}
```

---

### **4. 公平锁与非公平锁**
#### **(1) 创建方式**
```java
ReentrantLock fairLock = new ReentrantLock(true);   // 公平锁
ReentrantLock nonFairLock = new ReentrantLock();    // 非公平锁（默认）
```

#### **(2) 对比**
| **特性**       | **公平锁**                   | **非公平锁**                 |
| -------------- | ---------------------------- | ---------------------------- |
| **锁获取顺序** | 严格按等待队列顺序分配锁     | 允许新线程插队，可能导致饥饿 |
| **吞吐量**     | 较低（维护队列顺序增加开销） | 较高（减少线程切换）         |
| **适用场景**   | 高竞争场景，需避免线程饥饿   | 低竞争场景，追求高吞吐量     |

---

### **5. 条件变量（Condition）**
通过 `Condition` 实现更精细的线程等待与唤醒：
```java
import java.util.concurrent.locks.Condition;

public class ConditionExample {
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();
    private boolean isReady = false;

    public void waitForCondition() throws InterruptedException {
        lock.lock();
        try {
            while (!isReady) {
                condition.await(); // 释放锁并等待
            }
            // 条件满足后执行
        } finally {
            lock.unlock();
        }
    }

    public void signalCondition() {
        lock.lock();
        try {
            isReady = true;
            condition.signalAll(); // 唤醒所有等待线程
        } finally {
            lock.unlock();
        }
    }
}
```

---

### **6. ReentrantLock 与 synchronized 的对比**
| **特性**       | **ReentrantLock**            | **synchronized**                          |
| -------------- | ---------------------------- | ----------------------------------------- |
| **锁获取方式** | 显式调用 `lock()`/`unlock()` | 隐式通过代码块或方法修饰符                |
| **灵活性**     | 支持公平锁、可中断、超时等   | 仅支持非公平锁                            |
| **性能**       | 高并发场景下更优（优化竞争） | 低竞争场景下更简单高效                    |
| **条件变量**   | 支持多个 `Condition`         | 仅通过 `wait()`/`notify()` 实现单条件等待 |
| **锁释放**     | 必须显式释放（`unlock()`）   | 自动释放（代码块结束或方法返回）          |

---

### **7. 使用注意事项**
1. **始终在 `finally` 中释放锁**：避免锁泄漏导致死锁。
2. **避免嵌套锁**：容易引发死锁，尽量使用单锁或锁顺序协议。
3. **合理选择公平性**：非公平锁提升吞吐，公平锁避免饥饿。
4. **优先使用 `tryLock`**：避免长时间阻塞，增强系统健壮性。

---

### **8. 适用场景**
- **需要精细锁控制**：如超时锁、可中断锁。
- **高并发优化**：通过非公平锁减少竞争开销。
- **复杂条件等待**：使用多个 `Condition` 管理不同等待队列。
- **替代 `synchronized`**：当需要更灵活的锁策略时。

---

### **总结**
**ReentrantLock** 是 Java 并发编程中强大的锁工具，通过灵活的方法和策略，可以显著优化多线程程序的性能和可靠性。正确使用时需注意锁的获取与释放、公平性选择以及条件变量的应用，从而避免死锁和资源竞争问题。