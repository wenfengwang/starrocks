---
displayed_sidebar: "Japanese"
---

# DESC

## 説明

次の操作を実行するためにステートメントを使用できます。

- StarRocksクラスターに保存されているテーブルのスキーマを表示し、テーブルの[ソートキー](../../../table_design/Sort_key.md)および[マテリアライズドビュー](../../../using_starrocks/Materialized_view.md)のタイプを表示します。
- Apache Hive™などの外部データソースに保存されているテーブルのスキーマを表示します。この操作は、StarRocks 2.4以降のバージョンでのみ実行できます。

## 構文

```SQL
DESC[RIBE] [catalog_name.][db_name.]table_name [ALL];
```

## パラメーター

| **パラメーター** | **必須** | **説明**                                              |
| ----------------- | -------- | ------------------------------------------------------ |
| catalog_name      | いいえ   | 内部カタログまたは外部カタログの名前です。<ul><li>パラメーターの値を内部カタログの名前（`default_catalog`）に設定した場合、StarRocksクラスターに保存されているテーブルのスキーマを表示できます。</li><li>パラメーターの値を外部カタログの名前に設定した場合、外部データソースに保存されているテーブルのスキーマを表示できます。</li></ul> |
| db_name           | いいえ   | データベースの名前です。                                 |
| table_name        | はい     | テーブルの名前です。                                    |
| ALL               | いいえ   | <ul><li>このキーワードが指定されている場合、StarRocksクラスターに保存されているテーブルのソートキーのタイプ、マテリアライズドビュー、およびスキーマを表示できます。このキーワードが指定されていない場合、テーブルのスキーマのみを表示します。</li><li>外部データソースに保存されているテーブルのスキーマを表示する場合、このキーワードを指定しないでください。</li></ul> |

## 出力

```Plain
+-----------+---------------+-------+------+------+-----+---------+-------+
| IndexName | IndexKeysType | Field | Type | Null | Key | Default | Extra |
+-----------+---------------+-------+------+------+-----+---------+-------+
```

次の表は、このステートメントによって返されるパラメーターを説明しています。

| **パラメーター** | **説明**                                              |
| ----------------- | ------------------------------------------------------ |
| IndexName         | テーブル名です。外部データソースに保存されているテーブルのスキーマを表示する場合、このパラメーターは返されません。 |
| IndexKeysType     | テーブルのソートキーのタイプです。外部データソースに保存されているテーブルのスキーマを表示する場合、このパラメーターは返されません。 |
| Field             | 列名です。                                              |
| Type              | 列のデータ型です。                                      |
| Null              | 列の値がNULLであるかどうか。<ul><li>`yes`: 値がNULLであることを示します。</li><li>`no`: 値がNULLであることを示しません。</li></ul>|
| Key               | 列がソートキーとして使用されているかどうか。<ul><li>`true`: 列がソートキーとして使用されています。</li><li>`false`: 列がソートキーとして使用されていません。</li></ul>|
| Default           | 列のデータ型のデフォルト値です。データ型にデフォルト値がない場合、NULLが返されます。 |
| Extra             | <ul><li>StarRocksクラスターに保存されているテーブルのスキーマを表示する場合、このフィールドには列に関する次の情報が表示されます：<ul><li>列で使用される集約関数（`SUM`や`MIN`など）。</li><li>列にブルームフィルターインデックスが作成されているかどうか。作成されている場合、`Extra`の値は`BLOOM_FILTER`です。</li></ul></li><li>外部データソースに保存されているテーブルのスキーマを表示する場合、このフィールドには列がパーティション列であるかどうかが表示されます。列がパーティション列である場合、`Extra`の値は`partition key`です。</li></ul>|

> 注：出力でマテリアライズドビューがどのように表示されるかの情報については、Example 2を参照してください。

## 例

例 1：StarRocksクラスターに保存されている`example_table`のスキーマを表示します。

```SQL
DESC example_table;
```

または

```SQL
DESC default_catalog.example_db.example_table;
```

上記のステートメントの出力は次の通りです。

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

例 2：StarRocksクラスターに保存されている`sales_records`のスキーマ、ソートキーのタイプ、およびマテリアライズドビューを表示します。次の例では、`sales_records`を基にしたマテリアライズドビュー`store_amt`が1つ作成されています。

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

例 3：Hiveクラスターに保存されている`hive_table`のスキーマを表示します。

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