---
displayed_sidebar: Chinese
---

# DESC

## 機能

以下の操作がこのステートメントで行えます：

- StarRocksのテーブル構造、[ソートキー](../../../table_design/Sort_key.md) (Sort Key)のタイプ、および[マテリアライズドビュー](../../../using_starrocks/Materialized_view.md)を確認する。
- 外部データソース（例：Apache Hive™）のテーブル構造を確認する。この操作はStarRocks 2.4以降のバージョンでのみサポートされています。

## 文法

```SQL
DESC[RIBE] [catalog_name.][db_name.]table_name [ALL];
```

## パラメータ説明

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| catalog_name   | いいえ       | Internal catalogまたはexternal catalogの名前。<ul><li>internal catalogの名前を指定する場合、つまり`default_catalog`の場合は、現在のStarRocksクラスターの指定されたテーブル構造を確認します。</li><li>external catalogの名前を指定する場合は、外部データソースの指定されたテーブル構造を確認します。</li></ul> |
| db_name        | いいえ       | データベース名。                                               |
| table_name     | はい       | テーブル名。                                                   |
| ALL            | いいえ       | <ul><li>StarRocksのテーブルのソートキータイプとマテリアライズドビューを確認する場合は、このキーワードを指定します。テーブル構造のみを確認する場合は指定不要です。</li><li>外部データソースのテーブル構造を確認する場合は、このキーワードを指定できません。</li></ul> |

## 戻り値の説明

```Plain
+-----------+---------------+-------+------+------+-----+---------+-------+
| IndexName | IndexKeysType | Field | Type | Null | Key | Default | Extra |
+-----------+---------------+-------+------+------+-----+---------+-------+
```

戻り値のパラメータ説明：

| **パラメータ** | **説明**                                                     |
| -------------- | ------------------------------------------------------------ |
| IndexName      | テーブル名。外部データソースのテーブル構造を確認する場合、このパラメータは返されません。 |
| IndexKeysType  | テーブルのソートキータイプ。外部データソースのテーブル構造を確認する場合、このパラメータは返されません。 |
| Field          | 列名。                                                       |
| Type           | 列のデータ型。                                               |
| Null           | NULLを許可するかどうか。<ul><li>`yes`: NULLを許可する。</li><li>`no`: NULLを許可しない。</li></ul> |
| Key            | ソートキーかどうか。<ul><li>`true`: ソートキーである。</li><li>`false`: ソートキーではない。</li></ul> |
| Default        | データ型のデフォルト値。デフォルト値がない場合はNULLを返します。 |
| Extra          | <ul><li>StarRocksのテーブル構造を確認する場合、このパラメータは状況に応じて以下の情報を返します：<ul><li>どの集約関数が列に使用されているか、例えばSUMやMIN。</li><li>列にbloom filterインデックスが作成されているか。作成されている場合は、`BLOOM_FILTER`と表示されます。</li></ul></li><li>外部データソースのテーブル構造を確認する場合、このパラメータは列がパーティションキーかどうかを表示します。パーティションキーの場合は、`partition key`と表示されます。</li></ul> |

> 注：マテリアライズドビューの表示については、例2を参照してください。

## 例

例1：StarRocksの`example_table`テーブル構造を確認します。

```SQL
DESC example_table;
```

または

```SQL
DESC default_catalog.example_db.example_table;
```

戻り値は以下の通りです：

```Plain
+-------+---------------+------+-------+---------+-------+
| Field | Type          | Null | Key   | Default | Extra |
+-------+---------------+------+-------+---------+-------+
| k1    | TINYINT       | Yes  | true  | NULL    |       |
| k2    | DECIMAL(10,2) | Yes  | true  | 10.5    |       |
| k3    | CHAR(10)      | Yes  | false | NULL    |       |
| v1    | INT           | Yes  | false | NULL    |       |
+-------+---------------+------+-------+---------+-------+
```

例2：StarRocksの`sales_records`テーブル構造、ソートキータイプ、およびマテリアライズドビューを確認します。以下に示すように、`sales_records`テーブルには`store_amt`というマテリアライズドビューが1つだけあります。

```Plain
DESC db1.sales_records ALL;

+---------------+---------------+-----------+--------+------+-------+---------+-------+
| IndexName     | IndexKeysType | Field     | Type   | Null | Key   | Default | Extra |
+---------------+---------------+-----------+--------+------+-------+---------+-------+
| sales_records | DUP_KEYS      | record_id | INT    | Yes  | true  | NULL    |       |
|               |               | seller_id | INT    | Yes  | true  | NULL    |       |
|               |               | store_id  | INT    | Yes  | true  | NULL    |       |
|               |               | sale_date | DATE   | Yes  | false | NULL    | NONE  |
|               |               | sale_amt  | BIGINT | Yes  | false | NULL    | NONE  |
|               |               |           |        |      |       |         |       |
| store_amt     | AGG_KEYS      | store_id  | INT    | Yes  | true  | NULL    |       |
|               |               | sale_amt  | BIGINT | Yes  | false | NULL    | SUM   |
+---------------+---------------+-----------+--------+------+-------+---------+-------+
```

例3：Hiveの`hive_table`テーブル構造を確認します。

```SQL
DESC hive_catalog.hive_db.hive_table;

+-------+----------------+------+-------+---------+---------------+ 
| Field | Type           | Null | Key   | Default | Extra         | 
+-------+----------------+------+-------+---------+---------------+ 
| id    | INT            | Yes  | false | NULL    |               | 
| name  | VARCHAR(65533) | Yes  | false | NULL    |               | 
| date  | DATE           | Yes  | false | NULL    | partition key | 
+-------+----------------+------+-------+---------+---------------+
```

## 関連文書

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)
