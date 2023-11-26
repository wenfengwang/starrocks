---
displayed_sidebar: "Japanese"
---

# 外部データのクエリ

このトピックでは、外部カタログを使用して外部データソースからデータをクエリする方法について説明します。

## 前提条件

外部カタログは外部データソースに基づいて作成されます。サポートされている外部カタログのタイプについては、[カタログ](../catalog/catalog_overview.md#catalog)を参照してください。

## 手順

1. StarRocksクラスタに接続します。
   - StarRocksクラスタにMySQLクライアントを使用して接続する場合、接続後にデフォルトで`default_catalog`に移動します。
   - StarRocksクラスタにJDBCを使用して接続する場合、接続時に`default_catalog.db_name`を指定することで、デフォルトカタログ内の宛先データベースに直接移動できます。

2. (オプション) 以下のステートメントを実行して、すべてのカタログを表示し、作成した外部カタログを見つけます。このステートメントの出力を確認するには、[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を参照してください。

      ```SQL
      SHOW CATALOGS;
      ```

3. (オプション) 以下のステートメントを実行して、外部カタログ内のすべてのデータベースを表示します。このステートメントの出力を確認するには、[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を参照してください。

      ```SQL
      SHOW DATABASES FROM catalog_name;
      ```

4. (オプション) 以下のステートメントを実行して、外部カタログ内の宛先データベースに移動します。

      ```SQL
      USE catalog_name.db_name;
      ```

5. 外部データをクエリします。SELECTステートメントのさまざまな使い方については、[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を参照してください。

      ```SQL
      SELECT * FROM table_name;
      ```

      前の手順で外部カタログとデータベースを指定しなかった場合、セレクトクエリで直接指定することができます。

      ```SQL
      SELECT * FROM catalog_name.db_name.table_name;
      ```

## 例

すでに`hive1`というHiveカタログを作成し、Apache Hive™クラスタの`hive_db.hive_table`からデータをクエリしたい場合、次の操作のいずれかを実行できます。

```SQL
USE hive1.hive_db;
SELECT * FROM hive_table limit 1;
```

または

```SQL
SELECT * FROM hive1.hive_db.hive_table limit 1;
```

## 参考

StarRocksクラスタからデータをクエリする方法については、[デフォルトカタログ](../catalog/default_catalog.md)を参照してください。
