---
displayed_sidebar: "Japanese"
---

# DESC

## 説明

次の操作を実行するために、このステートメントを使用できます。

- StarRocksクラスタに保存されているテーブルのスキーマを表示します。テーブルの[ソートキー](../../../table_design/Sort_key.md)のタイプと[マテリアライズドビュー](../../../using_starrocks/Materialized_view.md)のタイプも表示されます。
- Apache Hive™などの外部データソースに保存されているテーブルのスキーマを表示します。この操作は、StarRocks 2.4以降のバージョンでのみ実行できます。

## 構文

```SQL
DESC[RIBE] [catalog_name.][db_name.]table_name [ALL];
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                    |
| ------------- | ------------ | ------------------------------------------------------------ |
| catalog_name  | No           | 内部カタログまたは外部カタログの名前です。 <ul><li>パラメータの値を内部カタログの名前である`default_catalog`に設定すると、StarRocksクラスタに保存されているテーブルのスキーマを表示できます。 </li><li>パラメータの値を外部カタログの名前に設定すると、外部データソースに保存されているテーブルのスキーマを表示できます。</li></ul> |
| db_name       | No           | データベースの名前です。                                           |
| table_name    | Yes          | テーブルの名前です。                                              |
| ALL           | No           | <ul><li>このキーワードを指定すると、StarRocksクラスタに保存されているテーブルのソートキーのタイプ、マテリアライズドビューのタイプ、およびスキーマを表示できます。このキーワードを指定しない場合、テーブルのスキーマのみを表示します。 </li><li>外部データソースに保存されているテーブルのスキーマを表示する場合は、このキーワードを指定しないでください。</li></ul> |

## 出力

```Plain
+-----------+---------------+-------+------+------+-----+---------+-------+
| IndexName | IndexKeysType | Field | Type | Null | Key | Default | Extra |
+-----------+---------------+-------+------+------+-----+---------+-------+
```

次の表は、このステートメントによって返されるパラメータを説明しています。

| **パラメータ** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| IndexName     | テーブルの名前です。外部データソースに保存されているテーブルのスキーマを表示する場合、このパラメータは返されません。 |
| IndexKeysType | テーブルのソートキーのタイプです。外部データソースに保存されているテーブルのスキーマを表示する場合、このパラメータは返されません。 |
| Field         | カラムの名前です。                                             |
| Type          | カラムのデータ型です。                                 |
| Null          | カラムの値がNULLになる可能性があるかどうかを示します。 <ul><li>`yes`：値がNULLになる可能性があります。 </li><li>`no`：値がNULLになりません。 </li></ul>|
| Key           | カラムがソートキーとして使用されているかどうかを示します。 <ul><li>`true`：カラムがソートキーとして使用されています。 </li><li>`false`：カラムがソートキーとして使用されていません。 </li></ul>|
| Default       | カラムのデータ型のデフォルト値です。データ型にデフォルト値がない場合、NULLが返されます。 |
| Extra         | <ul><li>StarRocksクラスタに保存されているテーブルのスキーマを表示する場合、このフィールドには次のようなカラムに関する情報が表示されます: <ul><li>カラムで使用される集計関数（`SUM`や`MIN`など）。</li><li>カラムにブルームフィルタインデックスが作成されているかどうか。作成されている場合、`Extra`の値は`BLOOM_FILTER`です。</li></ul></li><li>外部データソースに保存されているテーブルのスキーマを表示する場合、このフィールドにはカラムがパーティションカラムであるかどうかが表示されます。カラムがパーティションカラムである場合、`Extra`の値は`partition key`です。</li></ul>|

> 注意: マテリアライズドビューが出力にどのように表示されるかの情報については、例2を参照してください。

## 例

例1: StarRocksクラスタに保存されている`example_table`のスキーマを表示します。

```SQL
DESC example_table;
```

または

```SQL
DESC default_catalog.example_db.example_table;
```

前述のステートメントの出力は次のようになります。

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

例2: StarRocksクラスタに保存されている`sales_records`のスキーマ、ソートキーのタイプ、およびマテリアライズドビューを表示します。次の例では、`sales_records`を基にしたマテリアライズドビュー`store_amt`が1つ作成されています。

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

例3: Hiveクラスタに保存されている`hive_table`のスキーマを表示します。

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
