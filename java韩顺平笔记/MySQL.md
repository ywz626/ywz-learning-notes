# Day01 

## Mysql常用命令

以下是MySQL数据库的常用命令整理，涵盖数据库操作、表管理、数据增删改查、用户权限等场景：

---

### **一、数据库操作**
1. **查看所有数据库**  
   ```sql
   SHOW DATABASES;
   ```

2. **创建数据库**  
   ```sql
   CREATE DATABASE db_name;
   -- 指定字符集
   CREATE DATABASE db_name DEFAULT CHARSET utf8mb4;
   ```

3. **删除数据库**  
   
   ```sql
   DROP DATABASE db_name;
   ```
   
4. **切换数据库**  
   
   ```sql
   USE db_name;
   ```

---

### **二、表操作**
1. **查看所有表**  
   ```sql
   SHOW TABLES;
   ```

2. **查看表结构**  
   ```sql
   DESC table_name;    -- 简写
   DESCRIBE table_name;
   ```

3. **创建表**  
   ```sql
   CREATE TABLE table_name (
       id INT PRIMARY KEY AUTO_INCREMENT,
       name VARCHAR(50) NOT NULL,
       age INT DEFAULT 0,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```

4. **删除表**  
   
   ```sql
   DROP TABLE table_name;
   ```
   
5. **修改表名**  
   
   ```sql
   ALTER TABLE old_table RENAME TO new_table;
   ```
   
6. **添加列**  
   ```sql
   ALTER TABLE table_name ADD COLUMN email VARCHAR(100);
   ```

7. **删除列**  
   ```sql
   ALTER TABLE table_name DROP COLUMN email;
   ```

8. **添加索引**  
   
   ```sql
   CREATE INDEX idx_name ON table_name (column_name);
   ```

---

### **三、数据操作（DML）**
1. **插入数据**  
   ```sql
   INSERT INTO table_name (name, age) VALUES ('Alice', 25);
   -- 批量插入
   INSERT INTO table_name (name, age) VALUES ('Bob', 30), ('Charlie', 28);
   ```

2. **查询数据**  
   
   ```sql
   SELECT * FROM table_name;                     -- 所有字段
   SELECT name, age FROM table_name WHERE age > 20; -- 条件查询
   SELECT DISTINCT age FROM table_name;           -- 去重
   ```
   
3. **更新数据**  
   
   ```sql
   UPDATE table_name SET age = 26 WHERE name = 'Alice';
   ```
   
4. **删除数据**  
   
   ```sql
   DELETE FROM table_name WHERE id = 1;           -- 按条件删除
   TRUNCATE TABLE table_name;                     -- 清空表（不可回滚）
   ```

---

### **四、查询进阶**
1. **排序（ORDER BY）**  
   ```sql
   SELECT * FROM table_name ORDER BY age DESC;    -- 降序
   ```

2. **分页（LIMIT）**  
   ```sql
   SELECT * FROM table_name LIMIT 10 OFFSET 20;   -- 第3页（每页10条）
   ```

3. **分组统计（GROUP BY）**  
   ```sql
   SELECT age, COUNT(*) FROM table_name GROUP BY age;
   ```

4. **连接查询（JOIN）**  
   
   ```sql
   -- 内连接
   SELECT a.name, b.order_id 
   FROM users a 
   JOIN orders b ON a.id = b.user_id;
   ```
   
5. **模糊查询（LIKE）**  
   
   ```sql
   SELECT * FROM table_name WHERE name LIKE 'A%'; -- 以A开头
   ```

---

### **五、用户与权限**
1. **创建用户**  
   ```sql
   CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
   ```

2. **授权**  
   ```sql
   GRANT SELECT, INSERT ON db_name.* TO 'username'@'localhost';
   -- 授予所有权限
   GRANT ALL PRIVILEGES ON *.* TO 'username'@'%';
   ```

3. **撤销权限**  
   ```sql
   REVOKE INSERT ON db_name.* FROM 'username'@'localhost';
   ```

4. **删除用户**  
   ```sql
   DROP USER 'username'@'localhost';
   ```

5. **刷新权限**  
   ```sql
   FLUSH PRIVILEGES;
   ```

---

### **六、备份与恢复**
1. **导出数据库**  
   ```bash
   mysqldump -u root -p db_name > backup.sql
   ```

2. **导入数据库**  
   ```bash
   mysql -u root -p db_name < backup.sql
   ```

---

### **七、其他常用命令**
1. **查看MySQL版本**  
   ```sql
   SELECT VERSION();
   ```

2. **查看当前用户**  
   
   ```sql
   SELECT USER();
   ```
   
3. **查看进程列表**  
   
   ```sql
   SHOW PROCESSLIST;
   ```
   
4. **终止查询**  
   
   ```sql
   KILL process_id;  -- 根据SHOW PROCESSLIST的Id
   ```

---

### **八、事务控制**
1. **开启事务**  
   ```sql
   START TRANSACTION;
   ```

2. **提交事务**  
   ```sql
   COMMIT;
   ```

3. **回滚事务**  
   ```sql
   ROLLBACK;
   ```

---

### **注意事项**
- **SQL关键字不区分大小写**，但建议统一风格（如关键字大写）。
- 字符串使用单引号 `'`，字段名可反引号包裹（避免与关键字冲突）。
- `WHERE` 条件避免全表扫描，合理使用索引。

掌握这些命令可覆盖日常开发中80%的MySQL操作！

## Mysql语法

以下是SQL的必学核心语法总结，涵盖数据定义（DDL）、数据操作（DML）、数据查询（DQL）、数据控制（DCL）和事务控制（TCL），适合快速掌握SQL核心能力：

---

### **一、数据定义（DDL）**
1. **创建表**  
   
   ```sql
   CREATE TABLE users (
       id INT PRIMARY KEY AUTO_INCREMENT,  -- 主键自增
       name VARCHAR(50) NOT NULL,          -- 非空约束
       email VARCHAR(100) UNIQUE,          -- 唯一约束
       age INT DEFAULT 18,                 -- 默认值
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```
   
2. **修改表**  
   
   ```sql
   -- 添加列
   ALTER TABLE users ADD COLUMN phone VARCHAR(20);
   -- 删除列
   ALTER TABLE users DROP COLUMN phone;
   -- 修改列类型
   ALTER TABLE users MODIFY COLUMN name VARCHAR(100);
   ```
   
3. **删除表**  
   ```sql
   DROP TABLE users;
   ```

---

### **二、数据操作（DML）**
1. **插入数据**  
   
   ```sql
   INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');
   ```
   
2. **更新数据**  
   
   ```sql
   UPDATE users SET age = 25 WHERE id = 1;
   ```
   
3. **删除数据**  
   ```sql
   DELETE FROM users WHERE id = 1;    -- 按条件删除
   TRUNCATE TABLE users;              -- 清空表（不可回滚）
   ```

---

### **三、数据查询（DQL）**
1. **基础查询**  
   
   ```sql
   SELECT * FROM users;                     -- 查询所有字段
   SELECT name, age FROM users WHERE age > 20; -- 条件筛选
   SELECT DISTINCT age FROM users;           -- 去重查询
   ```
   
2. **聚合函数**  
   
   ```sql
   SELECT COUNT(*) FROM users;              -- 统计行数
   SELECT AVG(age) FROM users;              -- 平均值
   SELECT SUM(age) FROM users WHERE age > 18; -- 条件求和
   ```
   
3. **分组与过滤**  
   
   ```sql
   SELECT age, COUNT(*) FROM users 
   GROUP BY age 
   HAVING COUNT(*) > 1;  -- HAVING对分组后结果过滤
   ```
   
4. **排序与分页**  
   
   ```sql
   SELECT * FROM users 
   ORDER BY age DESC, name ASC  -- 先按年龄降序，再按名字升序
   LIMIT 10 OFFSET 20;          -- 分页（每页10条，第3页）
   ```
   
5. **多表连接**  
   
   ```sql
   -- 内连接（交集）
   SELECT u.name, o.order_id 
   FROM users u 
   JOIN orders o ON u.id = o.user_id;
   
   -- 左连接（左表全保留）
   SELECT u.name, o.order_id 
   FROM users u 
   LEFT JOIN orders o ON u.id = o.user_id;
   ```
   
6. **子查询**  
   
   ```sql
   SELECT name FROM users 
   WHERE age > (SELECT AVG(age) FROM users); -- 子查询作为条件
   ```

---

### **四、数据控制（DCL）**
1. **用户权限**  
   ```sql
   GRANT SELECT, INSERT ON db.users TO 'user1'@'localhost';
   REVOKE DELETE ON db.users FROM 'user1'@'localhost';
   ```

---

### **五、事务控制（TCL）**
1. **事务操作**  
   
   ```sql
   START TRANSACTION;        -- 开启事务
   INSERT INTO orders (...) VALUES (...);
   UPDATE products SET stock = stock - 1 WHERE id = 100;
   COMMIT;                   -- 提交事务（或 ROLLBACK 回滚）
   ```

---

### **六、常用函数**
1. **字符串函数**  
   
   ```sql
   SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;
   SELECT SUBSTRING(email, 1, 5) FROM users;      -- 截取前5字符
   SELECT UPPER(name) FROM users;                 -- 转大写
   ```
   
2. **日期函数**  
   
   ```sql
   SELECT NOW();                        -- 当前时间
   SELECT DATE_FORMAT(created_at, '%Y-%m-%d') FROM users; -- 格式化日期
   ```
   
3. **条件函数**  
   
   ```sql
   SELECT name, 
       CASE WHEN age < 18 THEN '未成年' 
            ELSE '成年' 
       END AS age_group 
   FROM users;
   ```

---

### **七、索引优化**
1. **创建索引**  
   ```sql
   CREATE INDEX idx_age ON users (age);          -- 单列索引
   CREATE INDEX idx_name_age ON users (name, age); -- 组合索引
   ```

2. **查看索引**  
   ```sql
   SHOW INDEX FROM users;
   ```

---

### **八、必学高级语法**
1. **窗口函数**（统计排名）  
   ```sql
   SELECT name, age, 
       RANK() OVER (ORDER BY age DESC) AS rank 
   FROM users;
   ```

2. **通用表表达式（CTE）**  
   ```sql
   WITH cte AS (
       SELECT * FROM users WHERE age > 20
   )
   SELECT * FROM cte;
   ```

3. **UNION 合并结果集**  
   ```sql
   SELECT name FROM users 
   UNION 
   SELECT name FROM admins;  -- 去重合并
   ```

---

### **九、注意事项**
1. **避免全表扫描**：WHERE条件尽量使用索引字段。
2. **性能优化**：使用`EXPLAIN`分析查询计划。
3. **SQL注入**：使用参数化查询（PreparedStatement）而非字符串拼接。
4. **事务隔离级别**：了解`READ COMMITTED`、`REPEATABLE READ`等隔离级别。

---

### **总结**  
掌握以上语法可覆盖90%的SQL使用场景。实际开发中需结合具体数据库（如MySQL、PostgreSQL）的方言调整语法细节，并关注索引优化和事务控制。

## 基础查询操作

select 字段 

from 表

where  条件

多个字段查询用逗号隔开

字段as …可以给列起别名

select * from  查询所有字段

条件查询  and的优先级比or 高，需要使用括号

模糊查询：“%d%” 查询有d的数据

%表示任意单位长度字段

_表示一个单位长度

排序  select  字段

from 表

order by   desc  降序排序,asc 升序排序

练习题 :找出薪资在一千二百五到三千之间的员工,要求按照薪资降序

select name

from biao

where sal>=1250 and sal<=3000

order by  sal desc;

## 单行处理函数

SQL 中常用的单行处理函数主要用于逐行处理数据，每个函数对单行输入返回一个结果。以下是常见的分类及函数示例，不同数据库可能存在语法差异：

---

### **1. 字符串函数**
- **`UPPER(str)` / `LOWER(str)`**：将字符串转为全大写/小写。
- **`CONCAT(str1, str2)`**：拼接多个字符串（Oracle 也可用 `||`）。
- **`SUBSTR(str, start, length)`** / `SUBSTRING`：截取子字符串。//start下标从1开始
- **`TRIM([LEADING|TRAILING|BOTH] 'char' FROM str)`**：去除首尾空格或指定字符。
- **`LENGTH(str)`** / `LEN()`：返回字符串长度。
- **`REPLACE(str, old, new)`**：替换字符串中的内容。
- **`INSTR(str, substring)`**：返回子字符串的位置（类似 `CHARINDEX`）。

---

### **2. 数值函数**
- **`ROUND(num, decimals)`**：四舍五入到指定小数位。
- **`TRUNC(num, decimals)`**：截断小数位（不四舍五入）。
- **`CEIL(num)` / `FLOOR(num)`**：向上/向下取整。
- **`ABS(num)`**：返回绝对值。
- **`MOD(num, divisor)`**：取余数。
- **`POWER(num, exponent)`**：计算幂次方。

---

### **3. 日期函数**
- **`CURRENT_DATE` / `NOW()` / `SYSDATE`**：获取当前日期时间。
- **`DATE_ADD(date, INTERVAL)` / `ADD_MONTHS(date, n)`**：日期增减（如加1个月）。
- **`DATEDIFF(end, start)` / `MONTHS_BETWEEN`**：计算日期差值。
- **`EXTRACT(YEAR|MONTH|DAY FROM date)`**：提取日期部分。
- **`TO_CHAR(date, format)`**：将日期转为字符串（如 `'YYYY-MM-DD'`）。
- **`TO_DATE(str, format)`**：将字符串转为日期。

---

### **4. 转换函数**
- **`CAST(expr AS type)`**：通用类型转换（如 `CAST('123' AS INT)`）。
- **`COALESCE(expr1, expr2, ...)`**：返回第一个非空值。
- **`NVL(expr, default)`**（Oracle）/ `IFNULL(expr, default)`（MySQL）/ `ISNULL(expr, default)`（SQL Server）：替换空值为默认值。
- **`NULLIF(expr1, expr2)`**：两值相等时返回 `NULL`，否则返回第一个值。

---

### **5. 条件函数**
- **`CASE WHEN`**：条件分支判断。
  ```sql
  CASE WHEN score >= 90 THEN 'A'
       WHEN score >= 80 THEN 'B'
       ELSE 'C' END
  ```
- **`DECODE`（Oracle）**：简化条件判断。
  
  ```sql
  DECODE(column, value1, result1, value2, r
  ```

### **6. 其他函数**
- **`NVL2(expr, val_if_not_null, val_if_null)`**（Oracle）：根据是否为空返回不同值。
- **`GREATEST(val1, val2, ...)` / `LEAST`**：返回多个值中的最大/最小值。
- **`COALESCE`**：跨数据库通用的空值处理函数。

---

### **注意**
- **数据库差异**：不同数据库（如 MySQL、Oracle、SQL Server）的函数名或用法可能不同（例如日期处理函数）。
- **函数组合**：单行函数可嵌套使用（如 `UPPER(SUBSTR(name, 1, 1))`）。

这些函数能高效处理单行数据，常用于查询结果的格式化、计算或转换。

## 分组函数

以下是 SQL 中常用的分组函数（聚合函数）及其核心总结，用于对数据集进行分组后的统计计算：

---

### **1. 常用分组函数**
- **`COUNT(expr)`**：统计行数（`COUNT(*)` 统计所有行，`COUNT(列名)` 忽略 NULL）。
- **`SUM(expr)`**：计算数值列的总和（忽略 NULL）。
- **`AVG(expr)`**：计算数值列的平均值（忽略 NULL）。
- **`MAX(expr)`**：返回列中的最大值（适用于数值、日期、字符串）。
- **`MIN(expr)`**：返回列中的最小值（同上）。
- **`GROUP_CONCAT(expr)`**（MySQL）/ **`STRING_AGG(expr, separator)`**（SQL Server/PostgreSQL）：将多行值合并为字符串。
- **`STDDEV(expr)`** / **`VAR(expr)`**：计算标准差或方差（统计类函数）。

---

### **2. 分组函数特性**
1. **与 `GROUP BY` 配合使用**：  
   分组函数通常与 `GROUP BY` 子句结合，按指定列分组后统计。
   
   ```sql
   SELECT department, AVG(salary) 
   FROM employees 
   GROUP BY department;
   ```
   
2. **`HAVING` 过滤分组结果**：  
   分组后的结果用 `HAVING` 过滤（`WHERE` 在分组前过滤行）。
   
   ```sql
   SELECT department, AVG(salary)
   FROM employees
   GROUP BY department
   HAVING AVG(salary) > 5000;
   ```
   
3. **对 NULL 值的处理**：  
   
   - `COUNT(列名)` 忽略 NULL 值。  
   - `SUM`、`AVG`、`MAX`、`MIN` 自动忽略 NULL 值。  
   - 若需统计 NULL，可用 `COUNT(*)` 或 `COALESCE` 转换。
   
4. **结合 `DISTINCT` 去重统计**：  
   
   ```sql
   SELECT COUNT(DISTINCT department) 
   FROM employees; -- 统计不重复的部门数量
   ```

---

### **3. 高级分组函数**
- **`ROLLUP`**：生成多级分组的小计和总计（层次化聚合）。
  
  ```sql
  SELECT department, job, SUM(salary)
  FROM employees
  GROUP BY ROLLUP(department, job); -- 按部门+职位、部门、总计三级统计
  ```
  
- **`CUBE`**：生成所有可能的组合小计（多维聚合）。
- **`GROUPING SETS`**：自定义分组维度组合。

---

### **4. 注意事项**
- **`SELECT` 中的非聚合列**：  
  未在 `GROUP BY` 中出现的列，必须使用聚合函数包裹（如 `MAX(column)`）。
  ```sql
  -- 错误示例（非聚合列 name 未在 GROUP BY 中）：
  SELECT department, name, AVG(salary) FROM employees GROUP BY department;
  
  -- 正确示例：
  SELECT department, MAX(name), AVG(salary) FROM employees GROUP BY department;
  ```

- **数据库差异**：  
  - MySQL 允许 `GROUP BY` 后省略 `SELECT` 中的非聚合列（非严格模式）。  
  - PostgreSQL/Oracle 等严格遵循 SQL 标准，要求完全匹配。

---

### **5. 典型场景示例**
1. **统计每个地区的销售总额**：
   ```sql
   SELECT region, SUM(sales) 
   FROM orders 
   GROUP BY region;
   ```

2. **计算各部门的最高和最低工资**：
   ```sql
   SELECT department, MAX(salary), MIN(salary)
   FROM employees
   GROUP BY department;
   ```

3. **按年月统计订单数量**：
   ```sql
   SELECT EXTRACT(YEAR FROM order_date) AS year,
          EXTRACT(MONTH FROM order_date) AS month,
          COUNT(*) AS order_count
   FROM orders
   GROUP BY year, month;
   ```

---

### **总结**
- 分组函数用于 **聚合多行数据**，生成分组统计结果。
- 核心操作是 `GROUP BY` + 分组函数，过滤分组结果用 `HAVING`。
- 注意处理 NULL 值及不同数据库的分组语法差异。

## 分组查询

以下是 SQL 中 **分组查询（GROUP BY 查询）** 的核心总结，涵盖语法、规则、常见用法及注意事项：

---

### **1. 分组查询的核心作用**
对数据集按指定列分组，**对每个分组进行聚合统计**（如求和、平均、计数等）。  
**典型场景**：按部门统计薪资、按地区计算销售额、按日期分组统计订单量等。

---

### **2. 基本语法**
```sql
SELECT 
   分组列, 
   聚合函数(列) AS 别名 
FROM 表名 
WHERE 过滤条件（分组前过滤行）
GROUP BY 分组列 
HAVING 过滤条件（分组后过滤组）
ORDER BY 排序列;
```

#### **执行顺序**  
`FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`

---

### **3. 关键规则与用法**

#### **(1) 分组列与聚合列**
- **`GROUP BY` 后的列**：决定分组的维度（如 `department`, `year`）。  
- **`SELECT` 中的非分组列**：必须使用聚合函数包裹（如 `MAX(name)`, `SUM(sales)`）。

**错误示例**：  
```sql
-- 错误：name 未在 GROUP BY 中，也未用聚合函数包裹
SELECT department, name, AVG(salary) 
FROM employees 
GROUP BY department;
```

**正确写法**：  

```sql
-- 正确：name 用 MAX 聚合或从 GROUP BY 中添加
SELECT department, MAX(name) AS latest_name, AVG(salary) 
FROM employees 
GROUP BY department;
```

---

#### **(2) `WHERE` vs `HAVING`**
- **`WHERE`**：在分组前过滤行（无法使用聚合函数）。  
- **`HAVING`**：在分组后过滤组（可以使用聚合函数）。

**示例**：  
```sql
-- 统计部门平均工资 > 5000 的部门
SELECT department, AVG(salary) 
FROM employees 
WHERE salary > 3000  -- 先过滤掉工资 <=3000 的员工
GROUP BY department 
HAVING AVG(salary) > 5000;  -- 再过滤平均工资低的部门
```

---

#### **(3) 多列分组**
按多列组合分组，生成更细粒度的统计结果：  
```sql
-- 按部门和职位统计平均工资
SELECT department, job, AVG(salary)
FROM employees
GROUP BY department, job;
```

---

#### **(4) 结合 `DISTINCT` 去重统计**
在聚合函数内使用 `DISTINCT` 去重：  
```sql
-- 统计部门内不重复的职位数量
SELECT department, COUNT(DISTINCT job) 
FROM employees 
GROUP BY department;
```

---

### **4. 高级分组操作**

#### **(1) `ROLLUP` 层次化小计**
生成多级分组的小计和总计（如按部门+职位、部门、总计三级统计）：  
```sql
-- MySQL / PostgreSQL
SELECT department, job, SUM(salary)
FROM employees
GROUP BY ROLLUP(department, job);

-- SQL Server
SELECT department, job, SUM(salary)
FROM employees
GROUP BY department, job WITH ROLLUP;
```

#### **(2) `CUBE` 多维小计**
生成所有可能的组合小计（适用于多维分析）：  
```sql
-- PostgreSQL / SQL Server
SELECT department, job, SUM(salary)
FROM employees
GROUP BY CUBE(department, job);
```

#### **(3) `GROUPING SETS` 自定义分组维度**
灵活指定多个分组维度组合：  
```sql
-- 同时按部门和按职位统计
SELECT department, job, SUM(salary)
FROM employees
GROUP BY GROUPING SETS (department, job);
```

---

### **5. 注意事项**

#### **(1) 性能优化**
- 避免对高基数（唯一值多）的列分组（如 `user_id`），可能导致分组数过多，性能下降。  
- 结合索引优化分组列（如对 `department` 建索引）。

#### **(2) 空值处理**
- `GROUP BY` 会将 `NULL` 值视为同一分组。  
- 聚合函数（如 `SUM`、`AVG`）自动忽略 `NULL` 值。

#### **(3) 数据库差异**
- **MySQL**：宽松模式允许 `SELECT` 中的非聚合列不在 `GROUP BY` 中（但可能返回不确定值）。  
- **Oracle/SQL Server/PostgreSQL**：严格遵循 SQL 标准，要求完全匹配。

---

### **6. 典型场景示例**

#### **(1) 按日期统计订单量**
```sql
SELECT 
    DATE(order_time) AS order_date,
    COUNT(*) AS total_orders,
    SUM(amount) AS total_amount
FROM orders
GROUP BY DATE(order_time)
ORDER BY order_date;
```

#### **(2) 统计各部门薪资分布**
```sql
SELECT 
    department,
    AVG(salary) AS avg_salary,
    MAX(salary) AS max_salary,
    MIN(salary) AS min_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 5000;
```

---

### **总结**
- **分组查询 = `GROUP BY` + 聚合函数**，用于多维数据聚合统计。  
- **`WHERE` 先过滤行，`HAVING` 后过滤组**，二者分工明确。  
- 高级分组（`ROLLUP`/`CUBE`）支持复杂报表需求，但需注意性能影响。

# Day02

`LIMIT` 和 `OFFSET` 子句通常和`ORDER BY` 语句一起使用，当我们对整个结果集排序之后，我们可以 `LIMIT`来指定只返回多少行结果 ,用 `OFFSET`来指定从哪一行开始返回。你可以想象一下从一条长绳子剪下一小段的过程，我们通过 `OFFSET` 指定从哪里开始剪，用 `LIMIT` 指定剪下多少长度。

limit offset

## 多表查询

### 内连接

---

### **MySQL 多表查询之内连接（INNER JOIN）核心总结**

内连接（INNER JOIN）用于根据两个或多个表之间的关联列，**返回所有满足连接条件的匹配行**。未匹配的行（任一表中无对应值）将被排除。

---

### ***1. 核心语法***
#### **(1) 显式 `INNER JOIN` 语法**
```sql
SELECT 列名
FROM 表1
INNER JOIN 表2 
  ON 表1.关联列 = 表2.关联列
[WHERE 过滤条件];
```
#### **(2) 隐式语法（逗号分隔 + WHERE）**
```sql
SELECT 列名
FROM 表1, 表2
WHERE 表1.关联列 = 表2.关联列
[AND 其他过滤条件];
```
- **显式语法更推荐**：可读性强，尤其适合多表连接。

---

### **2. 核心特点**
| **特性**       | **说明**                             |
| -------------- | ------------------------------------ |
| **匹配条件**   | 仅返回两表中关联列值相等的行（交集） |
| **未匹配数据** | 不显示任何一表中未匹配的行           |
| **性能**       | 合理使用索引时效率高，大表连接需优化 |
| **替代写法**   | 隐式语法（不推荐）                   |
| **多表连接**   | 可链式连接多个表，逐层指定关联条件   |

---

### **3. 应用场景**
#### **(1) 获取关联数据**
**示例**：查询订单信息及对应的客户姓名  
```sql
SELECT orders.order_id, customers.customer_name
FROM orders
INNER JOIN customers 
  ON orders.customer_id = customers.customer_id;
```

#### **(2) 多表联合查询**
**示例**：查询员工姓名、部门及所在城市  
```sql
SELECT e.name, d.department_name, o.city
FROM employees e
INNER JOIN departments d 
  ON e.dept_id = d.dept_id
INNER JOIN offices o 
  ON d.office_id = o.office_id;
```

#### **(3) 自连接（同一表内关联）**
**示例**：查询员工及其经理姓名  
```sql
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
INNER JOIN employees e2 
  ON e1.manager_id = e2.employee_id;
```

---

### **4. 关键注意事项**
#### **(1) 别名使用**
- 为表设置别名（如 `employees e`），简化列引用，尤其在自连接或多表时。  
- **示例**：  
  ```sql
  SELECT p.product_name, c.category_name
  FROM products p
  INNER JOIN categories c 
    ON p.category_id = c.category_id;
  ```

#### **(2) 明确指定列名**
- 多表存在同名列时（如 `id`），需用 `表名.列名` 或别名避免歧义。  
- **错误示例**：  
  ```sql
  SELECT id, name  -- 歧义：id 属于哪个表？
  FROM orders
  INNER JOIN customers 
    ON orders.customer_id = customers.id;
  ```

#### **(3) 连接条件与过滤条件分离**
- **`ON` 子句**：仅定义表之间的连接条件。  
- **`WHERE` 子句**：对连接后的结果进行过滤。  
- **示例**：筛选2023年的订单  
  ```sql
  SELECT orders.order_id, customers.name
  FROM orders
  INNER JOIN customers 
    ON orders.customer_id = customers.id
  WHERE orders.order_date >= '2023-01-01';
  ```

---

### **5. 性能优化建议**
1. **索引优化**：  
   - 在关联列（如 `customer_id`）上创建索引，加速连接匹配。  
   - 复合索引根据查询需求设计。  

2. **减少连接字段的数据类型差异**：  
   - 确保关联列数据类型一致，避免隐式转换导致性能下降。  

3. **限制结果集大小**：  
   - 使用 `WHERE` 提前过滤无用数据，减少连接计算量。  

---

### **6. 内连接 vs 其他连接**
| **连接类型**   | **返回结果**                                                 |
| -------------- | ------------------------------------------------------------ |
| **INNER JOIN** | 仅返回匹配的行                                               |
| **LEFT JOIN**  | 返回左表所有行 + 右表匹配的行（右表无匹配则填充 NULL）       |
| **RIGHT JOIN** | 返回右表所有行 + 左表匹配的行（左表无匹配则填充 NULL）       |
| **FULL JOIN**  | 返回左右表所有行（无匹配则另一侧填充 NULL，但 MySQL 不支持，需用 UNION 模拟） |

---

### **7. 典型示例**
#### **(1) 统计每个部门的员工数量**
```sql
SELECT d.department_name, COUNT(e.employee_id) AS employee_count
FROM departments d
INNER JOIN employees e 
  ON d.dept_id = e.dept_id
GROUP BY d.department_name;
```

#### **(2) 查询库存不足的商品及其供应商**
```sql
SELECT p.product_name, s.supplier_name, p.stock
FROM products p
INNER JOIN suppliers s 
  ON p.supplier_id = s.supplier_id
WHERE p.stock < 10;
```

---

### **总结**
- **内连接核心**：精准匹配关联数据，排除不相关行。  
- **最佳实践**：显式语法 + 别名 + 索引优化。  
- **适用场景**：需要联合多表关联数据的精确查询（如订单-客户、员工-部门）。



##  SQL语法的声明顺序和执行顺序

---

### **SQL 的声明顺序（书写顺序）与执行顺序**

在 SQL 中，**声明顺序**是开发者编写语句时的顺序，而**执行顺序**是数据库实际处理查询的逻辑顺序。理解两者的差异，对编写高效、准确的 SQL 至关重要。

---

### **1. 声明顺序（书写顺序）**
开发者按以下顺序编写 SQL 语句：
```sql
SELECT   -- 选择列
FROM     -- 数据来源
WHERE    -- 行级过滤
GROUP BY -- 分组
HAVING   -- 组级过滤
ORDER BY -- 排序
LIMIT    -- 限制返回行数
```
示例：
```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE salary > 3000
GROUP BY department
HAVING AVG(salary) > 5000
ORDER BY avg_salary DESC
LIMIT 5;
```

---

### **2. 执行顺序**
数据库引擎按以下顺序处理 SQL 语句：
```sql
1. FROM      -- 确定数据来源（表或子查询）
2. JOIN      -- 连接表（在 FROM 阶段处理）
3. WHERE     -- 过滤行（在 JOIN 后执行）
4. GROUP BY  -- 分组数据
5. HAVING    -- 过滤分组后的结果
6. SELECT    -- 选择列并计算表达式（包括聚合函数）
7. DISTINCT  -- 去重（如果有）
8. ORDER BY  -- 排序结果
9. LIMIT     -- 限制返回行数
```

---

### **3. 关键差异与示例分析**

#### **(1) `SELECT` 中的别名不能在 `WHERE` 或 `GROUP BY` 中使用**
- **执行顺序**：`SELECT` 在 `WHERE` 和 `GROUP BY` 之后执行。  
- **示例**：
  ```sql
  -- 错误：WHERE 中无法使用 SELECT 定义的别名
  SELECT salary * 1.1 AS new_salary
  FROM employees
  WHERE new_salary > 5000; -- 报错！
  
  -- 正确：WHERE 中需使用原始表达式
  SELECT salary * 1.1 AS new_salary
  FROM employees
  WHERE salary * 1.1 > 5000;
  ```

#### **(2) `HAVING` 可访问 `SELECT` 中的聚合结果**
- **执行顺序**：`HAVING` 在 `GROUP BY` 之后、`SELECT` 之前执行。  
- **示例**：
  ```sql
  SELECT department, AVG(salary) AS avg_salary
  FROM employees
  GROUP BY department
  HAVING AVG(salary) > 5000; -- 正确：HAVING 直接使用聚合函数
  ```

#### **(3) `ORDER BY` 可访问 `SELECT` 别名**
- **执行顺序**：`ORDER BY` 在 `SELECT` 之后执行。  
- **示例**：
  ```sql
  SELECT department, AVG(salary) AS avg_salary
  FROM employees
  GROUP BY department
  ORDER BY avg_salary DESC; -- 正确：使用 SELECT 别名
  ```

---

### **4. 执行流程图解**
```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT
```

---

### **5. 典型场景与优化技巧**

#### **(1) 性能优化：尽早过滤数据**
- **优化前**（低效）：
  ```sql
  SELECT *
  FROM large_table
  GROUP BY category
  HAVING COUNT(*) > 100;
  ```
- **优化后**（高效）：
  
  ```sql
  SELECT category
  FROM large_table
  WHERE id IN (SELECT id FROM some_condition) -- 先过滤再分组
  GROUP BY category
  HAVING COUNT(*) > 100;
  ```

#### **(2) 避免冗余计算**
- **低效写法**：
  ```sql
  SELECT department, AVG(salary) AS avg_salary
  FROM employees
  GROUP BY department
  HAVING AVG(salary) > 5000  -- 重复计算 AVG(salary)
  ORDER BY AVG(salary) DESC;
  ```
- **高效写法**：
  ```sql
  SELECT department, AVG(salary) AS avg_salary
  FROM employees
  GROUP BY department
  HAVING avg_salary > 5000    -- 部分数据库支持（如 MySQL）
  ORDER BY avg_salary DESC;
  ```

---

### **6. 总结**
- **声明顺序**：开发者编写 SQL 的顺序，符合人类逻辑。  
- **执行顺序**：数据库处理查询的实际顺序，决定查询性能和结果正确性。  
- **核心原则**：  
  - 在 `WHERE` 中过滤数据，减少后续处理的数据量。  
  - 在 `HAVING` 中过滤分组结果，避免对不必要的数据进行聚合。  
  - 合理使用别名和表达式，避免重复计算。

## distinct关键字

select distinct 字段

表示去重

## 子查询

---

#### **一、子查询的分类**  
1. **按位置分类**  
   - **WHERE 子句子查询**：作为过滤条件（最常见）。  
     
     ```sql
     SELECT name FROM employees 
     WHERE salary > (SELECT AVG(salary) FROM employees);
     ```
   - **FROM 子句子查询**：作为临时表（需别名）。  
     
     ```sql
     SELECT dept, avg_salary 
     FROM (SELECT department AS dept, AVG(salary) AS avg_salary 
           FROM employees GROUP BY department) AS tmp;
     ```
   - **SELECT 子句子查询**：作为返回列的一部分（标量子查询）。  
     ```sql
     SELECT name, (SELECT COUNT(*) FROM orders WHERE orders.emp_id = employees.id) 
     AS order_count FROM employees;
     ```
   
2. **按返回结果分类**  
   - **标量子查询**：返回单值（一行一列）。  
   - **列子查询**：返回一列多行（常与 `IN` 或 `ANY` 配合）。  
   - **行子查询**：返回一行多列（需与行条件匹配）。  
   - **表子查询**：返回多行多列（通常用于 `FROM` 或 `JOIN`）。

3. **按依赖性分类**  
   - **非相关子查询**：独立于外部查询，仅执行一次。  
     ```sql
     SELECT name FROM products 
     WHERE price > (SELECT AVG(price) FROM products);
     ```
   - **相关子查询**：依赖外部查询的值，逐行执行（可能影响性能）。  
     ```sql
     SELECT name FROM employees e 
     WHERE salary > (SELECT AVG(salary) FROM employees WHERE department = e.department);
     ```

---

#### **二、子查询的典型场景**  
1. **动态条件过滤**  
   - 使用 `IN`、`NOT IN`、`ANY`、`ALL` 等操作符：  
     ```sql
     SELECT title FROM movies 
     WHERE id IN (SELECT movie_id FROM boxoffice WHERE rating > 8);
     ```

2. **聚合值的动态比较**  
   - 比较单行与聚合结果：  
     ```sql
     SELECT name FROM students 
     WHERE score > (SELECT AVG(score) FROM students);
     ```

3. **替代连接查询（需权衡性能）**  
   - 使用 `EXISTS` 或 `NOT EXISTS`：  
     ```sql
     SELECT name FROM departments d 
     WHERE EXISTS (SELECT 1 FROM employees WHERE department_id = d.id);
     ```

---

#### **三、子查询的性能与优化**  
1. **性能陷阱**  
   - **相关子查询**：逐行执行，大表场景性能差。  
   - **多层嵌套**：可读性低，优化器难以处理。

2. **优化策略**  
   - **用 `JOIN` 替代相关子查询**：  
     
     ```sql
     -- 原查询（相关子查询）
     SELECT e.name FROM employees e 
     WHERE salary > (SELECT AVG(salary) FROM employees WHERE department = e.department);
     
     -- 优化为 JOIN
     SELECT e.name 
     FROM employees e 
     JOIN (SELECT department, AVG(salary) AS avg_salary 
           FROM employees GROUP BY department) AS dept_avg 
     ON e.department = dept_avg.department 
     WHERE e.salary > dept_avg.avg_salary;
     ```
   - **使用临时表或 CTE**：  
     
     ```sql
     WITH dept_avg AS (
         SELECT department, AVG(salary) AS avg_salary 
         FROM employees GROUP BY department
     )
     SELECT e.name FROM employees e 
     JOIN dept_avg ON e.department = dept_avg.department 
     WHERE e.salary > dept_avg.avg_salary;
     ```

---

#### **四、注意事项**  
1. **NULL 值处理**  
   - `IN` 子查询包含 `NULL` 时可能返回意外结果，使用 `EXISTS` 更安全。  
   - 示例：  
     ```sql
     -- 错误：若子查询返回 NULL，结果可能不符合预期
     SELECT name FROM table1 WHERE id NOT IN (SELECT id FROM table2);
     
     -- 正确：使用 NOT EXISTS
     SELECT name FROM table1 t1 
     WHERE NOT EXISTS (SELECT 1 FROM table2 t2 WHERE t1.id = t2.id);
     ```

2. **结果类型匹配**  
   - 确保子查询返回的列数与数据类型与外部条件匹配。  
   - 错误示例：  
     ```sql
     -- 错误：子查询返回多列
     SELECT name FROM employees 
     WHERE (department, salary) IN (SELECT department, AVG(salary) FROM employees);
     ```

3. **避免过度嵌套**  
   - 多层子查询难以维护，优先使用 CTE（Common Table Expressions）或临时表。

---

#### **五、高级用法示例**  
1. **生成序列或虚拟数据**  
   - 结合 `GENERATE_SERIES`（PostgreSQL）或递归 CTE：  
     ```sql
     -- 生成 1-10 的数字序列
     WITH numbers AS (
         SELECT generate_series(1,10) AS num
     )
     SELECT num FROM numbers;
     ```

2. **动态分桶统计**  
   - 使用子查询定义分桶规则：  
     ```sql
     SELECT 
         CASE 
             WHEN age < (SELECT AVG(age) FROM users) THEN 'Below Average'
             ELSE 'Above Average'
         END AS age_group,
         COUNT(*) AS user_count
     FROM users
     GROUP BY age_group;
     ```

## limit和offset

limit 控制输出几行

offset 控制开始输出的是第几行

## union查询结果合并

---

### **MySQL 中 UNION 操作符的详细介绍**

在 MySQL 中，`UNION` 是一种用于合并两个或多个 `SELECT` 查询结果集的操作符。它能够将多个独立查询的结果合并为一个结果集，并支持去重或保留所有行的选项。以下是 `UNION` 的核心要点及详细说明：

---

#### **1. UNION 的基本语法**
```sql
SELECT column1, column2, ... FROM table1
[WHERE conditions]
UNION [ALL]
SELECT column1, column2, ... FROM table2
[WHERE conditions]
[ORDER BY column1, column2, ...];
```
- **关键规则**：
  - 所有 `SELECT` 语句的列数必须相同。
  - 对应列的数据类型需兼容（例如，`INT` 与 `VARCHAR` 可能隐式转换，但需谨慎）。
  - `ORDER BY` 仅能出现在最后一个 `SELECT` 语句后，作用于最终合并结果。

---

#### **2. UNION 与 UNION ALL 的区别**
| **特性**     | **UNION**                            | **UNION ALL**                  |
| ------------ | ------------------------------------ | ------------------------------ |
| **去重**     | 自动去除重复行                       | 保留所有行，包括重复行         |
| **性能**     | 较慢（需额外去重操作）               | 较快（无需去重）               |
| **适用场景** | 需要唯一结果的场景（如合并分类数据） | 需要保留重复数据（如日志合并） |

**示例**：

```sql
-- 使用 UNION（去重）
SELECT name FROM employees_departmentA
UNION
SELECT name FROM employees_departmentB;

-- 使用 UNION ALL（保留重复）
SELECT name FROM employees_departmentA
UNION ALL
SELECT name FROM employees_departmentB;
```

---

#### **3. UNION 的列匹配规则**
- **列名无关性**：结果集的列名以第一个 `SELECT` 语句的列名为准。
- **数据类型兼容**：对应列需隐式兼容，否则可能引发错误或数据截断。
  ```sql
  -- 示例：混合 INT 和 VARCHAR（隐式转换）
  SELECT id, name FROM users
  UNION
  SELECT '123', 'John' FROM dual; -- id 列将被转换为字符串
  ```

---

#### **4. 排序与过滤**
- **排序**：只能在最后一个 `SELECT` 后使用 `ORDER BY`，对整个结果集排序。
  ```sql
  SELECT name FROM tableA
  UNION
  SELECT name FROM tableB
  ORDER BY name DESC;
  ```
- **过滤**：每个 `SELECT` 可独立使用 `WHERE`、`GROUP BY` 等子句。
  ```sql
  SELECT product, price FROM inventory WHERE price > 100
  UNION
  SELECT product, price FROM archive WHERE price < 50;
  ```

---

#### **5. 高级用法**
##### **(1) 合并不同表的结构化数据**
```sql
-- 合并员工表和客户表的姓名
SELECT name, 'Employee' AS type FROM employees
UNION
SELECT name, 'Customer' AS type FROM customers;
```

##### **(2) 分页统计（如跨表分页）**
```sql
(SELECT id, title FROM news WHERE category = 'Tech' ORDER BY id LIMIT 10)
UNION
(SELECT id, title FROM blogs WHERE category = 'Tech' ORDER BY id LIMIT 10)
ORDER BY id LIMIT 10;
```

##### **(3) 动态生成虚拟数据**
```sql
SELECT 1 AS id, 'Apple' AS fruit
UNION
SELECT 2, 'Banana'
UNION
SELECT 3, 'Cherry';
```

---

#### **6. 性能优化建议**
1. **优先使用 UNION ALL**：若明确不需要去重，避免不必要的性能开销。
2. **限制子查询数据量**：在 `SELECT` 语句中使用 `WHERE` 提前过滤数据。
3. **索引优化**：确保参与 `UNION` 的列有适当索引，尤其是排序字段。

---

#### **7. 常见错误及解决方法**
- **错误 1：列数不匹配**
  ```sql
  -- 错误示例
  SELECT id, name FROM tableA
  UNION
  SELECT id FROM tableB; -- 列数不一致
  ```
  **解决**：确保所有 `SELECT` 的列数相同。

- **错误 2：数据类型不兼容**
  ```sql
  -- 错误示例
  SELECT id FROM tableA  -- id 是 INT
  UNION
  SELECT name FROM tableB; -- name 是 VARCHAR
  ```
  **解决**：显式转换数据类型，例如 `SELECT CAST(id AS CHAR) FROM tableA`。

- **错误 3：错误使用 ORDER BY**
  ```sql
  -- 错误示例
  SELECT name FROM tableA ORDER BY name
  UNION
  SELECT name FROM tableB;
  ```
  **解决**：将 `ORDER BY` 移至最后，且仅对整个结果集排序。

---

#### **8. 实际应用场景**
##### **场景 1：合并多表数据**
```sql
-- 合并不同年份的销售记录
SELECT product, sales_2022 FROM sales_2022
UNION ALL
SELECT product, sales_2023 FROM sales_2023;
```

##### **场景 2：动态生成配置项**
```sql
-- 合并系统默认配置与用户自定义配置
SELECT key, value FROM default_settings
UNION
SELECT key, value FROM user_settings;
```

##### **场景 3：跨数据库查询**
```sql
-- 合并不同数据库的表（需权限）
SELECT * FROM db1.employees
UNION
SELECT * FROM db2.contractors;
```

---

### **总结**
- **UNION** 是合并多查询结果的核心工具，适合数据整合与分析。
- **UNION ALL** 性能更优，但需手动处理重复数据。
- 注意列匹配、数据类型兼容及排序规则，避免常见错误。
- 在复杂查询中灵活运用 `UNION`，可显著提升数据处理效率。
