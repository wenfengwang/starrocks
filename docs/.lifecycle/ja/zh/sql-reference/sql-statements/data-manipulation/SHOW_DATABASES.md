---
displayed_sidebar: Chinese
---

# データベースを表示

## 機能

現在のStarRocksクラスターまたは外部データソースのデータベースを表示します。

## 文法

```SQL
SHOW DATABASES [FROM <catalog_name>]
```

## パラメータ説明

| **パラメータ**    | **必須** | **説明**                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| catalog_name      | いいえ   | Internal catalogまたはexternal catalogの名前。<ul><li>指定しない場合、またはinternal catalogの名前である`default_catalog`を指定した場合は、現在のStarRocksクラスターのデータベースを表示します。</li><li>external catalogの名前を指定した場合は、外部データソースのデータベースを表示します。現在のクラスターの内部および外部catalogを表示するには[SHOW CATALOGS](SHOW_CATALOGS.md)コマンドを使用できます。</li></ul> |

## 例

例1：現在のStarRocksクラスターのデータベースを表示します。

```SQL
SHOW DATABASES;
```

または

```SQL
SHOW DATABASES FROM default_catalog;
```

返される情報は以下の通りです：

```SQL
+----------+
| Database |
+----------+
| db1      |
| db2      |
| db3      |
+----------+
```

例2：external catalog `hive1`を通じてApache Hive™のデータベースを表示します。

```SQL
SHOW DATABASES FROM hive1;

+-----------+
| Database  |
+-----------+
| hive_db1  |
| hive_db2  |
| hive_db3  |
+-----------+
```

## 関連参照

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)
