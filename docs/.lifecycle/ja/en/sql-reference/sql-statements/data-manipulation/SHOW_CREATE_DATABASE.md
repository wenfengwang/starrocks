---
displayed_sidebar: English
---

# SHOW CREATE DATABASEの使用

データベースを作成するために使用されたSQLコマンドを表示します。

## 構文

```sql
SHOW CREATE DATABASE <db_name>
```

## パラメータ

`db_name`: データベース名、必須です。

## 戻り値

- `Database`: データベース名

- `Create Database`: データベースを作成するために使用されたSQLコマンド

## 例

```sql
mysql > SHOW CREATE DATABASE zj_test;
+----------+---------------------------+
| Database | Create Database           |
+----------+---------------------------+
| zj_test  | CREATE DATABASE `zj_test` |
+----------+---------------------------+
```

## 参照

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW DATABASES](SHOW_DATABASES.md)
- [USE](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)
