# 存储引擎概述
MySQL存储引擎决定了数据在磁盘上的存储方式和访问方式。不同的存储引擎实现了不同的存储和检索算法，因此它们在处理和管理数据的方式上存在差异。

MySQL常见的存储引擎包括InnoDB、MyISAM、Memory、Archive等。每个存储引擎都有自己的特点和适用场景。

例如，

- InnoDB引擎支持事务和行级锁定，适用于需要高并发读写的应用；
- MyISAM引擎不支持事务，但适用于读操作较多的应用；
- Memory引擎数据全部存储在内存中，适用于对读写速度要求很高的应用等等。

选择适合的存储引擎可以提高MySQL的性能和效率，并且根据应用需求来合理选择存储引擎可以提供更好的数据管理和查询功能。

![](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&from=url&id=M5q7i&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=shadow&title=)
# MySQL支持哪些存储引擎
使用`show engines \G;`命令可以查看所有的存储引擎：
```java
*************************** 1. row ***************************
      Engine: MEMORY
     Support: YES
     Comment: Hash based, stored in memory, useful for temporary tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 2. row ***************************
      Engine: MRG_MYISAM
     Support: YES
     Comment: Collection of identical MyISAM tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 3. row ***************************
      Engine: CSV
     Support: YES
     Comment: CSV storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 4. row ***************************
      Engine: FEDERATED
     Support: NO
     Comment: Federated MySQL storage engine
Transactions: NULL
          XA: NULL
  Savepoints: NULL
*************************** 5. row ***************************
      Engine: PERFORMANCE_SCHEMA
     Support: YES
     Comment: Performance Schema
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 6. row ***************************
      Engine: MyISAM
     Support: YES
     Comment: MyISAM storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 7. row ***************************
      Engine: InnoDB
     Support: DEFAULT
     Comment: Supports transactions, row-level locking, and foreign keys
Transactions: YES
          XA: YES
  Savepoints: YES
*************************** 8. row ***************************
      Engine: ndbinfo
     Support: NO
     Comment: MySQL Cluster system information storage engine
Transactions: NULL
          XA: NULL
  Savepoints: NULL
*************************** 9. row ***************************
      Engine: BLACKHOLE
     Support: YES
     Comment: /dev/null storage engine (anything you write to it disappears)
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 10. row ***************************
      Engine: ARCHIVE
     Support: YES
     Comment: Archive storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 11. row ***************************
      Engine: ndbcluster
     Support: NO
     Comment: Clustered, fault-tolerant tables
Transactions: NULL
          XA: NULL
  Savepoints: NULL
```
`Support`是`Yes`的表示支持该存储引擎。当前MySQL的版本是`8.0.33`
MySQL默认的存储引擎是：`InnoDB`

![](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&from=url&id=SagiX&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=shadow&title=)
# 指定和修改存储引擎
## 指定存储引擎
在MySQL中，你可以在创建表时指定使用的存储引擎。通过在CREATE TABLE语句中使用ENGINE关键字，你可以指定要使用的存储引擎。

以下是指定存储引擎的示例：

```sql
CREATE TABLE my_table (column1 INT, column2 VARCHAR(50)) ENGINE = InnoDB;
```

在这个例子中，我们创建了一个名为my_table的表，并指定了使用InnoDB存储引擎。

如果你不显式指定存储引擎，MySQL将使用默认的存储引擎。默认情况下，MySQL 8的默认存储引擎是InnoDB。

![](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&from=url&id=GDROH&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=shadow&title=)
## 修改存储引擎
在MySQL中，你可以通过ALTER TABLE语句修改表的存储引擎。下面是修改存储引擎的示例：

```sql
ALTER TABLE my_table ENGINE = MyISAM;
```

在这个例子中，我们使用ALTER TABLE语句将my_table表的存储引擎修改为MyISAM。

请注意，在修改存储引擎之前，你需要考虑以下几点：

1.  修改存储引擎可能需要执行复制表的操作，因此可能会造成数据的丢失或不可用。确保在执行修改之前备份你的数据。 
2.  不是所有的存储引擎都支持相同的功能。要确保你选择的新存储引擎支持你应用程序所需的功能。 
3.  修改表的存储引擎可能会影响到现有的应用程序和查询。确保在修改之前评估和测试所有的影响。 
4.  ALTER TABLE语句可能需要适当的权限才能执行。确保你拥有足够的权限来执行修改存储引擎的操作。 

总而言之，修改存储引擎需要谨慎进行，且需要考虑到可能的影响和风险。建议在进行修改之前进行适当的测试和备份。

![](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&from=url&id=zd3z2&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=shadow&title=)
# 常用的存储引擎及适用场景
在实际开发中，以下存储引擎是比较常用的：

1.  InnoDB：
   1. MySQL默认的事务型存储引擎 
   2. 支持ACID事务
   3. 具有较好的并发性能和数据完整性
   4. 支持行级锁定。
   5. 适用于大多数应用场景，尤其是需要事务支持的应用。 
2.  MyISAM：
   1. 是MySQL早期版本中常用的存储引擎
   2. 支持全文索引和表级锁定
   3. 不支持事务
   4. 由于其简单性和高性能，在某些特定的应用场景中会得到广泛应用，如**读密集**的应用。 
3.  MEMORY：
   1. 称为HEAP，是将表存储在内存中的存储引擎
   2. 具有非常高的读写性能，但数据会在服务器重启时丢失。
   3. 适用于需要快速读写的临时数据集、缓存和临时表等场景。 
4.  CSV：
   1. 将数据以纯文本格式存储的存储引擎
   2. 适用于需要处理和导入/导出CSV格式数据的场景。 
5.  ARCHIVE：
   1. 将数据高效地进行压缩和存储的存储引擎
   2. 适用于需要长期存储大量历史数据且不经常查询的场景。 
