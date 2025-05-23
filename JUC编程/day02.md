从75集到 95 集



# monitor

## 问题：Java多线程中的monitor

Java中的monitor是一种用于实现多线程同步的机制，每个对象都关联一个内置的monitor，通过`synchronized`关键字、`wait()`、`notify()`等方法实现线程间的协作与互斥。以下是关键点总结：

### 1. **Monitor的核心组成**
   - **锁（Lock）**：每个对象都有一个monitor锁，线程进入`synchronized`代码块时尝试获取锁，成功则独占访问。
   - **计数器（Counter）**：记录锁的重入次数。可重入性允许线程多次获取同一锁（如递归调用或调用同一对象的其他同步方法）。
   - **Entry Set（入口集）**：等待获取锁的线程阻塞队列。锁释放时，JVM从入口集中选择一个线程授予锁。
   - **Wait Set（等待集）**：调用`wait()`的线程进入此队列，等待其他线程通过`notify()`或`notifyAll()`唤醒。

### 2. **synchronized关键字**
   - 修饰代码块或方法时，自动获取对象monitor的锁，退出时释放。
   - 底层通过JVM指令`monitorenter`和`monitorexit`实现，确保代码块的原子性。

### 3. **wait()、notify()与notifyAll()**
   - **`wait()`**：释放锁并进入等待集，线程挂起，需在`synchronized`块中调用。
   - **`notify()`**：随机唤醒一个等待集中的线程，该线程需重新竞争锁。
   - **`notifyAll()`**：唤醒所有等待线程，它们移至入口集竞争锁。
   - 使用时应配合循环检查条件，避免虚假唤醒：
     ```java
     synchronized (obj) {
         while (条件不满足) {
             obj.wait();
         }
         // 处理业务
     }
     ```

### 4. **可重入性**
   - 同一线程可重复获取已持有的锁，计数器递增，避免死锁。例如：
     ```java
     public synchronized void methodA() {
         methodB(); // 直接调用另一个同步方法，无需重新获取锁
     }
     public synchronized void methodB() { /* ... */ }
     ```

### 5. **对象锁与类锁**
   - **实例对象锁**：作用于对象实例，不同实例的锁互不干扰。
   - **类锁**：作用于类的Class对象（如静态同步方法），所有实例共享同一锁。

### 6. **Monitor与Lock的区别**
   - **Monitor**：通过`synchronized`隐式管理，简单但功能有限。
   - **Lock接口**（如`ReentrantLock`）：需显式加锁/解锁，支持超时、中断、公平锁等高级功能，基于AQS（AbstractQueuedSynchronizer）实现。

### 7. **注意事项**
   - **死锁风险**：多个线程互相等待对方释放锁，需避免嵌套锁或使用超时机制。
   - **性能影响**：过度同步可能导致线程竞争，降低并发效率。

### 示例代码
```java
public class MonitorExample {
    private final Object lock = new Object();
    private boolean condition = false;

    public void doWait() throws InterruptedException {
        synchronized (lock) {
            while (!condition) {
                lock.wait(); // 释放锁，进入等待集
            }
            // 条件满足后的操作
        }
    }

    public void doNotify() {
        synchronized (lock) {
            condition = true;
            lock.notifyAll(); // 唤醒所有等待线程
        }
    }
}
```

### 总结
Monitor是Java多线程同步的基础机制，通过内置锁和等待/通知机制协调线程。理解其原理有助于编写高效、安全的并发代码，而后续的并发工具类（如`Lock`、`Condition`）则在此基础上提供了更灵活的扩展。



## 问题：每个对象都有一个monitor锁么，即使他没有被synchronized修饰

是的，在 **Java 的底层实现中，每个对象（包括普通对象和数组）从设计上都被赋予了支持同步的能力**，即使它没有被任何 `synchronized` 代码块或方法修饰。但这一机制的实现细节需要结合 JVM 规范（理论层面）和具体虚拟机的实际行为（如 HotSpot）来理解。

---

### **1. 理论层面：JVM 规范的定义**
根据 **Java 虚拟机规范（JVM Specification）**：
- **每个对象都有一个关联的监视器（monitor）**，这是 JVM 为实现同步机制预留的逻辑结构。
- 监视器的存在是隐式的，**无论对象是否实际被用作锁**。这意味着：
  - 即使一个对象从未被 `synchronized` 修饰，它在 JVM 的视角中仍具备支持同步的“潜力”。
  - 当线程首次尝试通过 `synchronized` 获取该对象的锁时，JVM 会按需初始化与该对象关联的监视器。

---

### **2. 实践层面：HotSpot 虚拟机的实现**
在 Oracle 的 **HotSpot JVM** 中，对象的同步能力通过 **对象头（Object Header）** 实现，而不是为每个对象预先分配一个完整的监视器结构。对象头中存储以下关键信息：
- **Mark Word**：记录对象的哈希码、锁状态（无锁、偏向锁、轻量级锁、重量级锁）、GC 分代年龄等。
- **Klass Pointer**：指向类元数据的指针。

#### **锁的按需升级机制**
HotSpot 使用 **锁膨胀（Lock Inflation）** 机制优化同步性能：
1. **偏向锁（Biased Locking）**  
   - 当对象第一次被线程访问时，JVM 会尝试将线程 ID 记录在 Mark Word 中，避免后续重复加锁的开销。
   - **此时并未创建完整的监视器（Monitor）**。

2. **轻量级锁（Lightweight Locking）**  
   - 当多个线程轻度竞争时，JVM 通过 CAS 操作将 Mark Word 替换为指向线程栈中锁记录的指针。
   - **仍无需创建完整的监视器**。

3. **重量级锁（Heavyweight Locking）**  
   - 当竞争激烈时，锁会膨胀为重量级锁，此时 **JVM 会创建与该对象关联的监视器（Monitor）**。
   - 监视器包含 Entry Set、Wait Set 等结构，用于管理线程的阻塞和唤醒。

---

### **3. 关键结论**
- **每个对象在逻辑上都支持同步**（因为 JVM 规范要求），但 **物理上是否实际分配监视器（Monitor）取决于锁竞争情况**。
- **未被 `synchronized` 修饰的对象**：
  - 如果从未被用作锁，其对象头中的锁状态可能一直处于“无锁”或“偏向锁”状态，**不会创建完整的监视器**。
  - 若后续被用作锁且发生竞争，JVM 会按需创建监视器（重量级锁）。

---

### **4. 示例验证**
#### 场景：未使用 `synchronized` 的对象调用 `wait()`
```java
Object obj = new Object();
obj.wait(); // 抛出 IllegalMonitorStateException
```
- 虽然 `obj` 理论上具备监视器，但直接调用 `wait()` 会抛出异常，因为 **必须先在 `synchronized` 块中获取对象的锁**。
- 这说明监视器的使用必须遵循显式的同步规则，即使底层存在相关机制。

---

### **5. 性能与资源优化**
- JVM **不会为所有对象预先分配监视器**，因为这会浪费内存（Java 程序可能创建海量对象）。
- 通过锁膨胀机制，JVM 仅在必要时（如高竞争场景）创建监视器，平衡了同步功能与资源开销。

---

### **总结**
- **每个对象都有支持同步的潜力**，这是 JVM 规范的要求。
- **实际是否分配监视器**（重量级锁）**取决于锁竞争情况**，由 JVM 按需处理。
- 即使对象未被 `synchronized` 修饰，其底层仍然具备同步的“基础设施”，但仅在显式同步时才会激活。



## 问题：Java对象头是什么，markword是什么

在 Java 中，**对象头（Object Header）** 是每个对象在内存中的一部分，用于存储对象的元数据（如锁状态、哈希码、GC 分代年龄等）。它是 JVM 实现对象管理、垃圾回收、多线程同步等功能的基础。其中，**Mark Word（标记字段）** 是对象头中最重要的部分，记录了对象运行时的关键信息。

---

### **1. Java 对象的内存结构**
每个 Java 对象在内存中的布局分为三部分：
1. **对象头（Object Header）**  
   - **Mark Word**：存储对象的运行时元数据（如哈希码、锁状态、分代年龄等）。
   - **Klass Pointer**（类型指针）：指向类元数据的指针（即对象的类型信息在方法区中的地址）。
   - （如果是数组对象）**数组长度**：记录数组的长度。

2. **实例数据（Instance Data）**  
   - 对象的实际字段数据（包括父类继承的字段）。

3. **对齐填充（Padding）**  
   - 为了内存对齐而填充的额外字节（HotSpot JVM 要求对象大小是 8 字节的整数倍）。

---

### **2. Mark Word 的作用与结构**
Mark Word 是对象头的核心部分，长度为 **32 位或 64 位**（取决于 JVM 是 32 位还是 64 位）。它的内容会根据对象的锁状态动态变化，涵盖以下信息：

#### **Mark Word 的通用结构**
| 锁状态       | 存储内容（以 64 位 JVM 为例）                                |
| ------------ | ------------------------------------------------------------ |
| **无锁**     | 哈希码（25b） + 分代年龄（4b） + 偏向锁标志（1b） + 锁标志（2b） |
| **偏向锁**   | 线程 ID（54b） + Epoch（2b） + 分代年龄（4b） + 偏向锁标志（1b） + 锁标志（2b） |
| **轻量级锁** | 指向线程栈中锁记录的指针（62b） + 锁标志（2b）               |
| **重量级锁** | 指向监视器（Monitor）的指针（62b） + 锁标志（2b）            |
| **GC 标记**  | 空（所有位用于 GC 标记） + 锁标志（2b）                      |

#### **关键字段说明**
1. **哈希码（Identity Hash Code）**  
   - 调用 `hashCode()` 方法生成的哈希码（如果未重写，默认是对象的内存地址的派生值）。
   - **一旦哈希码被写入 Mark Word，对象将无法进入偏向锁状态**（因为偏向锁需要占用哈希码的存储空间）。

2. **分代年龄（Age）**  
   - 对象经历 GC 的次数（4 位，最大值 15，触发阈值后对象会被移到老年代）。

3. **锁标志（Lock Flags）**  
   - 2 位标志，标识当前锁状态：
     - `00`：轻量级锁
     - `01`：无锁或偏向锁（通过偏向锁标志区分）
     - `10`：重量级锁
     - `11`：GC 标记

4. **偏向锁标志（Biased Lock Flag）**  
   - 1 位，标识是否启用偏向锁：
     - `0`：无锁或未启用偏向锁
     - `1`：偏向锁

---

### **3. Mark Word 的锁状态转换**
HotSpot JVM 通过锁膨胀（Lock Inflation）机制优化同步性能，Mark Word 的结构会根据锁竞争动态变化：

#### **锁升级流程**
1. **无锁**  
   - 初始状态，未被任何线程锁定。
   - 哈希码未被计算时，Mark Word 可存储偏向锁信息。

2. **偏向锁**  
   - 当对象被一个线程多次访问时，JVM 将线程 ID 写入 Mark Word，避免重复加锁。
   - **适用于单线程重复访问的场景**（如方法局部锁）。

3. **轻量级锁**  
   - 当多个线程轻度竞争时，JVM 将 Mark Word 复制到线程栈，并通过 CAS 操作尝试获取锁。
   - **适用于短时间、低竞争的同步**。

4. **重量级锁**  
   - 当竞争激烈时，锁膨胀为重量级锁，Mark Word 指向 Monitor（监视器）对象。
   - **线程竞争时直接阻塞**，由操作系统管理线程唤醒。

---

### **4. Klass Pointer（类型指针）**
- 指向方法区中类的元数据（Klass），用于标识对象的类型。
- 在 64 位 JVM 中默认开启指针压缩（-XX:+UseCompressedOops），Klass Pointer 占用 32 位。

---

### **5. 查看对象头**
可以通过工具（如 JOL，Java Object Layout）查看对象头信息：

#### **示例代码**
```java
// 添加 Maven 依赖：org.openjdk.jol:jol-core
import org.openjdk.jol.vm.VM;

public class ObjectHeaderExample {
    public static void main(String[] args) {
        Object obj = new Object();
        System.out.println(VM.current().detailsOf(obj));
    }
}
```

#### **输出结果**
```
OFFSET  SIZE   TYPE DESCRIPTION
0     4        (object header)  01 00 00 00 (00000001 00000000 00000000 00000000) (1)
4     4        (object header)  00 00 00 00 (00000000 00000000 00000000 00000000) (0)
8     4        (object header)  e5 01 00 20 (11100101 00000001 00000000 00100000) (536871397)
...
```

---

### **6. 总结**
- **对象头**是 Java 对象的元数据容器，包含 Mark Word 和 Klass Pointer。
- **Mark Word** 是对象头的核心，动态存储锁状态、哈希码、分代年龄等信息。
- 锁状态的升级（无锁 → 偏向锁 → 轻量级锁 → 重量级锁）通过修改 Mark Word 实现，是 JVM 优化同步性能的关键机制。
- 理解对象头和 Mark Word 有助于分析多线程竞争、内存占用优化等问题。



## 问题：一旦哈希码被写入 Mark Word，对象将无法进入偏向锁状态，这句话是什么意思，一个对象怎么才能不让哈希码写入mark word，如果对象进入锁状态，就没有哈希码了？那它就不能加入hashmap了？


Java对象的哈希码与偏向锁状态之间存在互斥关系，这是由Mark Word的空间设计决定的。以下是对问题的详细解答：

---

### **1. 哈希码与偏向锁的互斥性**

#### **为什么哈希码写入后无法进入偏向锁？**
- **空间冲突**：  
  在无锁状态下，Mark Word需要存储哈希码（25位）、分代年龄（4位）、偏向锁标志（1位）和锁标志（2位）。  
  偏向锁需要存储**线程ID（54位）**、epoch（2位）、分代年龄（4位）等数据。  
  由于Mark Word的总长度固定（64位），一旦哈希码被写入，会占用原本用于存储偏向锁信息的位置，导致无法再启用偏向锁。

- **触发条件**：  
  当首次调用对象的`hashCode()`方法（默认实现）时，哈希码会被计算并写入Mark Word。此时，偏向锁所需的空间被占用，对象无法进入偏向锁状态。

---

### **2. 如何避免哈希码写入Mark Word？**

#### **方法一：不调用默认的`hashCode()`**
- 如果不调用对象的`hashCode()`方法（如未将对象放入`HashMap`或`HashSet`），Mark Word中的哈希码字段可能保持为空，对象仍可进入偏向锁状态。  
- **局限性**：实际开发中，对象常被用作集合的键，很难避免`hashCode()`的调用。

#### **方法二：重写`hashCode()`方法**
- 若重写`hashCode()`，使其不依赖默认实现（即不调用`Object.hashCode()`），可避免哈希码写入Mark Word。例如：
  ```java
  @Override
  public int hashCode() {
      // 返回固定值或基于其他字段计算的哈希码
      return 42; 
  }
  ```
- **注意**：此方法需谨慎，不合理的哈希码实现会破坏哈希集合的性能。

#### **方法三：禁用偏向锁**
- 通过JVM参数`-XX:-UseBiasedLocking`禁用偏向锁，此时所有对象直接使用轻量级锁或重量级锁，无需关心哈希码的影响。

---

### **3. 锁状态与哈希码的关系**

#### **锁状态是否会导致哈希码丢失？**
- **不会丢失**。哈希码一旦生成，即使对象进入锁状态（轻量级锁或重量级锁），JVM会将哈希码转移到安全位置（如线程栈的锁记录或Monitor对象中），确保后续调用`hashCode()`时仍能正确返回。  
- **示例**：  
  1. 对象处于无锁状态，哈希码直接存储在Mark Word中。  
  2. 进入轻量级锁时，Mark Word被复制到线程栈的锁记录中，哈希码也随之保存。  
  3. 升级到重量级锁时，哈希码存储在Monitor对象中。

#### **对象能否加入HashMap？**
- **可以正常使用**。即使对象处于锁状态，其哈希码仍有效，因此可以作为`HashMap`的键。例如：
  
  ```java
  Object key = new Object();
  synchronized (key) {  // 进入重量级锁，哈希码存储在Monitor中
      map.put(key, "value"); // 调用key.hashCode()仍能正确获取哈希码
  }
  ```

---

### **4. 总结**

| 场景                   | 哈希码存储位置              | 是否支持偏向锁             | 对HashMap的影响              |
| ---------------------- | --------------------------- | -------------------------- | ---------------------------- |
| 未调用`hashCode()`     | Mark Word未写入哈希码       | ✅ 支持                     | 无法作为键（除非重写哈希码） |
| 调用了默认`hashCode()` | Mark Word存储哈希码         | ❌ 不支持                   | 可作为键                     |
| 对象处于锁状态         | 哈希码转移至锁记录或Monitor | 由锁状态决定（可能不支持） | 仍可作为键                   |

#### **关键结论**
1. **哈希码与偏向锁互斥**：哈希码写入Mark Word会占用偏向锁所需的空间。
2. **避免哈希码写入的代价**：需牺牲默认哈希码的便利性或禁用偏向锁。
3. **锁状态不影响哈希码有效性**：即使对象被锁定，仍可安全用于哈希集合。



## 问题： lock record是什么

---

在 Java 的轻量级锁（Lightweight Locking）机制中，**Lock Record（锁记录）** 是线程在尝试获取锁时，在**线程栈帧**中创建的一个临时数据结构，用于存储锁对象的原始 Mark Word 信息，并通过 CAS（Compare and Swap）操作竞争锁。它的存在是为了优化同步性能，避免直接进入重量级锁的阻塞开销。

---

### **1. Lock Record 的作用**
- **存储原始 Mark Word**：在轻量级锁获取过程中，线程会将锁对象的 Mark Word 拷贝到 Lock Record 中。
- **支持锁竞争**：通过 CAS 操作尝试将锁对象的 Mark Word 替换为指向 Lock Record 的指针，若成功则获取锁。
- **实现锁释放**：解锁时，通过 CAS 将 Mark Word 恢复为 Lock Record 中保存的原始值。

---

### **2. Lock Record 的结构**
在 HotSpot JVM 中，Lock Record 的结构简化如下：
```plaintext
+-------------------+-------------------+
|  锁对象的原始 Mark Word (Displaced Mark Word)  |
+-------------------+-------------------+
|        指向锁对象的指针（可选，用于释放锁时验证）      |
+-------------------+-------------------+
```
- **Displaced Mark Word**：保存锁对象的原始 Mark Word（无锁状态下的内容）。
- **对象指针**（可选）：部分 JVM 实现会记录锁对象的引用，用于解锁时验证锁的持有者。

---

### **3. 轻量级锁的加锁流程**
以下步骤展示了 Lock Record 在轻量级锁中的使用：

#### **步骤 1：创建 Lock Record**
当线程尝试进入 `synchronized` 代码块时，JVM 会在**当前线程的栈帧**中分配一个 Lock Record。

#### **步骤 2：保存原始 Mark Word**
将锁对象的当前 Mark Word 复制到 Lock Record 的 **Displaced Mark Word** 字段。

#### **步骤 3：CAS 竞争锁**
通过 CAS 操作尝试将锁对象的 Mark Word 更新为指向当前线程的 Lock Record 的指针：
- **成功**：锁对象的 Mark Word 变为轻量级锁状态（锁标志位 `00`），线程获取锁。
- **失败**：说明存在竞争，轻量级锁膨胀为**重量级锁**（锁标志位 `10`）。

#### **示例（伪代码）**：
```java
Object obj = new Object();
synchronized (obj) { 
    // 轻量级锁加锁流程
    LockRecord lr = currentThread.stackFrame.allocateLockRecord();
    lr.displacedMarkWord = obj.markWord; // 保存原始 Mark Word
    if (CAS(obj.markWord, lr)) {        // 尝试替换为指向 Lock Record 的指针
        // 加锁成功
    } else {
        // 锁膨胀为重量级锁
    }
}
```

---

### **4. 轻量级锁的解锁流程**
#### **步骤 1：恢复 Mark Word**
通过 CAS 操作将锁对象的 Mark Word 恢复为 Lock Record 中保存的 **Displaced Mark Word**。
- **成功**：锁释放，对象回到无锁状态。
- **失败**：说明锁已膨胀为重量级锁，需通过 Monitor 机制唤醒其他线程。

#### **步骤 2：清除 Lock Record**
释放线程栈帧中的 Lock Record 空间。

---

### **5. Lock Record 与重量级锁的关系**
- **轻量级锁**：依赖线程栈中的 Lock Record，通过 CAS 实现无阻塞竞争。
- **重量级锁**：当轻量级锁竞争失败时，锁膨胀为重量级锁，锁对象的 Mark Word 指向 **Monitor 对象**（操作系统级别的互斥量），此时 Lock Record 不再起作用。

---

### **6. 为什么需要 Lock Record？**
1. **减少阻塞**：轻量级锁通过 CAS 操作避免线程直接进入内核态阻塞，提升低竞争场景的性能。
2. **空间复用**：Lock Record 分配在线程栈帧中，生命周期与代码块同步，无需全局管理。
3. **锁状态回溯**：保存原始 Mark Word 确保锁释放后对象状态可恢复。

---

### **7. 示例分析**
#### 场景：两个线程交替执行同步块
```java
public class LightweightLockExample {
    private static final Object lock = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (lock) {  // 线程1获取轻量级锁
                System.out.println("Thread 1 locked");
            }
        }).start();

        new Thread(() -> {
            synchronized (lock) {  // 线程2尝试获取锁，触发轻量级锁膨胀
                System.out.println("Thread 2 locked");
            }
        }).start();
    }
}
```
- **线程1**：成功通过 CAS 将锁对象的 Mark Word 替换为指向其 Lock Record 的指针。
- **线程2**：CAS 失败，触发锁膨胀，锁升级为重量级锁，Mark Word 指向 Monitor 对象。

---

### **8. 总结**
| **特性**       | **Lock Record**         | **Monitor（重量级锁）** |
| -------------- | ----------------------- | ----------------------- |
| **存储位置**   | 线程栈帧                | 堆内存（Monitor 对象）  |
| **竞争方式**   | CAS 操作                | 操作系统互斥量（mutex） |
| **适用场景**   | 低竞争、短时同步        | 高竞争、长时同步        |
| **性能开销**   | 低（用户态操作）        | 高（涉及内核态切换）    |
| **锁状态标志** | Mark Word 锁标志位 `00` | Mark Word 锁标志位 `10` |

- **Lock Record** 是轻量级锁的核心实现机制，通过线程本地存储和 CAS 操作优化低竞争场景的同步性能。
- 理解 Lock Record 的工作机制，有助于分析多线程竞争问题及优化高并发代码。



## 问题：jvm中栈和堆是什么有什么联系

在 Java 虚拟机（JVM）中，**栈（Stack）**和**堆（Heap）**是内存管理的核心组成部分，它们分工明确，但又通过对象的引用关系紧密关联。以下是它们的定义、区别和联系：

---

### **1. 栈（Stack）**
#### **定义**
- **线程私有**：每个线程在创建时都会分配一个独立的栈空间，生命周期与线程相同。
- **存储内容**：
  - **栈帧（Stack Frame）**：每个方法的调用会创建一个栈帧，方法结束时栈帧被销毁。
  - **栈帧内部**：
    - **局部变量表**：存储方法中的基本类型变量（`int`、`boolean`等）和对象引用（如 `Object obj`）。
    - **操作数栈**：用于方法执行时的计算操作（如算术运算）。
    - **动态链接**：指向方法区中方法的符号引用。
    - **返回地址**：方法执行完毕后需要返回的位置。

#### **特点**
- **高效分配与回收**：栈内存的分配和释放由 JVM 自动管理（通过压栈和弹栈）。
- **容量有限**：栈的大小通过 `-Xss` 参数设置（默认 1MB，不同 JVM 可能不同），可能抛出 `StackOverflowError`。

---

### **2. 堆（Heap）**
#### **定义**
- **线程共享**：所有线程共享堆内存，用于存储对象实例和数组。
- **存储内容**：
  - **对象实例**：通过 `new` 关键字创建的对象。
  - **数组**：如 `int[] arr = new int[10];`。
  - **运行时常量池**：字符串常量（JDK 7 后移至堆中）。

#### **特点**
- **动态分配**：对象内存按需分配，由垃圾回收器（GC）自动回收。
- **容量可扩展**：堆的大小通过 `-Xms`（初始大小）和 `-Xmx`（最大大小）参数设置，可能抛出 `OutOfMemoryError`。
- **分代管理**：堆分为 **新生代**（Young Generation）、**老年代**（Old Generation）和**元空间**（Metaspace，JDK 8 后取代永久代），用于优化垃圾回收。

---

### **3. 栈与堆的区别**
| **特性**         | **栈（Stack）**                              | **堆（Heap）**                   |
| ---------------- | -------------------------------------------- | -------------------------------- |
| **存储内容**     | 局部变量（基本类型、对象引用）、方法调用栈帧 | 对象实例、数组、字符串常量池     |
| **线程共享性**   | 线程私有                                     | 线程共享                         |
| **内存分配速度** | 快速（固定大小，LIFO）                       | 较慢（动态分配，需 GC 管理）     |
| **生命周期**     | 随线程结束自动释放                           | 对象无引用时由 GC 回收           |
| **异常**         | `StackOverflowError`（栈溢出）               | `OutOfMemoryError`（堆内存不足） |

---

### **4. 栈与堆的联系**
#### **(1) 对象引用关系**
- **栈中存储对象引用**：  
  例如，方法中的局部变量 `Person p = new Person();`，变量 `p` 是对象的引用（存储在栈中），而 `Person` 对象实例本身存储在堆中。
- **堆中对象可能引用其他对象**：  
  堆中的对象可能包含成员变量（如 `Person` 对象的 `name` 字段），这些成员变量如果是对象类型，也会指向堆中的其他对象。

#### **(2) 方法执行依赖堆**
- 方法中操作的对象（如调用 `p.getName()`）必须通过栈中的引用访问堆中的对象实例。

#### **(3) 异常协作**
- **栈溢出**（`StackOverflowError`）：通常由递归调用过深导致。
- **堆溢出**（`OutOfMemoryError`）：通常由对象过多或内存泄漏导致。

#### **(4) 逃逸分析与栈上分配**
- **逃逸分析优化**：如果 JVM 检测到某个对象不会逃逸出方法（即不会被其他线程或方法访问），可能会直接在栈上分配该对象（而非堆中），以减少 GC 压力。

#### **(5) 方法区与堆的关系**
- **方法区**（JDK 8 后为元空间）：存储类的元数据、静态变量等。虽然逻辑上独立于堆，但某些 JVM 实现（如 HotSpot）将方法区作为堆的一部分管理。

---

### **5. 示例代码分析**
```java
public class StackHeapExample {
    public static void main(String[] args) {
        int age = 30;                  // 基本类型变量（存储在栈中）
        Person person = new Person();  // person 是引用（栈中），Person 对象在堆中
        person.setName("Alice");       // "Alice" 是字符串常量（堆中的字符串常量池）
    }
}

class Person {
    private String name;  // name 是引用（栈中），String 对象在堆中
    public void setName(String name) {
        this.name = name;
    }
}
```

#### **内存分配**
1. `age`（基本类型）直接存储在栈帧的局部变量表中。
2. `person` 是引用变量（栈中），指向堆中的 `Person` 对象实例。
3. `"Alice"` 是字符串常量，存储在堆的字符串常量池中。
4. `Person` 对象的 `name` 字段是引用，指向堆中的字符串对象。

---

### **6. 总结**
- **栈**：管理方法调用和局部变量，高效但容量有限。
- **堆**：管理对象实例和数组，动态分配但依赖 GC 回收。
- **联系**：栈通过引用操作堆中的对象，堆中的对象通过字段引用其他对象，二者共同支撑 Java 程序的运行。理解它们的区别与协作，对优化内存使用、排查内存泄漏和性能调优至关重要。



## 锁重入有什么用，一般什么时候用

锁重入（Reentrant Lock）是并发编程中的重要机制，它的核心作用是允许 **同一个线程多次获取同一把锁**，从而避免因自身重复请求锁而导致的死锁问题。这一机制在以下场景中尤为关键：

---

### **一、锁重入的作用**
1. **避免死锁**  
   如果锁不可重入，当线程在已持有锁的情况下再次请求同一把锁时，会因为等待自身释放锁而永久阻塞（死锁）。  
   **示例**：  
   ```java
   public synchronized void methodA() {
       methodB(); // 若锁不可重入，此处会阻塞
   }
   
   public synchronized void methodB() { 
       // 业务逻辑
   }
   ```
   - 可重入锁允许线程从 `methodA` 进入 `methodB`，无需释放锁再重新获取。

2. **简化代码设计**  
   在嵌套调用或递归场景中，开发者无需手动跟踪锁的获取和释放次数，降低代码复杂度。

3. **支持递归操作**  
   递归函数可能需要多次进入同步代码块，可重入锁确保递归逻辑正常执行。

---

### **二、锁重入的典型使用场景**
#### **1. 递归调用**
```java
public class Counter {
    private int count = 0;
    
    public synchronized void increment(int n) {
        if (n <= 0) return;
        count++;
        increment(n - 1); // 递归调用，锁重入避免死锁
    }
}
```
- 每次递归调用 `increment()` 时，线程会重复获取同一把锁。

#### **2. 嵌套同步代码块**
```java
public class DatabaseService {
    private final Object lock = new Object();
    
    public void writeData() {
        synchronized (lock) {
            validate(); // 嵌套调用需要同一把锁
            // 写入数据
        }
    }
    
    private void validate() {
        synchronized (lock) { // 锁重入
            // 校验逻辑
        }
    }
}
```
- 外层 `writeData()` 和内层 `validate()` 使用同一把锁，重入机制确保线程安全。

#### **3. 类继承中的同步方法**
```java
class Parent {
    public synchronized void parentMethod() {
        // 父类同步逻辑
    }
}

class Child extends Parent {
    @Override
    public synchronized void childMethod() {
        super.parentMethod(); // 调用父类同步方法（锁重入）
        // 子类逻辑
    }
}
```
- 子类方法 `childMethod()` 调用父类同步方法 `parentMethod()`，需要锁重入支持。

#### **4. 复杂业务逻辑的模块化**
```java
public class OrderProcessor {
    private final ReentrantLock lock = new ReentrantLock();
    
    public void processOrder() {
        lock.lock();
        try {
            checkInventory();
            calculatePrice();
            updateDatabase();
        } finally {
            lock.unlock();
        }
    }
    
    private void checkInventory() {
        lock.lock(); // 重入锁
        try {
            // 检查库存
        } finally {
            lock.unlock();
        }
    }
    
    // 其他方法同理...
}
```
- 通过 `ReentrantLock` 的显式重入特性，支持细粒度锁控制。

---

### **三、Java中的锁重入实现**
#### **1. `synchronized` 关键字**
- 底层通过 **对象头中的锁计数器** 实现重入。  
- 每次进入同步块，计数器 `+1`；退出时 `-1`，归零后释放锁。

#### **2. `ReentrantLock` 类**
- 通过 `AQS（AbstractQueuedSynchronizer）` 的 `state` 字段记录重入次数。  
- 必须显式调用 `lock()` 和 `unlock()`，且解锁次数需与加锁次数匹配。

---

### **四、锁重入的注意事项**
1. **避免过度同步**  
   锁重入虽然方便，但过度使用会导致锁持有时间过长，降低并发性能。  
   **优化方案**：缩小同步范围，使用更细粒度的锁。

2. **防止解锁次数不匹配**  
   使用 `ReentrantLock` 时，必须确保 `lock()` 和 `unlock()` 成对出现，否则可能导致锁泄漏或死锁。  
   **最佳实践**：在 `try-finally` 块中加锁/解锁。

3. **谨慎使用递归锁**  
   递归调用可能引发栈溢出（如 `StackOverflowError`），需控制递归深度。

---

### **五、总结**
| **场景**           | **锁重入的作用**           | **实现方式**                     |
| ------------------ | -------------------------- | -------------------------------- |
| 递归调用           | 确保递归函数正常执行       | `synchronized` / `ReentrantLock` |
| 嵌套同步代码块     | 简化多层级同步逻辑         | 对象锁计数器 / AQS state         |
| 类继承中的同步方法 | 支持父子类方法间的同步协作 | 隐式锁重入                       |
| 模块化业务逻辑     | 允许不同方法共享同一把锁   | 显式锁重入（如 `ReentrantLock`） |

锁重入是 Java 并发编程的基石之一，它通过 **避免自死锁** 和 **简化同步逻辑**，显著提升了多线程代码的健壮性和可维护性。理解其原理和使用场景，是编写高效并发程序的关键。

## 问题：偏向锁的使用场景是单线程，那我既然都单线程了，我为什么还要加锁呢

偏向锁（Biased Locking）的设计初衷并非针对严格的单线程程序，而是为了优化**在无竞争或极少竞争的多线程环境中的同步性能**。以下是详细解释其使用场景和存在意义：

---

### **1. 偏向锁的核心目标：减少无竞争场景的同步开销**
在大多数应用中，许多同步代码块（如 `synchronized` 方法或代码块）**实际运行时可能并未发生线程竞争**。例如：
- **初始化阶段**：对象仅在主线程中被初始化。
- **线程局部对象**：某些对象虽然可能被多线程访问，但在其生命周期中大部分时间仅被单个线程使用（如缓存中的线程私有数据）。
- **低竞争场景**：某些同步块仅在极少数情况下被多线程访问。

在这些场景中，偏向锁通过以下机制优化性能：
- **首次获取锁时**：将线程 ID 记录到对象头的 Mark Word 中，后续同一线程访问时无需任何同步操作（如 CAS）。
- **减少锁操作的开销**：避免了轻量级锁的 CAS 竞争和锁记录管理。

---

### **2. 看似“单线程”的加锁意义**
#### **(1) 代码的多线程兼容性**
- 即使当前代码在单线程下运行，**未来可能扩展为多线程**。例如：
  ```java
  // 当前单线程使用，但后续可能被多线程调用
  public class Singleton {
      private static Singleton instance;
      
      public static synchronized Singleton getInstance() {
          if (instance == null) {
              instance = new Singleton();
          }
          return instance;
      }
  }
  ```
  - 偏向锁在单线程初始化时优化性能，而当多线程首次访问时，锁会升级为轻量级或重量级锁。

#### **(2) 统一编程规范**
- 某些代码规范或框架要求对共享资源（如缓存、工具类）**默认加锁**，即使当前未发生竞争。例如：
  ```java
  // 工具类方法可能被多线程调用，需保证线程安全
  public class StringUtils {
      private static final Map<String, String> cache = new HashMap<>();
      
      public static synchronized String process(String input) {
          return cache.computeIfAbsent(input, k -> heavyProcessing(k));
      }
  }
  ```

#### **(3) 面向接口或库设计**
- 类库设计者无法预知用户的使用场景（单线程或多线程），因此**默认实现线程安全**。例如：
  ```java
  // JDK 中的 SimpleDateFormat 是非线程安全的，但开发者可能误用
  // 若设计为线程安全（如使用 synchronized），则偏向锁优化单线程性能
  public class SafeDateFormat {
      private static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
      
      public synchronized String format(Date date) {
          return sdf.format(date);
      }
  }
  ```

---

### **3. 偏向锁的实际运行场景**
#### **(1) 单线程初始化后的多线程访问**
- **对象初始化由主线程完成**（偏向锁生效），后续被多线程访问时，锁会升级为轻量级锁或重量级锁。  
- **示例**：Spring 容器的 Bean 初始化。

#### **(2) 线程局部性（Thread-Locality）**
- 某些对象虽然全局存在，但实际被不同线程**分时独占使用**。例如：
  ```java
  // 每个线程独立使用 Connection，但 Connection 对象可能被池化管理
  public class ConnectionPool {
      private Connection connection;
      
      public synchronized Connection getConnection() {
          if (connection == null) {
              connection = createConnection();
          }
          return connection;
      }
  }
  ```

---

### **4. 偏向锁的局限性**
#### **(1) 竞争触发锁升级**
- 一旦其他线程尝试获取锁，偏向锁会被撤销并升级为轻量级锁，带来额外开销。
- **适用场景**：适合**长期无竞争**或**竞争极低**的同步块。

#### **(2) 默认延迟启用**
- JVM 默认在启动后 4 秒（`BiasedLockingStartupDelay`）才启用偏向锁，避免初始化阶段的锁竞争干扰。

---

### **5. 何时需要手动关闭偏向锁？**
若应用明确处于高竞争环境（如高频多线程访问同步块），可通过 JVM 参数禁用偏向锁：
```bash
-XX:-UseBiasedLocking
```
此时所有锁直接进入轻量级锁模式，避免偏向锁撤销的开销。

---

### **总结**
| **场景**         | **偏向锁的作用**           | **加锁的意义**                 |
| ---------------- | -------------------------- | ------------------------------ |
| 单线程初始化     | 优化首次锁获取性能         | 为未来多线程扩展预留线程安全   |
| 低竞争多线程环境 | 减少无竞争时的同步开销     | 保证代码的线程安全兼容性       |
| 类库或框架设计   | 默认线程安全，避免用户误用 | 统一接口行为，隐藏多线程复杂性 |

偏向锁的存在意义是：**在无竞争或低竞争的多线程环境中，通过消除同步操作的开销，优化性能**。即使某些代码当前看似在单线程下运行，加锁仍然是为了保障**潜在多线程访问的线程安全**，而偏向锁则是为此场景提供了一种高效的优化手段。



## 问题：偏向锁的批量冲偏向

在 Java 中，**偏向锁的批量重偏向（Bulk Rebiasing）** 是 JVM 针对偏向锁撤销开销的优化机制。当某个类的偏向锁被频繁撤销时，JVM 会对该类的所有实例进行批量重偏向，将锁的偏向权转移给新的线程，避免逐个撤销的性能损耗。以下是其核心原理和实际应用场景：

---

### **一、为什么需要批量重偏向？**
#### **1. 偏向锁的撤销开销**
- **单次撤销**：当一个偏向锁被其他线程竞争时，JVM 需要：
  1. 暂停持有偏向锁的线程（STW）。
  2. 遍历线程栈，检查锁记录。
  3. 撤销偏向锁并升级为轻量级锁。
- **高频撤销**：若某个类的实例频繁被多线程访问，逐个撤销会显著降低性能。

#### **2. 批量重偏向的目标**
- **减少撤销操作次数**：将同一类的大量偏向锁一次性重偏向到新线程。
- **提升性能**：通过批量处理降低锁状态转换的开销。

---

### **二、触发条件**
JVM 通过两个阈值控制批量重偏向和批量撤销：

| **参数**                           | **默认值** | **作用**                                           |
| ---------------------------------- | ---------- | -------------------------------------------------- |
| `BiasedLockingBulkRebiasThreshold` | 20         | 触发批量重偏向的阈值（撤销次数达到该值时触发）     |
| `BiasedLockingBulkRevokeThreshold` | 40         | 触发批量撤销的阈值（撤销次数超过该值时禁用偏向锁） |

#### **触发流程**
1. **初始状态**：类的所有对象默认支持偏向锁。
2. **撤销计数**：当类的偏向锁撤销次数达到 `BulkRebiasThreshold`（默认20）时，触发批量重偏向。
3. **批量重偏向**：将该类所有实例的偏向锁偏向权转移给当前触发线程，重置撤销计数器。
4. **批量撤销**：若撤销次数累计达到 `BulkRevokeThreshold`（默认40），则禁用该类的偏向锁，后续新对象直接进入轻量级锁。

---

### **三、示例场景**
#### **场景：线程池处理任务**
```java
public class TaskProcessor {
    // 任务类，可能被多个线程处理
    static class Task {
        synchronized void execute() { /* ... */ }
    }

    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(4);
        List<Task> tasks = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            tasks.add(new Task());
        }
        // 多线程处理任务
        tasks.forEach(task -> pool.submit(task::execute));
        pool.shutdown();
    }
}
```
- **问题**：每个 `Task` 对象被不同线程执行，导致偏向锁频繁撤销。
- **优化**：
  - 当第20个 `Task` 的偏向锁被撤销时，触发批量重偏向，所有 `Task` 实例偏向权转移给新线程。
  - 后续线程可直接以偏向锁模式访问，减少竞争开销。

---

### **四、底层实现**
#### **1. 类偏向状态标记**
- 每个类在方法区中有一个 `prototype_header` 字段，记录该类的偏向锁状态。
- 批量重偏向时，修改 `prototype_header` 的 epoch 值（偏向锁版本号），后续新对象继承该值。

#### **2. 对象重偏向检查**
- 对象在加锁时检查当前类的 `epoch` 是否与对象头的 `epoch` 一致。
  - **一致**：允许偏向锁。
  - **不一致**：尝试通过 CAS 重偏向到当前线程。

---

### **五、监控与调优**
#### **1. 查看偏向锁信息**
通过 JVM 参数输出偏向锁统计：
```bash
-XX:+PrintFlagsFinal -XX:+PrintBiasedLockingStatistics
```
输出示例：
```
Biased Locking Statistics:
# 撤销次数
 Revocations: 50
# 批量重偏向次数
 Rebias: 2
# 批量撤销次数
 Bulk revocations: 1
```

#### **2. 调优参数**
- **调整阈值**（谨慎使用）：
  ```bash
  -XX:BiasedLockingBulkRebiasThreshold=30  # 调高触发阈值
  -XX:BiasedLockingBulkRevokeThreshold=60  # 调高禁用阈值
  ```
- **禁用偏向锁**（高竞争场景）：
  ```bash
  -XX:-UseBiasedLocking
  ```

---

### **六、适用场景与限制**
| **场景**               | **是否适用批量重偏向**     | **说明**                |
| ---------------------- | -------------------------- | ----------------------- |
| 对象被多个线程交替访问 | ✅ 适用                     | 如线程池处理任务        |
| 对象被长期单线程访问   | ❌ 不适用                   | 无需重偏向              |
| 高竞争环境             | ❌ 不适用（建议禁用偏向锁） | 直接使用轻量级/重量级锁 |

---

### **总结**
- **批量重偏向**是 JVM 对偏向锁撤销开销的优化，适用于**同一类实例被多线程交替访问**的场景。
- 通过动态调整类的偏向锁状态，减少锁升级的开销，提升中低竞争场景的性能。
- 开发者可通过监控和调优参数，结合业务场景选择最合适的锁机制。



# wait

## 问题：Java的wait和notify

在Java中，`wait()`和`notify()`/`notifyAll()`是用于**线程间协作**的关键方法，它们依赖于对象的**监视器锁（monitor lock）**，是多线程编程中实现等待/通知机制的核心工具。以下是它们的核心原理、用法和注意事项：

---

### **一、基本概念**
#### 1. **所属类**
- 定义在`Object`类中，因此所有对象均可作为锁。
- **为什么不在`Thread`类中**？  
  锁是对象级别的（每个对象关联一个monitor），`wait`和`notify`需要基于锁操作，因此属于`Object`。

#### 2. **核心作用**
- **`wait()`**：使当前线程释放锁并进入等待状态，直到其他线程调用`notify()`或`notifyAll()`唤醒它。
- **`notify()`**：随机唤醒一个在该对象上等待的线程。
- **`notifyAll()`**：唤醒所有在该对象上等待的线程。

---

### **二、使用前提**
**必须持有对象的监视器锁**（即在`synchronized`代码块或方法中调用），否则抛出`IllegalMonitorStateException`。

#### 示例代码
```java
public class WaitNotifyExample {
    private final Object lock = new Object();
    private boolean condition = false;

    public void waitMethod() throws InterruptedException {
        synchronized (lock) {
            while (!condition) { // 循环检查条件，防止虚假唤醒
                lock.wait(); // 释放锁并等待
            }
            // 条件满足后执行任务
        }
    }

    public void notifyMethod() {
        synchronized (lock) {
            condition = true;
            lock.notifyAll(); // 唤醒所有等待线程
        }
    }
}
```

---

### **三、核心原理**
#### 1. **等待队列（Wait Set）**
- 调用`wait()`的线程会被放入对象的**等待队列**，并释放锁。
- 线程状态变为`WAITING`或`TIMED_WAITING`（若使用`wait(long timeout)`）。

#### 2. **通知机制**
- `notify()`从等待队列中随机选择一个线程，将其移至**入口队列（Entry Set）**，等待重新竞争锁。
- `notifyAll()`将所有等待线程移至入口队列。

#### 3. **锁竞争**
- 被唤醒的线程需重新竞争锁，成功获取锁后才能继续执行。

---

### **四、使用场景**
#### 1. **生产者-消费者模型**
```java
public class Buffer {
    private Queue<Integer> queue = new LinkedList<>();
    private int maxSize = 10;

    public synchronized void produce(int value) throws InterruptedException {
        while (queue.size() == maxSize) {
            wait(); // 缓冲区满，生产者等待
        }
        queue.add(value);
        notifyAll(); // 唤醒消费者
    }

    public synchronized int consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait(); // 缓冲区空，消费者等待
        }
        int value = queue.poll();
        notifyAll(); // 唤醒生产者
        return value;
    }
}
```

#### 2. **任务协调**
- 主线程等待子线程完成任务后继续执行。

---

### **五、注意事项**
#### 1. **虚假唤醒（Spurious Wakeup）**
- 线程可能在没有调用`notify()`的情况下被唤醒。
- **解决方案**：始终在循环中检查条件，而非使用`if`。
  
  ```java
  synchronized (lock) {
      while (!condition) { // 用while，不是if！
          lock.wait();
      }
  }
  ```

#### 2. **优先使用`notifyAll()`**
- `notify()`可能无法唤醒目标线程，导致线程饥饿。
- `notifyAll()`更安全，但可能引发性能问题（需权衡）。

#### 3. **死锁风险**
- 若多个线程因不同条件等待，需确保唤醒条件精准匹配。

#### 4. **超时机制**
- 使用`wait(long timeout)`避免永久等待。

---

### **六、与`sleep()`的区别**
| **特性**     | `wait()`                         | `sleep()`      |
| ------------ | -------------------------------- | -------------- |
| **锁释放**   | 释放锁                           | 不释放锁       |
| **调用位置** | 必须在`synchronized`块中         | 任意位置       |
| **唤醒方式** | 需`notify()`/`notifyAll()`或超时 | 超时后自动恢复 |
| **所属类**   | `Object`                         | `Thread`       |

---

### **七、替代方案**
现代并发编程中，更推荐使用以下工具：
1. **`Lock`与`Condition`**  
   ```java
   Lock lock = new ReentrantLock();
   Condition condition = lock.newCondition();
   
   condition.await();      // 类似wait()
   condition.signal();     // 类似notify()
   condition.signalAll();  // 类似notifyAll()
   ```
   - 支持多个等待条件队列，灵活性更高。

2. **高级并发工具**  
   
   - `BlockingQueue`（如`ArrayBlockingQueue`）
   - `CountDownLatch`、`CyclicBarrier`、`Semaphore`

---

### **总结**
- `wait()`和`notify()`是Java线程协作的基础，但需严格遵循使用规则。
- 核心要点：**锁内调用、循环检查条件、避免虚假唤醒**。
- 在高并发场景中，优先考虑更安全的并发工具类（如`java.util.concurrent`包）。