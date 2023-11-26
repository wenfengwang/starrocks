---
displayed_sidebar: "Japanese"
---

# SHOW TABLES

## 説明

StarRocksデータベースまたは外部データソース（例：Hive、Iceberg、Hudi、またはDelta Lake）のデータベース内のすべてのテーブルを表示します。

> **注意**
>
> 外部データソースのテーブルを表示するには、そのデータソースに対応する外部カタログのUSAGE権限が必要です。

## 構文

```sql
SHOW TABLES [FROM <catalog_name>.<db_name>]
```

## パラメータ

 **パラメータ**          | **必須** | **説明**                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| catalog_name | いいえ       | 内部カタログまたは外部カタログの名前。<ul><li>このパラメータを指定しないか、`default_catalog`に設定すると、StarRocksデータベースのテーブルが返されます。</li><li>このパラメータを外部カタログの名前に設定すると、外部データソースのデータベースのテーブルが返されます。</li></ul>内部および外部カタログを表示するには、[SHOW CATALOGS](SHOW_CATALOGS.md)を実行できます。|
| db_name | いいえ       | データベース名。指定しない場合、デフォルトで現在のデータベースが使用されます。 |

## 例

例1：StarRocksクラスタに接続した後、`default_catalog`の`example_db`データベース内のテーブルを表示します。次の2つのステートメントは同等です。

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

例2：このデータベースに接続した後、現在のデータベース`example_db`内のテーブルを表示します。

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

例3：外部カタログ`hudi_catalog`の`hudi_db`データベース内のテーブルを表示します。

```plain
show tables from hudi_catalog.hudi_db;
+----------------------------+
| Tables_in_hudi_db          |
+----------------------------+
| hudi_sync_mor              |
| hudi_table1                |
+----------------------------+
```

または、SET CATALOGを実行して外部カタログ`hudi_catalog`に切り替え、`SHOW TABLES FROM hudi_db;`を実行することもできます。

## 参照

- [SHOW CATALOGS](SHOW_CATALOGS.md)：StarRocksクラスタ内のすべてのカタログを表示します。
- [SHOW DATABASES](SHOW_DATABASES.md)：内部カタログまたは外部カタログ内のすべてのデータベースを表示します。
- [SET CATALOG](../data-definition/SET_CATALOG.md)：カタログ間を切り替えます。
