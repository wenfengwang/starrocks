---
displayed_sidebar: Chinese
---

# SHOW CREATE DATABASE

指定されたデータベースの作成文を表示します。

## 構文

```sql
SHOW CREATE DATABASE <db_name>
```

## パラメータ説明

`db_name`：データベースの名前で、必須です。

## 戻り値の説明

2つのフィールドを返します：

- Database：データベースの名前

- Create Database：データベースの作成文

## 例

```sql
mysql > show create database zj_test;
+----------+---------------------------+
| Database | Create Database           |
+----------+---------------------------+
| zj_test  | CREATE DATABASE `zj_test` |
+----------+---------------------------+
```

## 関連リファレンス

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW DATABASES](SHOW_DATABASES.md)
- [USE](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)
