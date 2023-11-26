---
displayed_sidebar: "Japanese"
---

# SHOW DATABASES

## 説明

現在のStarRocksクラスタまたは外部データソースのデータベースを表示します。StarRocksは、v2.3以降、外部データソースのデータベースを表示することができます。

## 構文

```SQL
SHOW DATABASES [FROM <catalog_name>]
```

## パラメータ

| **パラメータ**   | **必須** | **説明**                                                                                                                                                                                                                       |
| ----------------- | ------------ | ------------------------------------------------------------ |
| catalog_name      | No           | 内部カタログまたは外部カタログの名前。<ul><li>パラメータを指定しないか、内部カタログの名前である`default_catalog`を指定すると、現在のStarRocksクラスタのデータベースを表示することができます。</li><li>パラメータの値を外部カタログの名前に設定すると、対応する外部データソースのデータベースを表示することができます。内部および外部のカタログを表示するには、[SHOW CATALOGS](SHOW_CATALOGS.md)を実行できます。</li></ul> |

## 例

例1：現在のStarRocksクラスタのデータベースを表示します。

```SQL
SHOW DATABASES;
```

または

```SQL
SHOW DATABASES FROM default_catalog;
```

上記のステートメントの出力は次のようになります。

```SQL
+----------+
| Database |
+----------+
| db1      |
| db2      |
| db3      |
+----------+
```

例2：`Hive1`外部カタログを使用してHiveクラスタのデータベースを表示します。

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
