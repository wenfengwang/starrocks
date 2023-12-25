---
displayed_sidebar: English
---

# DESC

## 説明

このステートメントを使用して、次の操作を実行できます。

- StarRocksクラスタに保存されているテーブルのスキーマを、[ソートキーのタイプ](../../../table_design/Sort_key.md)と[マテリアライズドビュー](../../../using_starrocks/Materialized_view.md)とともに表示します。
- Apache Hive™などの次の外部データソースに格納されているテーブルのスキーマを表示します。この操作はStarRocks 2.4以降のバージョンでのみ実行可能です。

## 構文

```SQL
DESC[RIBE] [catalog_name.][db_name.]table_name [ALL];
```

## パラメーター

| **パラメーター** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| catalog_name  | いいえ           | 内部カタログまたは外部カタログの名前。<ul><li>パラメータの値を内部カタログの名前`default_catalog`に設定すると、StarRocksクラスタに格納されているテーブルのスキーマを表示できます。</li><li>パラメータの値を外部カタログの名前に設定すると、外部データソースに格納されているテーブルのスキーマを表示できます。</li></ul> |
| db_name       | いいえ           | データベース名。                                           |
| table_name    | はい          | テーブル名。                                              |
| ALL           | いいえ           | <ul><li>このキーワードを指定すると、StarRocksクラスタに格納されているテーブルのソートキーのタイプ、マテリアライズドビュー、およびスキーマを表示できます。このキーワードを指定しない場合は、テーブルスキーマのみが表示されます。</li><li>外部データソースに格納されているテーブルのスキーマを表示する場合は、このキーワードを指定しないでください。</li></ul> |

## 出力

```Plain
+-----------+---------------+-------+------+------+-----+---------+-------+
| IndexName | IndexKeysType | Field | Type | Null | Key | Default | Extra |
+-----------+---------------+-------+------+------+-----+---------+-------+
```

次の表では、このステートメントによって返されるパラメータについて説明します。

| **パラメーター** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| IndexName     | テーブル名。外部データソースに格納されているテーブルのスキーマを表示する場合、このパラメータは返されません。 |
| IndexKeysType | テーブルのソートキーのタイプ。外部データソースに格納されているテーブルのスキーマを表示する場合、このパラメータは返されません。 |
| Field         | 列名。                                             |
| Type          | 列のデータタイプ。                                 |
| Null          | 列の値がNULLにできるかどうか。<ul><li>`yes`: 値がNULLにできることを示します。</li><li>`no`: 値がNULLにできないことを示します。</li></ul>|
| Key           | 列がソートキーとして使用されているかどうか。<ul><li>`true`: 列がソートキーとして使用されていることを示します。</li><li>`false`: 列がソートキーとして使用されていないことを示します。</li></ul>|
| Default       | 列のデータタイプのデフォルト値。データタイプにデフォルト値がない場合は、NULLが返されます。 |
| Extra         | <ul><li>StarRocksクラスタに格納されているテーブルのスキーマを表示する場合、このフィールドには列に関する情報が表示されます：<ul><li>列で使用される集約関数、例えば`SUM`や`MIN`。</li><li>列にブルームフィルターインデックスが作成されているかどうか。作成されている場合、`Extra`の値は`BLOOM_FILTER`です。</li></ul></li><li>外部データソースに格納されているテーブルのスキーマを表示する場合、このフィールドには列がパーティション列であるかどうかが表示されます。列がパーティション列の場合、`Extra`の値は`partition key`です。</li></ul>|

> 注: マテリアライズドビューが出力でどのように表示されるかについては、例2を参照してください。

## 例

例1: StarRocksクラスタに格納されている`example_table`のスキーマを表示します。

```SQL
DESC example_table;
```

または

```SQL
DESC default_catalog.example_db.example_table;
```

上記のステートメントの出力は次のようになります。

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

例2: StarRocksクラスタに格納されている`sales_records`のスキーマ、ソートキーのタイプ、およびマテリアライズドビューを表示します。次の例では、`sales_records`に基づいてマテリアライズドビュー`store_amt`が作成されています。

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

例3: Hiveクラスタに格納されている`hive_table`のスキーマを表示します。

```Plain
DESC hive_catalog.hive_db.hive_table;

+-------+----------------+------+-------+---------+---------------+ 
| Field | Type           | Null | Key   | Default | Extra         | 
+-------+----------------+------+-------+---------+---------------+ 
| id    | INT            | Yes  | false | NULL    |               | 
| name  | VARCHAR(65533) | Yes  | false | NULL    |               | 
| date  | DATE           | Yes  | false | NULL    | partition key | 
+-------+----------------+------+-------+---------+---------------+
```

## 参照

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)
