---
displayed_sidebar: "Japanese"
---

# デフォルトカタログ

このトピックでは、デフォルトカタログとは何か、およびデフォルトカタログを使用してStarRocksの内部データをクエリする方法について説明します。

StarRocks 2.3以降では、内部データを管理するための内部カタログが提供されています。各StarRocksクラスタには、`default_catalog`という名前の内部カタログが1つだけあります。現在、内部カタログの名前を変更することや新しい内部カタログを作成することはできません。

## 内部データのクエリ

1. StarRocksクラスタに接続します。
   - StarRocksクラスタにMySQLクライアントを使用して接続する場合、接続後にデフォルトで`default_catalog`に移動します。
   - StarRocksクラスタにJDBCを使用して接続する場合、接続時に`default_catalog.db_name`を指定することで、デフォルトカタログ内の目的のデータベースに直接移動することができます。

2. (任意) [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用してデータベースを表示します:

      ```SQL
      SHOW DATABASES;
      ```

      または

      ```SQL
      SHOW DATABASES FROM <catalog_name>;
      ```

3. (任意) [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して現在のセッションで目的のカタログに切り替えます:

    ```SQL
    SET CATALOG <catalog_name>;
    ```

    その後、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して現在のセッションでアクティブなデータベースを指定します:

    ```SQL
    USE <db_name>;
    ```

    または、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、目的のカタログ内のアクティブなデータベースに直接移動することもできます:

    ```SQL
    USE <catalog_name>.<db_name>;
    ```

4. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して内部データをクエリします:

      ```SQL
      SELECT * FROM <table_name>;
      ```

      前の手順でアクティブなデータベースを指定しなかった場合、セレクトクエリ内で直接指定することもできます:

      ```SQL
      SELECT * FROM <db_name>.<table_name>;
      ```

      または

      ```SQL
      SELECT * FROM default_catalog.<db_name>.<table_name>;
      ```

## 例

`olap_db.olap_table`のデータをクエリするには、次のいずれかの操作を実行できます:

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

## 参考

外部データソースからデータをクエリする方法については、[外部データのクエリ](../catalog/query_external_data.md)を参照してください。
