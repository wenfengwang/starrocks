---
displayed_sidebar: English
---

# SHOW CREATE DATABASE

展示用于创建数据库的 SQL 命令。

## 语法

```sql
SHOW CREATE DATABASE <db_name>
```

## 参数

`db_name`：数据库名称，必填。

## 返回值

- `Database`：数据库名称

- `Create Database`：用于创建数据库的 SQL 命令

## 示例

```sql
mysql > SHOW CREATE DATABASE zj_test;
+----------+---------------------------+
| Database | Create Database           |
+----------+---------------------------+
| zj_test  | CREATE DATABASE `zj_test` |
+----------+---------------------------+
```

## 参考资料

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW DATABASES](SHOW_DATABASES.md)
- [USE](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)