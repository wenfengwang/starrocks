---
displayed_sidebar: English
---

# 显示创建数据库

显示创建数据库时所使用的 SQL 命令。

## 语法

```sql
SHOW CREATE DATABASE <db_name>
```

## 参数

`db_name`：数据库名称，必填。

## 返回

- `Database`：数据库名称

- `Create Database`：用于创建数据库的 SQL 命令

## 例子

```sql
mysql > show create database zj_test;
+----------+---------------------------+
| Database | Create Database           |
+----------+---------------------------+
| zj_test  | CREATE DATABASE `zj_test` |
+----------+---------------------------+
```

## 引用

- [创建数据库](../data-definition/CREATE_DATABASE.md)
- [显示数据库](SHOW_DATABASES.md)
- [使用](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [删除数据库](../data-definition/DROP_DATABASE.md)
