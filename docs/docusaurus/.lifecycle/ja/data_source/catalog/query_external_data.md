---
displayed_sidebar: "Japanese"
---

# 外部データのクエリ

このトピックでは、外部カタログを使用して外部データソースからデータをクエリする方法について説明します。

## 前提条件

外部カタログは外部データソースに基づいて作成されます。外部カタログのサポートされているタイプの情報については、[カタログ](../catalog/catalog_overview.md#catalog)を参照してください。

## 手順

1. StarRocksクラスターに接続します。
   - StarRocksクラスターにMySQLクライアントを使用して接続する場合、接続後にデフォルトで `default_catalog` に移動します。
   - StarRocksクラスターにJDBCを使用して接続する場合、接続時に `default_catalog.db_name` を指定してデフォルトカタログ内の宛先データベースに直接移動できます。

2. (オプション) 次のステートメントを実行してすべてのカタログを表示し、作成した外部カタログを見つけます。このステートメントの出力を確認するには、[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を参照してください。

      ```SQL
      SHOW CATALOGS;
      ```

3. (オプション) 次のステートメントを実行して外部カタログ内のすべてのデータベースを表示します。このステートメントの出力を確認するには、[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を参照してください。

      ```SQL
      SHOW DATABASES FROM catalog_name;
      ```

4. (オプション) 次のステートメントを実行して外部カタログ内の宛先データベースに移動します。

      ```SQL
      USE catalog_name.db_name;
      ```

5. 外部データをクエリします。SELECTステートメントのその他の使用法については、[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を参照してください。

      ```SQL
      SELECT * FROM table_name;
      ```

      前述の手順で外部カタログとデータベースを指定しない場合、SELECTクエリで直接指定することができます。

      ```SQL
      SELECT * FROM catalog_name.db_name.table_name;
      ```

## 例

すでに`hive1`という名前のHiveカタログを作成し、Apache Hive™クラスターの`hive_db.hive_table`からデータをクエリする場合、次の操作のいずれかを実行できます。

```SQL
USE hive1.hive_db;
SELECT * FROM hive_table limit 1;
```

または

```SQL
SELECT * FROM hive1.hive_db.hive_table limit 1;
```

## 参照

StarRocksクラスターからデータをクエリするためには、[デフォルトカタログ](../catalog/default_catalog.md)を参照してください。