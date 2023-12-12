```yaml
---
displayed_sidebar: "Japanese"
---
# 外部データのクエリ

このトピックでは、外部カタログを使用して外部データソースからデータをクエリする方法について説明します。

## 前提条件

外部カタログは、外部データソースに基づいて作成されます。外部カタログのサポートされているタイプについての情報は、[カタログ](../catalog/catalog_overview.md#catalog)を参照してください。

## 手順

1. StarRocks クラスタに接続します。
   - StarRocks クラスタに MySQL クライアントを使用して接続する場合、接続後にデフォルトで `default_catalog` に移動します。
   - StarRocks クラスタに JDBC を使用して接続する場合、接続時に `default_catalog.db_name` を指定することで、デフォルトカタログ内の宛先データベースに直接移動できます。

2. (任意) 次のステートメントを実行して、すべてのカタログを表示し、作成した外部カタログを見つけます。このステートメントの出力を確認するには、[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を参照してください。

    ```SQL
    SHOW CATALOGS;
    ```

3. (任意) 次のステートメントを実行して、外部カタログ内のすべてのデータベースを表示します。このステートメントの出力を確認するには、[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を参照してください。

    ```SQL
    SHOW DATABASES FROM catalog_name;
    ```

4. (任意) 次のステートメントを実行して、外部カタログ内の宛先データベースに移動します。

    ```SQL
    USE catalog_name.db_name;
    ```

5. 外部データをクエリします。SELECT ステートメントのその他の使用法については、[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を参照してください。

    ```SQL
    SELECT * FROM table_name;
    ```

    前述の手順で外部カタログとデータベースを指定しない場合、SELECT クエリで直接指定できます。

    ```SQL
    SELECT * FROM catalog_name.db_name.table_name;
    ```

## 例

既に `hive1` という名前の Hive カタログを作成し、Apache Hive™ クラスタの `hive_db.hive_table` からデータをクエリしたい場合、次の操作のいずれかを実行できます。

```SQL
USE hive1.hive_db;
SELECT * FROM hive_table limit 1;
```

または

```SQL
SELECT * FROM hive1.hive_db.hive_table limit 1;
```

## 参照

StarRocks クラスタからデータをクエリする場合は、[デフォルトカタログ](../catalog/default_catalog.md)を参照してください。
```