在算法解题中，熟练掌握 Java 自带数据结构的核心 API 至关重要。以下是必会数据结构的总结及关键 API，按使用频率排序：

---

### **1. 数组 (Array)**
**核心操作**：索引访问、排序、二分查找
```java
// 初始化
int[] arr = new int[10]; 
String[] strArr = {"a", "b", "c"};

// 常用工具类 Arrays
Arrays.sort(arr);                     // 排序（升序）
Arrays.fill(arr, 0);                  // 填充默认值
int pos = Arrays.binarySearch(arr, 5); // 二分查找（需先排序）
int[] copy = Arrays.copyOf(arr, len); // 复制数组
String s = Arrays.toString(arr);      // 转字符串
```

---

### **2. 动态数组 (ArrayList)**
**核心操作**：动态扩容、随机访问
```java
List<Integer> list = new ArrayList<>();

// 增删改查
list.add(10);                         // 尾部添加
list.add(0, 5);                       // 指定索引插入
list.remove(0);                       // 按索引删除
list.remove(Integer.valueOf(5));      // 按值删除
list.set(0, 100);                     // 修改索引0的值
int val = list.get(0);                // 获取索引0的值

// 常用操作
int size = list.size();               // 元素数量
boolean isEmpty = list.isEmpty();     // 判空
boolean exists = list.contains(5);    // 存在性检查
Collections.sort(list);               // 排序（需 Collections 类）
```

---

### **3. 哈希表 (HashMap)**
**核心操作**：O(1) 查找、键值映射
```java
Map<String, Integer> map = new HashMap<>();

// 增删改查
map.put("key", 10);                   // 添加/更新键值对
map.remove("key");                    // 删除键
int val = map.get("key");             // 获取值（键不存在返回 null）
map.getOrDefault("key", 0);           // 安全获取（避免 NullPointerException）

// 常用操作
boolean exists = map.containsKey("key"); // 检查键是否存在
int size = map.size();                // 键值对数量
Set<String> keys = map.keySet();      // 获取所有键
Collection<Integer> vals = map.values(); // 获取所有值

// 遍历（Java 8+）
map.forEach((k, v) -> System.out.println(k + ":" + v));
```

---

### **4. 双端队列 (ArrayDeque)**
**核心操作**：栈/队列操作、高效头尾操作
```java
Deque<Integer> deque = new ArrayDeque<>();

// 作为队列使用（FIFO）
deque.offer(10);                      // 队尾添加（推荐）
deque.poll();                         // 队头弹出（推荐）

// 作为栈使用（LIFO）
deque.push(10);                       // 栈顶压入
deque.pop();                          // 栈顶弹出

// 其他操作
int first = deque.peekFirst();        // 查看队头（不删除）
int last = deque.peekLast();          // 查看队尾（不删除）
boolean isEmpty = deque.isEmpty();
```

---

### **5. 优先队列 (PriorityQueue)**
**核心操作**：堆实现、快速获取最值
```java
// 最小堆（默认）
Queue<Integer> minHeap = new PriorityQueue<>(); 

// 最大堆（自定义比较器）
Queue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);

minHeap.offer(5);                     // 添加元素
int top = minHeap.peek();             // 查看堆顶（不删除）
minHeap.poll();                       // 弹出堆顶
int size = minHeap.size();
```

---

### **6. 集合 (HashSet)**
**核心操作**：去重、O(1) 存在性检查
```java
Set<Integer> set = new HashSet<>();
set.add(10);                          // 添加元素
set.remove(10);                       // 删除元素
boolean exists = set.contains(10);    // 存在性检查
int size = set.size();
```

---

### **7. 字符串 (String)**
**核心操作**：不可变、常用工具方法
```java
String s = "hello";

// 基础操作
int len = s.length();                 // 长度
char c = s.charAt(0);                 // 获取字符
String sub = s.substring(0, 3);       // 截取子串 [0,3)

// 转换
char[] chars = s.toCharArray();       // 转字符数组
String numStr = String.valueOf(100);  // 数字转字符串

// 工具方法
s.trim();                             // 去除首尾空格
s.toLowerCase();                      // 转小写
s.split(",");                         // 分割字符串
s.replace("a", "b");                  // 替换字符
```

---

### **8. 链表 (LinkedList)**
**核心操作**：高效头尾操作（比 ArrayList 更适合频繁插入删除）
```java
List<Integer> list = new LinkedList<>();
// API 同 ArrayList（但底层为链表）
list.addFirst(1);                     // 头部插入（特有）
list.addLast(2);                      // 尾部插入（特有）
```

---

### **关键技巧总结**
1. **排序**：
   - 数组 → `Arrays.sort(arr)`
   - 集合 → `Collections.sort(list)`
2. **二分查找**：
   - 数组 → `Arrays.binarySearch(arr, key)`
3. **高频数据结构选择**：
   - 快速查找 → `HashMap`
   - 排序/最值 → `PriorityQueue`
   - 栈/队列 → `ArrayDeque`
   - 动态数组 → `ArrayList`
4. **避免常见错误**：
   - `ArrayList.remove(int index)` vs `remove(Object)`（用 `Integer.valueOf()` 区分）
   - `Map.get()` 返回 `null` 时使用 `getOrDefault()`
   - `String` 比较用 `equals()` 而非 `==`

掌握这些 API 可覆盖 90% 的算法题目需求，建议通过刷题熟练使用场景（如 BFS 用 `Queue`，DFS 用 `Stack`，去重用 `Set` 等）。