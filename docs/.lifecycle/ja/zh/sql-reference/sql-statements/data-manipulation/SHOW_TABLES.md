---
displayed_sidebar: Chinese
---

# テーブルを表示

## 機能

指定されたデータベース内のすべてのテーブルを表示します。StarRocksの内部テーブルまたは外部データソースのテーブルが対象です。

> **注意**
>
> 外部データソースのテーブルを表示するには、対応するExternal CatalogのUSAGE権限が必要です。

## 文法

```sql
SHOW TABLES [FROM <catalog_name>.<db_name>]
```

## パラメータ説明

| **パラメータ**    | **必須** | **説明**                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| catalog_name      | いいえ   | Internal catalogまたはExternal catalogの名前。<ul><li>指定しない場合、またはinternal catalogの名前（`default_catalog`）を指定した場合は、現在のStarRocksクラスター内のデータベースを表示します。</li><li>external catalogの名前を指定した場合は、外部データソース内のデータベースを表示します。</li></ul> 現在のクラスターの内部および外部catalogを表示するには、[SHOW CATALOGS](SHOW_CATALOGS.md) コマンドを使用できます。|
| db_name           | いいえ   | データベース名。指定しない場合は、デフォルトのデータベースが使用されます。 |

## 例

例1：StarRocksクラスターに接続した後、Internal catalogにあるデータベース`example_db`のテーブルを表示します。以下の2つのステートメントは同等の機能を持っています。

```plain
show tables from example_db;
+----------------------------+
| Tables_in_example_db       |
+----------------------------+
| depts                      |
| depts_par                  |
| emps                       |
| emps2                      |
+----------------------------+

show tables from default_catalog.example_db;
+----------------------------+
| Tables_in_example_db       |
+----------------------------+
| depts                      |
| depts_par                  |
| emps                       |
| emps2                      |
+----------------------------+
```

例2：`USE <db_name>;` を使用して`example_db`データベースに接続した後、現在のデータベースにあるテーブルを表示します。

```plain
show tables;

+----------------------------+
| Tables_in_example_db       |
+----------------------------+
| depts                      |
| depts_par                  |
| emps                       |
| emps2                      |
+----------------------------+
```

例3：External catalog `hudi_catalog`にある指定されたデータベース`hudi_db`のテーブルを表示します。

```plain
show tables from hudi_catalog.hudi_db;
+----------------------------+
| Tables_in_hudi_db          |
+----------------------------+
| hudi_sync_mor              |
| hudi_table1                |
+----------------------------+
```

また、`SET CATALOG <catalog_name>;` を使用して`hudi_catalog`に切り替え、その後で`SHOW TABLES FROM hudi_db;`を実行して、そのデータベースのテーブルを表示することもできます。

## 関連文書

- [SHOW CATALOGS](SHOW_CATALOGS.md):現在のクラスターにあるすべてのカタログを表示します。
- [SHOW DATABASES](SHOW_DATABASES.md):Internal CatalogまたはExternal Catalogにあるすべてのデータベースを表示します。
- [SET CATALOG](../data-definition/SET_CATALOG.md):カタログを切り替えます。
