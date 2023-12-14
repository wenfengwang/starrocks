---
displayed_sidebar: "Chinese"
---

# 显示创建数据库

显示用于创建数据库的SQL命令。

## 语法

```sql
SHOW CREATE DATABASE <db_name>
```

## 参数

`db_name`: 数据库名称，必填。

## 返回

- `数据库`: 数据库名称

- `创建数据库`: 用于创建数据库的SQL命令

## 示例

```sql
mysql > show create database zj_test;
+----------+---------------------------+
| 数据库   | 创建数据库                |
+----------+---------------------------+
| zj_test  | CREATE DATABASE `zj_test` |
+----------+---------------------------+
```

## 参考

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW DATABASES](SHOW_DATABASES.md)
- [USE](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)