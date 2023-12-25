---
displayed_sidebar: English
---

# SHOW TABLES

## 説明

StarRocksデータベースや外部データソース（例：Hive、Iceberg、Hudi、Delta Lake）のデータベースにある全てのテーブルを表示します。

> **注記**
>
> 外部データソースのテーブルを表示するには、そのデータソースに対応する外部カタログに対するUSAGE権限が必要です。

## 構文

```sql
SHOW TABLES [FROM <catalog_name>.<db_name>]
```

## パラメーター

 **パラメーター**          | **必須** | **説明**                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| catalog_name | いいえ       | 内部カタログまたは外部カタログの名前。<ul><li>このパラメータを指定しないか、`default_catalog`に設定した場合、StarRocksデータベース内のテーブルが返されます。</li><li>このパラメータを外部カタログの名前に設定すると、外部データソースのデータベース内のテーブルが返されます。</li></ul> [SHOW CATALOGS](SHOW_CATALOGS.md) を実行して、内部カタログと外部カタログを表示できます。|
| db_name | いいえ       | データベース名。指定しない場合、現在のデータベースがデフォルトで使用されます。 |

## 例

例1: StarRocksクラスタに接続した後、`default_catalog`の`example_db`データベース内のテーブルを表示します。以下の2つのステートメントは同等です。

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

例2: このデータベースに接続した後、現在のデータベース`example_db`内のテーブルを表示します。

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

例3: 外部カタログ`hudi_catalog`の`hudi_db`データベース内のテーブルを表示します。

```plain
show tables from hudi_catalog.hudi_db;
+----------------------------+
| Tables_in_hudi_db          |
+----------------------------+
| hudi_sync_mor              |
| hudi_table1                |
+----------------------------+
```

または、`SET CATALOG`を実行して外部カタログ`hudi_catalog`に切り替えてから、`SHOW TABLES FROM hudi_db;`を実行することもできます。

## 参照

- [SHOW CATALOGS](SHOW_CATALOGS.md): StarRocksクラスタ内の全てのカタログを表示します。
- [SHOW DATABASES](SHOW_DATABASES.md): 内部カタログまたは外部カタログ内の全てのデータベースを表示します。
- [SET CATALOG](../data-definition/SET_CATALOG.md): カタログ間で切り替えます。
