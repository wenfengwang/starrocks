---
displayed_sidebar: English
---

# SHOW DATABASES

## 説明

現在のStarRocksクラスタまたは外部データソースのデータベースを表示します。StarRocksはv2.3以降、外部データソースのデータベース表示をサポートしています。

## 構文

```SQL
SHOW DATABASES [FROM <catalog_name>]
```

## パラメーター

| **パラメーター**     | **必須** | **説明**                                              |
| ----------------- | ------------ | ------------------------------------------------------------ |
| catalog_name      | いいえ           | 内部カタログまたは外部カタログの名前。<ul><li>パラメータを指定しない場合、または`default_catalog`という内部カタログの名前を指定した場合は、現在のStarRocksクラスタ内のデータベースを表示できます。</li><li>パラメータの値を外部カタログの名前に設定すると、対応する外部データソースのデータベースを表示できます。内部カタログと外部カタログを表示するには、[SHOW CATALOGS](SHOW_CATALOGS.md)を実行します。</li></ul> |

## 例

例 1: 現在のStarRocksクラスタ内のデータベースを表示します。

```SQL
SHOW DATABASES;
```

または

```SQL
SHOW DATABASES FROM default_catalog;
```

上記のステートメントの出力は以下の通りです。

```SQL
+----------+
| Database |
+----------+
| db1      |
| db2      |
| db3      |
+----------+
```

例 2: `Hive1`外部カタログを使用してHiveクラスタ内のデータベースを表示します。

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
- [DESCRIBE](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)
