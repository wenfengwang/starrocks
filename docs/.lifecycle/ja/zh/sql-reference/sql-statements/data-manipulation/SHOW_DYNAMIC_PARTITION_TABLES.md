---
displayed_sidebar: Chinese
---

# 動的パーティションテーブルの表示

## 機能

このステートメントは、現在のデータベース db において動的パーティション属性が設定されているすべてのパーティションテーブルの状態を表示するために使用されます。

## 文法

```sql
SHOW DYNAMIC PARTITION TABLES [FROM <db_name>]
```

## 例

1. データベース database に設定されている動的パーティション属性を持つすべてのパーティションテーブルの状態を表示します。

```sql
SHOW DYNAMIC PARTITION TABLES FROM database;
```
