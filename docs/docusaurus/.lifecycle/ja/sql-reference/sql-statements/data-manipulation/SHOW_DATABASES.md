---
displayed_sidebar: "Japanese"
---

# データベースを表示する

## 説明

現在のStarRocksクラスタまたは外部データソースのデータベースを表示します。StarRocksはv2.3から外部データソースのデータベースを表示することがサポートされています。

## 構文

```SQL
SHOW DATABASES [FROM <catalog_name>]
```

## パラメータ

| **パラメータ**  | **必須** | **説明**                                              |
| ---------------- | -------- | ------------------------------------------------------ |
| catalog_name     | いいえ     | 内部カタログまたは外部カタログの名前。<ul><li>パラメータを指定しないか、内部カタログの名前である`default_catalog`を指定した場合、現在のStarRocksクラスタのデータベースを表示できます。</li><li>パラメータの値を外部カタログの名前に設定すると、対応する外部データソースのデータベースを表示できます。[SHOW CATALOGS](SHOW_CATALOGS.md)を実行して、内部および外部のカタログを表示できます。</li></ul> |

## 例

Example 1: 現在のStarRocksクラスタのデータベースを表示する。

```SQL
SHOW DATABASES;
```

または

```SQL
SHOW DATABASES FROM default_catalog;
```

前述のステートメントの出力は次のとおりです。

```SQL
+----------+
| Database |
+----------+
| db1      |
| db2      |
| db3      |
+----------+
```

Example 2: `Hive1`外部カタログを使用してHiveクラスタのデータベースを表示する。

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

## 参照

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)