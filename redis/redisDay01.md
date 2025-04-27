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