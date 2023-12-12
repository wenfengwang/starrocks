---
displayed_sidebar: "Japanese"
---

# DESC（ショートカット）

## 説明

次の操作を行うには、このステートメントを使用できます。

- StarRocksクラスターに格納されているテーブルのスキーマを表示できます。[ソートキー](../../../table_design/Sort_key.md)と[マテリアライズドビュー](../../../using_starrocks/Materialized_view.md)のタイプ、テーブルのスキーマを表示することができます。
- Apache Hive™などの外部データソースに格納されているテーブルのスキーマを表示できます。なお、この操作はStarRocks 2.4以降でのみ実行できます。

## 構文

```SQL
DESC[RIBE] [catalog_name.][db_name.]table_name [ALL];
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| catalog_name  | いいえ           | 内部カタログまたは外部カタログの名前です。<ul><li>パラメータの値を内部カタログの名前である`default_catalog`に設定すると、StarRocksクラスターに格納されているテーブルのスキーマを表示できます。</li><li>パラメータの値を外部カタログの名前に設定すると、外部データソースに格納されているテーブルのスキーマを表示できます。</li></ul> |
| db_name       | いいえ           | データベースの名前です。                                           |
| table_name    | はい          | テーブル名です。                                              |
| ALL           | いいえ       | <ul><li>このキーワードが指定されている場合、StarRocksクラスターに格納されているテーブルのソートキーのタイプ、マテリアライズドビュー、およびスキーマを表示できます。このキーワードが指定されていない場合、テーブルのスキーマのみを表示します。</li><li>外部データソースに格納されているテーブルのスキーマを表示する場合は、このキーワードを指定しないでください。</li></ul> |

## 出力

```プレーン
+-----------+---------------+-------+------+------+-----+---------+-------+
| IndexName | IndexKeysType | Field | Type | Null | Key | Default | Extra |
+-----------+---------------+-------+------+------+-----+---------+-------+
```

次の表は、このステートメントによって返されるパラメータについて説明しています。

| **パラメータ** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| IndexName     | テーブル名です。外部データソースに格納されているテーブルのスキーマを表示する場合、このパラメータは返されません。 |
| IndexKeysType | テーブルのソートキーのタイプです。外部データソースに格納されているテーブルのスキーマを表示する場合、このパラメータは返されません。 |
| Field         | カラム名です。                                             |
| Type          | カラムのデータ型です。                                 |
| Null          | カラムの値がNULLであることを示します。<ul><li>`yes`：値がNULLであることを示します。</li><li>`no`：値がNULLでないことを示します。</li></ul>|
| Key           | カラムがソートキーとして使用されているかどうかを示します。<ul><li>`true`：カラムがソートキーとして使用されていることを示します。</li><li>`false`：カラムがソートキーとして使用されていないことを示します。</li></ul>|
| Default       | カラムのデータ型のデフォルト値です。データ型にデフォルト値がない場合、NULLが返されます。 |
| Extra         | <ul><li>StarRocksクラスターに格納されているテーブルのスキーマを表示する場合、このフィールドにはカラムに関する以下の情報が表示されます：<ul><li>カラムが使用する集計関数、例えば`SUM`や`MIN`など。</li><li>カラムにBloomフィルターインデックスが作成されているかどうか。その場合、`Extra`の値は`BLOOM_FILTER`です。</li></ul></li><li>外部データソースに格納されているテーブルのスキーマを表示する場合、このフィールドにはカラムがパーティションカラムであるかどうかが表示されます。カラムがパーティションカラムである場合、`Extra`の値が`partition key`となります。</li></ul>|

> 注意：出力でマテリアライズドビューがどのように表示されるかの情報については、Example 2を参照してください。

## 例

Example 1: StarRocksクラスターに格納されている`example_table`のスキーマを表示します。

```SQL
DESC example_table;
```

または

```SQL
DESC default_catalog.example_db.example_table;
```

上記のステートメントの出力は次のようになります。

```プレーン
+-------+---------------+------+-------+---------+-------+
| Field | Type          | Null | Key   | Default | Extra |
+-------+---------------+------+-------+---------+-------+
| k1    | TINYINT       | Yes  | true  | NULL    |       |
| k2    | DECIMAL(10,2) | Yes  | true  | 10.5    |       |
| k3    | CHAR(10)      | Yes  | false | NULL    |       |
| v1    | INT           | Yes  | false | NULL    |       |
+-------+---------------+------+-------+---------+-------+
```

Example 2: StarRocksクラスターに格納されている`sales_records`のスキーマ、ソートキーのタイプ、およびマテリアライズドビューを表示します。次の例では、`sales_records`に基づいて1つのマテリアライズドビュー`store_amt`が作成されています。

```プレーン
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

Example 3: Hiveクラスターに格納されている`hive_table`のスキーマを表示します。

```プレーン
DESC hive_catalog.hive_db.hive_table;

+-------+----------------+------+-------+---------+---------------+ 
| Field | Type           | Null | Key   | Default | Extra         | 
+-------+----------------+------+-------+---------+---------------+ 
| id    | INT            | Yes  | false | NULL    |               | 
| name  | VARCHAR(65533) | Yes  | false | NULL    |               | 
| date  | DATE           | Yes  | false | NULL    | パーティションキー | 
+-------+----------------+------+-------+---------+---------------+
```

## 参照

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)