---
displayed_sidebar: English
---

# 显示创建数据库的命令

展示用来创建数据库的SQL命令。

## 语法

```sql
SHOW CREATE DATABASE <db_name>
```

## 参数

db_name：数据库名，必须提供。

## 返回值

- 数据库：数据库名

- 创建数据库：创建该数据库的SQL命令

## 示例

```sql
mysql > show create database zj_test;
+----------+---------------------------+
| Database | Create Database           |
+----------+---------------------------+
| zj_test  | CREATE DATABASE `zj_test` |
+----------+---------------------------+
```

## 参考资料

- [创建数据库](../data-definition/CREATE_DATABASE.md)
- [SHOW DATABASES](SHOW_DATABASES.md)
- [使用](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [删除数据库](../data-definition/DROP_DATABASE.md)
