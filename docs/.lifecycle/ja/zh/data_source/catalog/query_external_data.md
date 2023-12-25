---
displayed_sidebar: Chinese
---

# 外部データのクエリ

本文では、External Catalog を使用して外部データをクエリする方法について説明します。External Catalog は、外部テーブルを作成することなく、様々な外部ソースに保存されているデータに簡単にアクセスしてクエリすることができます。

## 前提条件

データソースに基づいて、異なるタイプの External Catalog が作成されています。現在サポートされている External Catalog のタイプについては、[Catalog](../catalog/catalog_overview.md#catalog) を参照してください。

## 操作手順

1. StarRocks に接続します。
   - MySQL クライアントから StarRocks に接続する場合、接続後にデフォルトで `default_catalog` に入ります。
   - JDBC を使用して StarRocks に接続する場合、接続時に `default_catalog.db_name` を指定してデータベースを選択できます。

2. （オプション）以下のステートメントを実行して、現在の StarRocks クラスター内のすべての Catalog を表示し、指定された External Catalog を見つけます。戻り値の説明については、[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) を参照してください。

    ```SQL
    SHOW CATALOGS;
    ```

3. （オプション）以下のステートメントを実行して、指定された external catalog 内のデータベースを表示します。パラメータと戻り値の説明については、[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を参照してください。

    ```SQL
    SHOW DATABASES FROM catalog_name;
    ```

4. （オプション）以下のステートメントを実行して、現在のセッションを指定された external catalog のデータベースに切り替えます。パラメータの説明と例については、[USE](../../sql-reference/sql-statements/data-definition/USE.md) を参照してください。

    ```SQL
    USE catalog_name.db_name;
    ```

5. 外部データをクエリします。SELECT の使用方法については、[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を参照してください。

    ```SQL
    SELECT * FROM table_name;
    ```

    上記の手順で external catalog やデータベースを指定していない場合は、クエリ文中で直接指定することができます。例：

    ```SQL
    SELECT * FROM catalog_name.db_name.table_name;
    ```

## 例

`hive1` という名前の Hive catalog を作成します。`hive1` を通じて Apache Hive™ クラスター内の `hive_db.hive_table` のデータをクエリするには、以下の操作を行います：

```SQL
USE hive1.hive_db;
SELECT * FROM hive_table LIMIT 1;
```

または

```SQL
SELECT * FROM hive1.hive_db.hive_table LIMIT 1;  
```

## その他の操作

StarRocks の内部データをクエリする場合は、[内部データのクエリ](../catalog/default_catalog.md#内部データのクエリ) を参照してください。
