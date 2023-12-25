---
displayed_sidebar: English
---

# JSON

StarRocksはv2.2.0からJSONデータ型のサポートを開始しました。このトピックでは、JSONの基本概念について説明します。また、JSONカラムの作成方法、JSONデータのロード方法、JSONデータのクエリ方法、JSON関数とオペレーターを使用してJSONデータを構築および処理する方法についても説明します。

## JSONとは

JSONは、半構造化データ用に設計された軽量なデータ交換フォーマットです。JSONはデータを階層的なツリー構造で表現し、幅広いデータストレージや分析シナリオで柔軟かつ容易に読み書きできます。JSONは`NULL`値と以下のデータ型をサポートしています：NUMBER、STRING、BOOLEAN、ARRAY、OBJECT。

JSONについての詳細は、[JSONのウェブサイト](http://www.json.org/?spm=a2c63.p38356.0.0.50756b9fVEfwCd)をご覧ください。JSONの入力および出力の構文については、[RFC 7159のJSON仕様](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4)を参照してください。

StarRocksはJSONデータのストレージと効率的なクエリおよび分析をサポートしています。StarRocksは入力テキストを直接保存するのではなく、解析コストを削減しクエリ効率を向上させるためにJSONデータをバイナリ形式で保存します。

## JSONデータの使用

### JSONカラムの作成

テーブルを作成する際に、`JSON`キーワードを使用してカラム`j`をJSONカラムとして指定できます。

```sql
CREATE TABLE `tj` (
    `id` INT(11) NOT NULL COMMENT "",
    `j`  JSON NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`)
PROPERTIES (
    "replication_num" = "3",
    "storage_format" = "DEFAULT"
);
```

### データをロードしてJSONデータとして保存する

StarRocksは、データをロードしてJSONデータとして保存するための以下の方法を提供しています：

- 方法1: `INSERT INTO`を使用してテーブルのJSONカラムにデータを書き込みます。以下の例では、`tj`という名前のテーブルが使用され、カラム`j`はJSONカラムです。

```plaintext
INSERT INTO tj (id, j) VALUES (1, parse_json('{"a": 1, "b": true}'));
INSERT INTO tj (id, j) VALUES (2, parse_json('{"a": 2, "b": false}'));
INSERT INTO tj (id, j) VALUES (3, parse_json('{"a": 3, "b": true}'));
INSERT INTO tj (id, j) VALUES (4, json_object('a', 4, 'b', false)); 
```

> `parse_json`関数はSTRINGデータをJSONデータとして解釈できます。`json_object`関数はJSONオブジェクトを構築するか、既存のテーブルをJSONファイルに変換することができます。詳細は[parse_json](../../sql-functions/json-functions/json-constructor-functions/parse_json.md)および[json_object](../../sql-functions/json-functions/json-constructor-functions/json_object.md)を参照してください。

- 方法2: Stream Loadを使用してJSONファイルをロードし、そのファイルをJSONデータとして保存します。詳細は[JSONデータのロード](../../../loading/StreamLoad.md#load-json-data)を参照してください。

  - ルートJSONオブジェクトをロードする場合は、`jsonpaths`を`$`に設定します。
  - JSONオブジェクトの特定の値をロードする場合は、`jsonpaths`を`$.a`に設定し、`a`はキーを指定します。StarRocksでサポートされているJSONパス式については[JSONパス](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md#json-path-expressions)を参照してください。

- 方法3: Broker Loadを使用してParquetファイルをロードし、そのファイルをJSONデータとして保存します。詳細は[Broker Load](../data-manipulation/BROKER_LOAD.md)を参照してください。

StarRocksはParquetファイルのロード時に以下のデータ型変換をサポートしています。

| Parquetファイルのデータ型                                    | JSONデータ型 |
| ------------------------------------------------------------ | -------------- |
| 整数 (INT8、INT16、INT32、INT64、UINT8、UINT16、UINT32、およびUINT64) | NUMBER         |
| FLOATおよびDOUBLE                                             | NUMBER         |
| BOOLEAN                                                      | BOOLEAN        |
| STRING                                                       | STRING         |
| MAP                                                          | OBJECT         |
| STRUCT                                                       | OBJECT         |
| LIST                                                         | ARRAY          |
| その他のデータ型（UNIONやTIMESTAMPなど）                     | サポートされていません  |

- 方法4: [Routine Load](../../../loading/RoutineLoad.md)を使用して、KafkaからStarRocksにJSONデータを継続的にロードします。

### JSONデータのクエリと処理

StarRocksはJSONデータのクエリと処理、およびJSON関数とオペレーターの使用をサポートしています。

以下の例では、`tj`という名前のテーブルが使用され、カラム`j`はJSONカラムとして指定されています。

```plaintext
mysql> select * from tj;
+------+----------------------+
| id   |          j           |
+------+----------------------+
| 1    | {"a": 1, "b": true}  |
| 2    | {"a": 2, "b": false} |
| 3    | {"a": 3, "b": true}  |
| 4    | {"a": 4, "b": false} |
+------+----------------------+
```

例1: JSONカラムのデータをフィルタリングして、`id=1`のフィルタ条件を満たすデータを取得します。

```plaintext
mysql> select * from tj where id = 1;
+------+---------------------+
| id   |           j         |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
+------+---------------------+
```

例2: JSONカラム`j`のデータをフィルタリングして、指定されたフィルタ条件を満たすデータを取得します。

> `j->'a'`はJSONデータを返します。最初の例を使用してデータを比較することができます（この例では暗黙の型変換が行われます）。または、CAST関数を使用してJSONデータをINTに変換してからデータを比較することもできます。

```plaintext
mysql> select * from tj where j->'a' = 1;
+------+---------------------+
| id   | j                   |
+------+---------------------+
|    1 | {"a": 1, "b": true} |


mysql> select * from tj where cast(j->'a' as INT) = 1;
+------+---------------------+
| id   | j                   |
+------+---------------------+
|    1 | {"a": 1, "b": true} |
+------+---------------------+
```

例3: CAST関数を使用して、テーブルのJSONカラムの値をBOOLEAN値に変換します。次に、JSONカラムのデータをフィルタリングして、指定されたフィルタ条件を満たすデータを取得します。

```plaintext
mysql> select * from tj where cast(j->'b' as BOOLEAN);
+------+---------------------+
|  id  |          j          |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
| 3    | {"a": 3, "b": true} |
+------+---------------------+
```

例4: CAST関数を使用して、テーブルのJSONカラムの値をBOOLEAN値に変換します。次に、JSONカラムのデータをフィルタリングして、指定されたフィルタ条件を満たすデータを取得し、そのデータに対して算術演算を行います。

```plaintext
mysql> select cast(j->'a' as INT) from tj where cast(j->'b' as BOOLEAN);
+-----------------------+
|  CAST(j->'a' AS INT)  |
+-----------------------+
|          3            |
|          1            |
+-----------------------+

mysql> select sum(cast(j->'a' as INT)) from tj where cast(j->'b' as BOOLEAN);
+----------------------------+
| sum(CAST(j->'a' AS INT))   |
+----------------------------+
|              4             |
+----------------------------+
```

例5: JSONカラムをソートキーとして使用して、テーブルのデータをソートします。

```plaintext
mysql> select * from tj
    ->        where j->'a' <= parse_json('3')
    ->        order by cast(j->'a' as INT);
+------+----------------------+
| id   | j                    |
+------+----------------------+
|    1 | {"a": 1, "b": true}  |
|    2 | {"a": 2, "b": false} |
|    3 | {"a": 3, "b": true}  |
|    4 | {"a": 4, "b": false} |
+------+----------------------+
4 rows in set (0.05 sec)
```

## JSON関数とオペレーター

JSON関数とオペレーターを使用してJSONデータを構築および処理することができます。詳細は[JSON関数とオペレーターの概要](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md)を参照してください。

## 制限事項と使用上の注意点

- JSON値の最大長は16MBです。

- ORDER BY、GROUP BY、およびJOIN句はJSONカラムへの参照をサポートしていません。JSONカラムへの参照を作成する場合は、参照を作成する前にCAST関数を使用してJSONカラムをSQLカラムに変換してください。詳細は[cast](../../sql-functions/json-functions/json-query-and-processing-functions/cast.md)を参照してください。

- JSONカラムはDuplicate Key、Primary Key、およびUnique Keyテーブルでサポートされていますが、Aggregateテーブルではサポートされていません。

- JSONカラムはDUPLICATE KEY、PRIMARY KEY、およびUNIQUE KEYテーブルのパーティションキー、バケッティングキー、またはディメンションカラムとして使用することはできません。また、ORDER BY、GROUP BY、およびJOIN句での使用もサポートされていません。

- StarRocksでは、以下のJSON比較オペレーターを使用してJSONデータをクエリできます：`<`、`<=`、`>`、`>=`、`=`、`!=`。`IN`を使用してJSONデータをクエリすることはできません。
