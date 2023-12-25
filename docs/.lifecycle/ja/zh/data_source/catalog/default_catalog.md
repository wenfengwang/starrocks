---
displayed_sidebar: Chinese
---

# デフォルトカタログ

この文書では、デフォルトカタログとは何か、およびデフォルトカタログを使用して StarRocks の内部データをクエリする方法について説明します。

StarRocks 2.3 以降のバージョンでは、Internal Catalog（内部データカタログ）が提供されており、StarRocks の[内部データ](../catalog/catalog_overview.md#基本概念)を管理するために使用されます。各 StarRocks クラスターには1つの Internal Catalog のみが存在し、その名前は `default_catalog` です。StarRocks では、Internal Catalog の名前を変更することはできませんし、新しい Internal Catalog を作成することもできません。

## 内部データのクエリ

1. StarRocks に接続します。
   - MySQL クライアントから StarRocks に接続する場合、接続後にデフォルトで `default_catalog` に入ります。
   - JDBC を使用して StarRocks に接続する場合、接続時に `default_catalog.db_name` を指定して接続するデータベースを指定できます。
2. （オプション）[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を使用してデータベースを表示します：

   ```SQL
   SHOW DATABASES;
   ```

   または

   ```SQL
   SHOW DATABASES FROM default_catalog;
   ```

3. （オプション）[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) を使用して、現在のセッションで有効なカタログを切り替えます：

   ```SQL
   SET CATALOG <catalog_name>;
   ```

   次に [USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用して、現在のセッションで有効なデータベースを指定します：

   ```SQL
   USE <db_name>;
   ```

   または、[USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用して、セッションを目的のカタログの指定されたデータベースに直接切り替えることもできます：

   ```SQL
   USE <catalog_name>.<db_name>;
   ```

4. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して内部データをクエリします：

   ```SQL
   SELECT * FROM <table_name>;
   ```

   上記の手順でデータベースを指定していない場合は、クエリ文で直接指定することができます。

   ```SQL
   SELECT * FROM <db_name>.<table_name>;
   ```

   または

   ```SQL
   SELECT * FROM default_catalog.<db_name>.<table_name>;
   ```

## 例

`olap_db.olap_table` のデータをクエリする場合は、以下のように操作します：

 ```SQL
USE olap_db;
SELECT * FROM olap_table limit 1;
```

または

```SQL
SELECT * FROM olap_db.olap_table limit 1;   
```

または

```SQL
SELECT * FROM default_catalog.olap_db.olap_table limit 1;
```

## その他の操作

外部データをクエリする場合は、[外部データのクエリ](./query_external_data.md)を参照してください。
