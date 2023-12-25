---
displayed_sidebar: English
---

# 外部データのクエリ

このトピックでは、外部カタログを使用して外部データソースからデータをクエリする方法について説明します。

## 前提条件

外部カタログは、外部データソースに基づいて作成されます。サポートされている外部カタログの種類については、[Catalog](../catalog/catalog_overview.md#catalog)を参照してください。

## 手順

1. StarRocksクラスタに接続します。
   - MySQLクライアントを使用してStarRocksクラスタに接続する場合、接続後にデフォルトで`default_catalog`に移動します。
   - JDBCを使用してStarRocksクラスタに接続する場合、`default_catalog.db_name`を指定することで、デフォルトカタログ内の目的のデータベースに直接移動できます。

2. （オプション）次のステートメントを実行して、すべてのカタログを表示し、作成した外部カタログを見つけます。このステートメントの出力を確認するには、[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を参照してください。

      ```SQL
      SHOW CATALOGS;
      ```

3. （オプション）次のステートメントを実行して、外部カタログ内のすべてのデータベースを表示します。このステートメントの出力を確認するには、[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を参照してください。

      ```SQL
      SHOW DATABASES FROM catalog_name;
      ```

4. （オプション）次のステートメントを実行して、外部カタログ内の目的のデータベースに移動します。

      ```SQL
      USE catalog_name.db_name;
      ```

5. 外部データをクエリします。SELECTステートメントのその他の使用方法については、[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を参照してください。

      ```SQL
      SELECT * FROM table_name;
      ```

      前の手順で外部カタログとデータベースを指定していない場合、SELECTクエリで直接指定することができます。

      ```SQL
      SELECT * FROM catalog_name.db_name.table_name;
      ```

## 例

`hive1`という名前のHiveカタログをすでに作成しており、Apache Hive™クラスターの`hive_db.hive_table`からデータをクエリする場合は、次のいずれかの操作を実行できます。

```SQL
USE hive1.hive_db;
SELECT * FROM hive_table LIMIT 1;
```

または

```SQL
SELECT * FROM hive1.hive_db.hive_table LIMIT 1;
```

## 参照

StarRocksクラスターからデータをクエリするには、[Default catalog](../catalog/default_catalog.md)を参照してください。
