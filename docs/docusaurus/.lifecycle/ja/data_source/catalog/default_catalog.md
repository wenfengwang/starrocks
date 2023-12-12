---
displayed_sidebar: "Japanese"
---

# デフォルトのカタログ

このトピックでは、デフォルトのカタログと、デフォルトのカタログを使用してStarRocksの内部データをクエリする方法について説明します。

StarRocks 2.3以降では、StarRocksの内部データを管理するための内部カタログが提供されます。各StarRocksクラスタには、`default_catalog`という名前の1つの内部カタログしかありません。現時点では、内部カタログの名前を変更したり、新しい内部カタログを作成したりすることはできません。

## 内部データのクエリ

1. StarRocksクラスタに接続します。
   - StarRocksクラスタにMySQLクライアントを使用して接続する場合、接続後にデフォルトで`default_catalog`に移動します。
   - StarRocksクラスタにJDBCを使用して接続する場合、接続時に`default_catalog.db_name`を指定してデフォルトのカタログ内の宛先データベースに直接移動することができます。

2. (任意) [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用してデータベースを表示します。

      ```SQL
      SHOW DATABASES;
      ```

      または

      ```SQL
      SHOW DATABASES FROM <catalog_name>;
      ```

3. (任意) [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションで宛先カタログに切り替えます。

    ```SQL
    SET CATALOG <catalog_name>;
    ```

    次に、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、現在のセッションでアクティブなデータベースを指定します。

    ```SQL
    USE <db_name>;
    ```

    または、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、宛先カタログ内のアクティブなデータベースに直接移動することもできます。

    ```SQL
    USE <catalog_name>.<db_name>;
    ```

4. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、内部データをクエリします。

      ```SQL
      SELECT * FROM <table_name>;
      ```

      前述の手順でアクティブなデータベースを指定しない場合、selectクエリで直接指定することもできます。

      ```SQL
      SELECT * FROM <db_name>.<table_name>;
      ```

      または

      ```SQL
      SELECT * FROM default_catalog.<db_name>.<table_name>;
      ```

## 例

`olap_db.olap_table`のデータをクエリするには、次のいずれかの操作を行うことができます。

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

## 参照

外部データソースからデータをクエリする場合は、[外部データのクエリ](../catalog/query_external_data.md)を参照してください。